# MSSP-Overview — Product Requirements & Deploy Notes

## Original Problem Statement
Deploy the existing MSSP/SOC analytics dashboard (FastAPI + React + MongoDB) onto Emergent
managed hosting with clean secrets and a seeded admin. Local-LLM (Ollama) requirement is
handled via the repo's self-host Docker path (documented, not managed). "Deploy as-is" — no
feature/UI changes, only deploy config (secrets, CORS, health, cleanup).

Repo pulled from: https://github.com/roh11n/MSSP-Overview.git

## User Decisions
- IRIS chat on managed = **rule-based** (Ollama only on self-host).
- Fresh start: seeded admin + demo tenants (Acme, GlobalBank), no data migration.
- Auto-generated JWT_SECRET + ADMIN_PASSWORD (shared with user).

## Architecture
- **Backend** (`/app/backend`): FastAPI, all routes under `/api`, runs 0.0.0.0:8001 (supervisor).
  Modules: server, auth (JWT+bcrypt), xsoar_ingest, ti_ingest, rules_ingest, logval_ingest,
  copilot (IRIS), llm (Ollama client w/ rule-based fallback), pptx_export, emailer, scheduler,
  recommendations, tenants, mock_data.
- **Frontend** (`/app/frontend`): React (CRA/craco), Bearer-token auth via localStorage,
  all API calls use `REACT_APP_BACKEND_URL`. 7 dashboards + IRIS copilot + Settings.
- **DB**: MongoDB via `MONGO_URL` + `DB_NAME` (from env).
- **LLM**: rule-based on managed (Ollama unreachable → graceful fallback); real local Ollama
  on the self-host docker-compose stack.

## Core Requirements (static)
- 7 dashboards: Executive, SOC Manager, Client, Detection (MITRE), Threat Intel, SOAR, Comparison.
- Live-only KPIs from uploads (XSOAR, Threat Intel, Rule Catalog, Log Validation); empty until upload.
- Snapshot + comparison (weekly/monthly/quarterly) with delta badges.
- PPTX export + scheduled email reports.
- IRIS chat (rule-based on managed).

## Implemented / Done (2026-06)
- Pulled repo into `/app`, dropped junk (zips, xlsx, test_reports, hf_cache).
- Managed wiring: `/api` prefix confirmed, `MONGO_URL`/`DB_NAME` from env, added `GET /api/health`
  (Mongo ping). CORS permissive (same-origin via ingress).
- Fresh secrets in `/app/backend/.env`: new `JWT_SECRET`, `ADMIN_EMAIL`, `ADMIN_PASSWORD`.
  Removed hardcoded admin-password fallback in `auth.py`; login UI no longer pre-fills a default.
- Seed on startup: admin (`admin@mssp-soc.io`) + demo tenants (all, acme-corp, globalbank).
- Self-host artifacts preserved: `docker-compose.yml`, Dockerfiles, `nginx.conf`, `.env.example`,
  `DEPLOYMENT.md` (Ollama `qwen2.5:1.5b` auto-pull).
- Smoke test PASSED: login, empty dashboards, upload→KPIs compute (SOC/Executive live),
  PPTX export 200, IRIS responds in rule mode. Deployment-readiness = PASS.

## Backlog / Remaining
- P1: (optional) Rewire IRIS to a hosted LLM (Claude/GPT/Gemini) for real AI on managed URL.
- P2: Seed additional role accounts (soc_manager/client/etc.) if multi-role demo needed.
- P2: Lock CORS to explicit origins if backend is ever served cross-origin.

## Next Tasks
- User presses the managed one-click **Deploy** button.
- Phase 2 (self-host): run docker-compose stack on a 4GB+ VM for local Ollama IRIS.
