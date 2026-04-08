# RevvTech Agent VPS — Ops & Eval Docs

> Single source of truth for VPS agent platform planning, deployment logs, and architecture decisions.

## What's Here

| File | Purpose |
|---|---|
| [docs/eval/vps/PLAN.md](docs/eval/vps/PLAN.md) | Eval plan — goals, phases, success criteria |
| [docs/eval/vps/VPS-AGENT.md](docs/eval/vps/VPS-AGENT.md) | VPS architecture map — what runs where, why, how |
| [docs/eval/vps/OPS_LOG.md](docs/eval/vps/OPS_LOG.md) | Phase-by-phase execution log — what was done, outcomes, decisions |

## VPS Overview

| Item | Value |
|---|---|
| **Server** | `srv1096690.hstgr.cloud` (72.61.98.2) |
| **OS** | Ubuntu 24.04 LTS |
| **Specs** | 2 vCPU, 8 GB RAM, 100 GB SSD |
| **Expires** | 2027-10-31 |

## What's Running

| Service | URL | How it runs |
|---|---|---|
| **Dify** (agent management UI) | `http://72.61.98.2` | Docker Compose (11 containers) |
| **Agno Playground** (agent runtime) | `http://72.61.98.2:7777` | Python venv + systemd |

## Architecture (two runtimes)

```
Client browser
    │
    ▼
┌─────────────────────────────┐
│  Dify (port 80)             │
│  - Visual workflow builder  │
│  - RAG / knowledge bases    │
│  - Chat UIs for clients     │
│  - Calls Agno via HTTP ──────────┐
└─────────────────────────────┘    │
                                    ▼
                          ┌──────────────────────┐
                          │  Agno (port 7777)    │
                          │  - Python agents     │
                          │  - MCP tool access   │
                          │  - FastAPI endpoints  │
                          └──────────────────────┘
```

- **Dify** = control plane + UI + RAG + client-facing apps
- **Agno** = execution runtime + MCP + Python-native agents
- Dify calls Agno agents as HTTP endpoints

## How to Read the Docs

1. **Start with** [PLAN.md](docs/eval/vps/PLAN.md) — understand the goals and phases
2. **Then read** [VPS-AGENT.md](docs/eval/vps/VPS-AGENT.md) — understand what's on the VPS and why
3. **Track progress in** [OPS_LOG.md](docs/eval/vps/OPS_LOG.md) — every action, decision, and outcome logged

## How to Reproduce the VPS Eval

```bash
# 1. SSH into the VPS
ssh louis@srv1096690.hstgr.cloud

# 2. Check Dify status
cd /opt/revvtech/dify/docker && docker compose ps

# 3. Check Agno status
sudo systemctl status agno-playground

# 4. Test health endpoints
curl -s http://localhost/console/api/setup        # Dify → should return 200
curl -s http://localhost:7777/health              # Agno → {"status":"ok"}
```

## Key Decisions

| Decision | Rationale |
|---|---|
| Docker for Dify only | Dify requires 11 containers; no bare-metal option |
| No Docker for Agno | Python venv is simpler and lighter on 8 GB RAM |
| IP-based access for eval | Domain + TLS deferred to production phase |
| Redis volume permissions fix | Host-mounted dir had wrong perms; chmod 777 applied |

## Security Notes

- **No secrets in this repo.** All passwords, API keys, and tokens are redacted with `[REDACTED]`.
- Secrets live in `/opt/revvtech/dify/docker/.env` on the VPS only.
- SSH access is key-based only.

## Next Steps

- [ ] Complete Dify admin setup via `http://72.61.98.2/install`
- [ ] Create first test agent in Dify Studio
- [ ] Test Dify → Agno HTTP call (end-to-end)
- [ ] Optional: domain + TLS (Let's Encrypt)
- [ ] Optional: automated backups for Docker volumes

---

*Maintained by RevvTech. Last updated: 2026-04-08.*
