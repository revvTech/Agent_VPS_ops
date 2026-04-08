# RevvTech OPS Log — VPS Eval

## Phase 0 — Prep (2026-04-08)
- [x] PLAN.md created
- [x] OPS_LOG.md created
- [x] Two-runtime architecture confirmed (Dify + Agno)
- [x] SSH key reused from prior setup

## Phase 1 — Admin & Base Tooling (2026-04-08 ~09:08 UTC)
- [x] Admin user `louis` created with sudo + docker group
- [x] SSH key added to `/home/louis/.ssh/authorized_keys`
- [x] Docker 29.4.0 + Compose v5.1.1 installed
- [x] Python 3.12.3, git 2.43.0 installed
- [x] SSH login as `louis` verified
- Decision: skip SSH hardening (disable root) for now — root still needed for docker ops

## Phase 2 — Dify Deployment (2026-04-08 ~09:15 UTC)
- [x] Dify cloned to `/opt/revvtech/dify` from `github.com/langgenius/dify`
- [x] `.env` configured with auto-generated secrets
- [x] `docker compose up -d` — 11 containers started
- [x] Containers: api, web, nginx, db_postgres, redis, weaviate, sandbox, ssrf_proxy, plugin_daemon, worker, worker_beat
- [x] Nginx exposed on port 80
- Credentials generated:
  - INIT_PASSWORD=[REDACTED]
  - SECRET_KEY stored in `/opt/revvtech/dify/docker/.env`

### Redis MISCONF Fix (2026-04-08 ~09:44 UTC)
- Issue: Redis couldn't persist RDB snapshots — `Permission denied` on `./volumes/redis/data`
- Root cause: host-mounted volume dir had wrong permissions
- Fix: `chmod 777 ./volumes/redis/data` + `chmod 777 ./volumes/db/data` + full stack restart
- Result: Redis healthy, RDB loading from disk, API migrations complete
- Endpoints verified:
  - `http://72.61.98.2/console/api/setup` → 200
  - `http://72.61.98.2/console/api/system-features` → 200

## Phase 3 — Agno Playground (2026-04-08 ~09:17–09:37 UTC)
- [x] Python venv created at `/opt/revvtech/agno-playground/venv`
- [x] `agno[os]`, `mcp`, `fastapi`, `uvicorn`, `openai` installed
- [x] Initial stub deployed — failed (missing `agno.playground` module, deprecated API)
- [x] Rewrote to use `agno.os.AgentOS` (current API per docs.agno.com)
- [x] Real `agno_playground.py` deployed with 3 agents:
  - RevvTech Assistant
  - DDG Outreach Agent
  - DDG Data Acquisition Agent
- [x] systemd service `agno-playground` created, enabled, started
- [x] Service runs via uvicorn on port 7777
- Verified:
  - `http://72.61.98.2:7777/health` → `{"status":"ok"}`
  - `http://72.61.98.2:7777/` → AgentOS API info

## Phase 4 — Firewall (2026-04-08 ~09:17 UTC)
- [x] UFW enabled
- [x] Ports allowed: 22, 80, 443, 7777
- [x] `ufw status` confirmed active

## Current State (2026-04-08 11:53 SAST)

| Service | URL | Status |
|---|---|---|
| Dify UI | http://72.61.98.2 | ✅ Running |
| Dify API | http://72.61.98.2/console/api/setup | ✅ 200 |
| Agno Playground | http://72.61.98.2:7777 | ✅ Running |
| Agno Health | http://72.61.98.2:7777/health | ✅ OK |

## Files Touched
- `/opt/revvtech/dify/` — Dify repo + docker config
- `/opt/revvtech/dify/docker/.env` — secrets, passwords
- `/opt/revvtech/dify/docker/volumes/redis/data` — permissions fixed
- `/opt/revvtech/dify/docker/volumes/db/data` — permissions fixed
- `/opt/revvtech/agno-playground/agno_playground.py` — real AgentOS code
- `/opt/revvtech/agno-playground/venv/` — Python venv
- `/etc/systemd/system/agno-playground.service` — systemd unit
- Local: `~/code/revvtech-workspace/internal/agent-management-platform/deploy/PLAN.md`
- Local: `~/code/revvtech-workspace/internal/agent-management-platform/deploy/OPS_LOG.md`
- Local: `~/code/revvtech-workspace/internal/agent-management-platform/deploy/VPS-AGENT.md`

## Next Steps
- [ ] Complete Dify admin setup via http://72.61.98.2/install
- [ ] Create first test agent in Dify Studio
- [ ] Test Dify → Agno HTTP call (end-to-end validation)
- [ ] Optional: domain + TLS (Let's Encrypt)
- [ ] Optional: SSH hardening (disable root login)
- [ ] Optional: automated backups for Docker volumes
