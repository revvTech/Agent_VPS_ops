# RevvTech VPS Eval Plan — Dify + Agno

Date: 2026-04-28

Scope
- Eval setup on the VPS srv1096690.hstgr.cloud for two runtimes: Dify (Docker-based orchestration/UI) and Agno Playground (Python runtime).
- Phase-based bootstrap with auditable logs, plan in PLAN.md, actions in OPS_LOG.md.
- Two-runtime approach supported: Dify for orchestration/UI; Agno for execution. Possibility to run separately or together depending on eval results.

Goals
- Establish a minimally viable platform to validate multi-tenant agent hosting, client onboarding, and end-to-end flows.
- Ensure security basics: key-based SSH, non-root admin, firewall, TLS can be added later.
- Provide a repeatable bootstrap for future evals/production.

Architecture at a glance
- Runtimes:
  - Dify: Docker Compose with 11 containers (nginx, api, web, db_postgres, redis, weaviate, sandbox, ssrf_proxy, plugin_daemon, etc.). Exposed publicly on port 80.
  - Agno Playground: Python virtual environment; serves a FastAPI app on port 7777.
- Storage:
  - Docker volumes for Dify data.
  - Local files in /opt/revvtech for Agno data and scripts.
- Connectivity:
  - MCP via Agno for tool integration; Dify calls Agno HTTP endpoints.
- Security:
  - SSH key-based admin access; root login disabled; passwords disabled for SSH.
- Domain:
  - No domain required for eval; domain TLS to be added later if needed.

Implementation Plan (Phases)
- Phase 0 — Planning & Logging
  - Create PLAN.md (this document) and OPS_LOG.md to capture decisions, milestones, and outcomes.
  - Define success criteria for the eval.
- Phase 1 — Admin & Base Tooling
  - Create admin user: louis; authorize SSH key (reuse existing key).
  - Harden SSH: disable root login; disable password auth.
  - Install Docker and docker-compose-plugin; install Nginx.
- Phase 2 — Dify Deployment (Docker)
  - Clone dify repository; configure and start via docker compose.
  - Set env vars (API keys via env or host envs).
  - Validate Dify UI and basic API health.
- Phase 3 — Agno Playground (venv)
  - Create /opt/revvtech/agno-playground; setup Python venv; install agno, fastapi, uvicorn.
  - Run agno-playground.py; ensure port 7777 accessible.
- Phase 4 — Networking & Security
  - Configure Nginx reverse proxy for Dify (port 80) and Agno Playground (port 7777).
  - Enable UFW; allow 22, 80, 443, 7777; domain TLS later if needed.
- Phase 5 — Observability & Validation
  - Basic logging: docker logs, agno logs, system logs.
  - Validate a simple end-to-end flow: Dify calls Agno endpoint; log results.
- Phase 6 — Production Readiness (optional for eval)
  - Add a domain and TLS (Let’s Encrypt).
  - Add systemd services for Agno to auto-start; backup plan for Docker volumes.

Success Criteria (Eval)
- Dify is accessible on port 80; Agno Playground on port 7777.
- A test tenant flow can be executed end-to-end with Dify calling Agno.
- Logs show decisions and avoid secrets in code.

Notes & Governance
- All changes are logged in OPS_LOG.md; PLAN.md is the source of truth.
- Any changes to architecture or multi-tenant policies require updates to PLAN.md and OPS_LOG.md.
- If uncertain, we pause and log the question in OPS_LOG.md and ask for decision.

Next steps
- Please confirm to proceed with the bootstrap script (Phase 1–2) or request any adjustments.
