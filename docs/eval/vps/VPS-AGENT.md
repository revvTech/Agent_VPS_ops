# RevvTech Agent Platform — VPS Agent Context

> Read this first if you are an AI agent working on this VPS.
> This file IS the map (Layer 1) for `srv1096690.hstgr.cloud`.

**Last updated:** 2026-04-08

---

## What This VPS Is

RevvTech's dedicated agent hosting server. It runs two platforms side-by-side:

| Platform | Role | How it runs | Port |
|---|---|---|---|
| **Dify** | Agent management UI — build, deploy, expose agents to clients | Docker Compose (11 containers) | 80 |
| **Agno Playground** | Agent runtime — run Python-native agents directly | Python venv (no Docker) | 7777 |

**This is NOT a general-purpose server.** It hosts agent infrastructure only.

---

## Why This Architecture

### Philosophy: Computational Orchestration (LDP)

RevvTech builds agents using the Layered Design Pattern. The folder structure is the application. English is the routing software. No black boxes.

| Layer | On this VPS |
|---|---|
| Layer 1 — The Map | This file (`VPS-AGENT.md`) + `/opt/revvtech/` structure |
| Layer 2 — The Rooms | Each service has its own directory with config and context |
| Layer 3 — The Workspace | Dify Studio UI + Agno Playground where work happens |

### Why two platforms?

| Question | Answer |
|---|---|
| Why Dify? | Visual workflow builder, RAG, prompt versioning, multi-tenant workspaces, client-facing chat UIs. Best open-source option for a branded agent control centre. |
| Why Agno? | Python-native agent runtime. Builds, runs, and exposes agents via FastAPI. MCP integration for external tools. No Docker overhead. |
| Why not just one? | Dify doesn't natively manage Agno/Mastra/CrewAI agents. Agno doesn't have Dify's visual builder or RAG. They complement each other — Dify is the control plane, Agno is the execution runtime. Dify can call Agno agents as HTTP endpoints. |
| Why Docker for Dify only? | Dify requires 11 containers (API, web, worker, PostgreSQL, Redis, Weaviate, nginx, sandbox, etc). Docker Compose is the only supported deployment method. No bare-metal option exists. |
| Why no Docker for Agno? | Agno is a pip-installable Python library. A venv is simpler, lighter, and easier to debug. No container overhead on a 8GB RAM box. |

### Why this VPS specifically?

| Spec | Value | Why it works |
|---|---|---|
| CPU | 2 vCPU (AMD EPYC) | Meets Dify minimum (2 cores) |
| RAM | 8 GB | Meets Dify recommended (8 GB). Agno adds ~200MB. |
| Disk | 100 GB SSD | Dify + vectors + logs fit comfortably |
| OS | Ubuntu 24.04 LTS | Docker-compatible, long-term support |
| Provider | Hostinger KVM 2 | ~$10-15/month, good for the workload |
| Hostname | srv1096690.hstgr.cloud | |
| IP | 72.61.98.2 | |
| Expires | 2027-10-31 | |

---

## VPS Folder Structure

```
/opt/revvtech/                    ← all RevvTech services live here
├── VPS-AGENT.md                  ← this file (Layer 1 map — symlinked or copied)
├── dify/                         ← Dify platform (Docker)
│   └── docker/
│       ├── .env                  ← Dify config (secrets, API keys, URLs)
│       └── docker-compose.yaml   ← 11-container orchestration
├── agno/                         ← Agno Playground (Python venv)
│   ├── venv/                     ← Python virtual environment
│   ├── agno-playground.py        ← agent definitions + FastAPI server
│   └── agno-playground.log       ← runtime logs
└── backups/                      ← scheduled backup dumps
```

---

## Services Detail

### Dify (Docker Compose)

11 containers managed by `docker compose`:

| Container | Purpose | Exposed |
|---|---|---|
| nginx | Reverse proxy | Port 80 (public) |
| api | Dify API (FastAPI) | Internal only |
| web | Next.js frontend | Internal only |
| worker | Celery async tasks | Internal only |
| worker_beat | Celery scheduler | Internal only |
| db_postgres | PostgreSQL 15 | Internal only |
| redis | Cache + queue | Internal only |
| sandbox | Secure code exec | Internal only |
| ssrf_proxy | Squid proxy | Internal only |
| weaviate | Vector DB for RAG | Internal only |
| plugin_daemon | Plugin system | Internal only |

**Data persistence:** Docker volumes. PostgreSQL, Redis, and Weaviate data survive container restarts.

**Management commands:**
```bash
cd /opt/revvtech/dify/docker
docker compose ps              # status
docker compose logs -f api     # tail API logs
docker compose restart api     # restart single service
docker compose down            # stop all
docker compose up -d           # start all
```

### Agno Playground (Python venv)

Single Python process running FastAPI on port 7777.

| Agent | Purpose |
|---|---|
| RevvTech Assistant | RAG over company knowledge |
| DDG Outreach Agent | Personalised first-touch messages |
| DDG Data Acquisition | ICP prospecting (demo mode) |

**Management commands:**
```bash
# Start
source /opt/revvtech/agno/venv/bin/activate
python3 /opt/revvtech/agno/agno-playground.py

# Background (survives disconnect)
nohup /opt/revvtech/agno/venv/bin/python3 /opt/revvtech/agno/agno-playground.py \
  > /opt/revvtech/agno/agno-playground.log 2>&1 &

# Check if running
pgrep -f agno-playground

# Stop
pkill -f agno-playground

# Logs
tail -f /opt/revvtech/agno/agno-playground.log
```

---

## Access

| What | URL | Auth |
|---|---|---|
| Dify Studio | `http://72.61.98.2` | Setup on first visit at `/install` |
| Agno Playground | `http://72.61.98.2:7777` | None (internal use) |
| SSH | `ssh louis@72.61.98.2` | SSH key only |

**SSH key authorized:**
```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJxpUcjdrf6IuBA91Ha334LjSFmuzftEvrEnU2QqmqoI louisdup@louisdup-Vivobook-ASUSLaptop-X1504VA-X1504VA
```

---

## Security

| Rule | Setting |
|---|---|
| SSH access | Key-only. Password login disabled. |
| Root login | Disabled |
| Admin user | `louis` (sudo) |
| Firewall (UFW) | Ports 22, 80, 443, 7777 only |
| API keys | In `.env` files or env vars. Never in code or agent memory. |
| Client separation | Dify workspaces per client. Never mix. |

---

## How Dify + Agno Work Together

```
Client browser
    │
    ▼
┌─────────────────────────────┐
│  Dify (port 80)             │
│  - Visual workflow builder  │
│  - RAG / knowledge bases    │
│  - Chat UIs for clients     │
│  - Prompt versioning        │
│  - Calls Agno via HTTP ──────────┐
└─────────────────────────────┘    │
                                    ▼
                          ┌──────────────────────┐
                          │  Agno (port 7777)    │
                          │  - Python agents     │
                          │  - MCP tool access   │
                          │  - Direct LLM calls  │
                          │  - FastAPI endpoints  │
                          └──────────────────────┘
```

**Integration pattern:** Dify treats Agno agents as HTTP tool endpoints. A Dify workflow can call `http://localhost:7777/v1/agent/run` to invoke an Agno agent, then use the response in its own pipeline. This means:
- Dify = control plane + UI + RAG + client-facing
- Agno = execution runtime + MCP + Python-native agents

---

## Resource Budget (8 GB RAM)

| Service | Expected RAM | Notes |
|---|---|---|
| Dify (all containers) | ~4-5 GB | PostgreSQL + Weaviate are the heaviest |
| Agno Playground | ~200-400 MB | Single FastAPI process |
| OS + system | ~500 MB | Ubuntu baseline |
| **Headroom** | **~2-3 GB** | For spikes, new agents, caching |

**If memory becomes tight:**
1. Reduce Weaviate memory (`QUERY_MAXIMUM_RESULTS`, `PERSISTENCE_FLUSH_INTERVAL`)
2. Limit PostgreSQL `shared_buffers` and `work_mem`
3. Move Agno to a separate lightweight VPS if needed
4. Monitor with `htop` and `docker stats`

---

## Deployment Sequence

```
Step 1 — System prep          apt update, install Docker, create user, harden SSH
Step 2 — Dify deploy          clone → configure .env → docker compose up -d
Step 3 — Agno deploy          create venv → pip install agno → run playground
Step 4 — Firewall             ufw allow 22,80,443,7777 → ufw enable
Step 5 — Verify               curl localhost (Dify) + curl localhost:7777 (Agno)
Step 6 — Import agents        Import DSL files into Dify, configure Agno agents
```

**Estimated time:** ~35 minutes from SSH to fully running.

---

## Future Roadmap

| When | What |
|---|---|
| Now | Deploy Dify + Agno Playground to VPS |
| Soon | Fork Dify → `revvTech/dify-revvtech`, white-label branding |
| Soon | Custom domain (`agent.revvtech.com`) + Let's Encrypt SSL |
| Later | LiteLLM as API gateway between Dify and LLM providers |
| Later | Langfuse for observability |
| Later | systemd service for Agno (auto-restart on boot) |
| Later | Automated backups (cron + tar of Docker volumes) |

---

## Connections to RevvTech Stack

| System | Where | Relationship |
|---|---|---|
| Pi coding agent | Louis's laptop | Manages this VPS via SSH |
| Mastra | `~/Agents/Mastra/` (local) | TypeScript agents, can be called by Dify via HTTP |
| OpenClaw | srv1385761.hstgr.cloud | Claude Desktop proxy for specific clients |
| Task Clinical | srv1301748.hstgr.cloud | Agno agent (separate VPS, at capacity) |
| RevvTech Main | 72.62.235.141 | Production apps (at capacity, do NOT add agents here) |
| GitHub | github.com/revvTech/ | Source repos, Dify fork |

---

## Rules for Agents Working on This VPS

1. **Read this file first** before making any changes
2. **Never store secrets in code or markdown** — use `.env` files or env vars
3. **Never mix client data** — separate Dify workspaces per client
4. **Ask before destructive actions** — `docker compose down`, deleting volumes, changing SSH config
5. **Log decisions** — update this file or `DEPLOY.md` when architecture changes
6. **Monitor resources** — check `htop` / `docker stats` before adding new services
7. **Keep it simple** — two services (Dify + Agno). No scope creep.
