# Architecture Diagram Scope & Maintenance Guide

## Overview

CViper maintains a set of architecture diagrams for three audiences: technical reviewers, new developers, and non-technical stakeholders. Diagrams are displayed on the About page and used in sales collateral, pitch decks, and developer onboarding.

## Diagram Convention

| Rule | Detail |
|------|--------|
| **Format** | Hand-crafted SVG in `frontend/public/showcase/` |
| **Rendering** | Displayed on the About page via `AboutProject.jsx` |
| **Structural reference** | `.mmd` (Mermaid) files in `docs/diagrams/` — reference only, never rendered |
| **Sentinel** | Every SVG must contain `<!-- HAND-CRAFTED: Do not regenerate -->` |
| **Stats sync** | `python scripts/generate_stats.py` patches numeric stats into SVGs automatically |
| **Drift check** | `python scripts/docs_drift_check.py` verifies provider exclusions, stat accuracy, and sentinels |
| **Local-only providers** | `pluribus` and `ollama_relay` must NEVER appear as labelled boxes in showcase SVGs |
| **Auto-correction** | CLAUDE.md rules #12 (no SVG overwrite) and #16 (validation before commit) |

**Important**: Never auto-generate SVGs from `.mmd` files. The `.mmd` files are kept as structural reference for developers who want to understand the diagram topology in text form. The SVGs are hand-crafted with custom colours, layout, and styling for professional presentation.

## Diagram Registry

### Existing Diagrams (Maintained)

| # | Diagram | SVG Path | .mmd Reference | On About Page | Audience |
|---|---------|----------|----------------|---------------|----------|
| 1 | **High-Level System Architecture** | `frontend/public/showcase/system-architecture.svg` | `docs/diagrams/system-architecture.mmd` | Yes | All |
| 2 | **CI/CD Deployment Pipeline** | `frontend/public/showcase/cicd-pipeline.svg` | `docs/diagrams/cicd-pipeline.mmd` | Yes | Technical reviewers, DevOps |
| 3 | **AI Orchestration & Resilience** | `frontend/public/showcase/ai-routing.svg` | `docs/diagrams/ai-routing.mmd` | Yes | AI/ML engineers, backend devs |

| 4 | **Container & Local Dev Architecture** | `frontend/public/showcase/container-dev.svg` | `docs/diagrams/container-dev.mmd` | Yes | Developers |
| 5 | **Data Model / ERD** | `frontend/public/showcase/data-model.svg` | `docs/diagrams/data-model.mmd` | Yes | Developers, DB admins |
| 6 | **Auth & RBAC Flow** | `frontend/public/showcase/auth-rbac-flow.svg` | `docs/diagrams/auth-rbac-flow.mmd` | Yes | Security reviewers, developers |

## Diagram Content Requirements

### 1. High-Level System Architecture

**Purpose**: Big-picture overview of the entire system.

**Must show**:
- User's browser as entry point
- Cloudflare edge layer (DNS, TLS, WAF, CDN)
- Azure Container Apps environment (frontend + backend containers)
- Azure Database for PostgreSQL (production)
- AI Providers (9 cloud providers, grouped)
- Observability stack (Grafana, Prometheus, Loki)
- Request flow: User -> Cloudflare -> Frontend -> Backend API -> DB / AI Providers

**Stats to keep current** (auto-synced by `generate_stats.py`):
- Endpoint count (currently 259)
- Component count (currently 44)
- Fallback method count (currently 26)

### 2. CI/CD Deployment Pipeline

**Purpose**: Visualize the automated workflow from commit to production.

**Must show**:
- Trigger (push / PR / workflow_dispatch)
- Preflight checks (CI ancestry verification)
- Parallel test stages: Backend (pytest), Frontend (Vitest), Lint, Security SCA/SAST
- Schema drift check (PostgreSQL 16 service container)
- Integration smoke tests
- Docker build & push to ACR
- Deploy gate: `@critical-regression` Playwright tests (hard gate)
- Parallel deploy: backend + frontend Container Apps
- Post-deploy health checks
- CDN cache purge

**Stats to keep current** (auto-synced):
- Backend test file count
- Backend test approx count

### 3. AI Orchestration & Resilience Architecture

**Purpose**: Explain the AI routing, retry, fallback, and degradation strategy.

**Must show**:
- API request entry point
- Tier classification (Premium / Standard / Light)
- Provider pool per tier with examples
- Circuit breaker pattern around each provider call
- Fallback chain: Primary -> Next provider in tier -> Lower tier -> Keyword fallback
- Caching layer (15-minute TTL, content-based hash)
- Hybrid scoring (80% AI + 20% keyword)

**Stats to keep current** (auto-synced):
- Fallback template count (currently 26)

### 4. Container & Local Dev Architecture (Planned)

**Purpose**: C4 Level 3 view for developer onboarding.

**Must show**:
- **Production (Azure)**:
  - `cviper-frontend` container (React/Vite, nginx reverse proxy)
  - `cviper-backend` container (FastAPI, Gunicorn)
  - Key env vars injected (DATABASE_URL, JWT_SECRET_KEY, AI provider keys)
  - Network: Frontend -> Backend API (internal), Backend -> PostgreSQL private endpoint
  - Azure Blob Storage for generated documents
  - Key Vault for secrets
- **Local Development** (`start-app.bat`):
  - Frontend: `npm run dev` on localhost:3000
  - Backend: `python main.py` on localhost:8000
  - Database: SQLite (file-based for dev, in-memory for tests)
  - No Key Vault / Blob Storage (graceful fallback)

### 5. Data Model / ERD (Planned)

**Purpose**: Show database table relationships and the dual-database strategy.

**Must show**:
- Core tables: users, jobs, searches, companies, salary_estimates, salary_benchmarks, cv_analyses, cv_versions, search_profiles, seen_jobs, skill_trends, config
- Relationships and cardinality (users 1->N jobs, jobs 1->N cv_analyses, etc.)
- Dual-database note: PostgreSQL (production) vs SQLite (dev/CI) via SQLAlchemy dialect abstraction
- Alembic migration chain (30 migrations)
- Key constraints: partial unique index on email, user_id scoping on all user-owned tables

### 6. Auth & RBAC Flow (Planned)

**Purpose**: Document the full authentication and authorization lifecycle.

**Must show**:
- Registration flow: form -> email pre-check -> account creation -> verification email -> deep-link token -> verified
- Login flow: credentials -> JWT issuance -> `completeAuthentication` state machine -> bearer token storage
- Demo login: sandbox user creation -> session expiry timer -> auto-logout
- Request authentication: `authFetch` -> bearer header -> `AuthMiddleware` -> `get_current_user` -> role check
- Header-gated 401 interceptor: `X-Auth-Status: session-expired` triggers global logout; per-endpoint 401s are local
- RBAC roles: Admin (full access), User (own data), Demo (sandbox providers, time-limited, read-heavy)
- Post-login fetch storm: `suspendAuthInterceptorFor(5000)` grace window

## Maintenance Procedures

### When to Update Diagrams

| Trigger | Action |
|---------|--------|
| New AI provider added/removed | Update AI Routing SVG (provider boxes) |
| New endpoint batch (>10) | Run `generate_stats.py` to auto-sync endpoint count |
| New fallback method | Run `generate_stats.py` to auto-sync fallback count |
| CI pipeline stage added | Update CI/CD Pipeline SVG |
| Infrastructure change (new Azure resource, network change) | Update System Architecture SVG |
| Database table added/removed | Update ERD (when created) |
| Auth flow changed | Update Auth & RBAC Flow (when created) |

### Update Checklist (CLAUDE.md rule #16)

When editing any showcase SVG:
1. Verify connector lines/paths do NOT cross through any box they are not connecting to
2. Run `python scripts/generate_stats.py` to auto-sync numeric stats
3. Run `python scripts/docs_drift_check.py` to verify no drift
4. Verify no `LOCAL_ONLY_PROVIDERS` (pluribus, ollama_relay) appear as labelled boxes
5. Verify the `<!-- HAND-CRAFTED: Do not regenerate -->` sentinel is present

### Creating a New Diagram

1. Create the SVG in `frontend/public/showcase/` with the hand-crafted sentinel
2. Create a `.mmd` structural reference in `docs/diagrams/`
3. Add the SVG to `AboutProject.jsx` (if it should appear on the About page)
4. Add a row to the Diagram Registry table above
5. Add any auto-syncable stats to `generate_stats.py`
6. Add any drift checks to `docs_drift_check.py`
7. Update this document

## Owner

**Primary**: TDA (Technical Documentation Architect)
**Contributors**: BE (backend accuracy), FE (frontend rendering), DO (infrastructure accuracy), SEC (auth flow accuracy)
