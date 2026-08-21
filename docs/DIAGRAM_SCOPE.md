# Architecture Diagram Scope & Maintenance Guide

## Overview

CViper maintains a set of architecture diagrams for three audiences: technical reviewers, new developers, and non-technical stakeholders. Diagrams are displayed on the About page and used in sales collateral, pitch decks, and developer onboarding.

## Diagram Convention

| Rule | Detail |
|------|--------|
| **Source of truth** | JSON specs in `docs/diagrams/*.json` — edit these, never the SVG |
| **Generator** | `python scripts/diagram_gen.py` renders `docs/diagrams/*.json` → `frontend/public/showcase/*.svg`. Commit the JSON + SVG together |
| **Styling tokens** | `frontend/src/utils/diagram-tokens.json` — shared palette, fonts, status badges, and connector styles. The single source that keeps the whole set visually consistent |
| **Rendering** | Displayed on the About page via `AboutProject.jsx` |
| **Structural reference** | The JSON spec itself. `.mmd` companions were retired by CV-1322 — see "Why there are no `.mmd` files" below |
| **Sentinel** | Every generated SVG carries `<!-- HAND-CRAFTED: Do not regenerate — generated from docs/diagrams/<name>.json by scripts/diagram_gen.py -->` (emitted by the generator; verified by the drift check) |
| **Stats sync** | Numeric stats use `@stats.<key>` references in the JSON; `python scripts/generate_stats.py` keeps them current — never hardcode counts |
| **Drift check** | `python scripts/docs_drift_check.py` verifies provider exclusions, stat accuracy, sentinels, and orthogonal connectors |
| **Local-only providers** | `pluribus` and `ollama_relay` must NEVER appear as labelled boxes in showcase SVGs |
| **Auto-correction** | CLAUDE.md rule #57 (SVGs are generated, not hand-edited — supersedes #12→#16→#29, LESSON-081) and #58 (cache-buster chain intact) |

**Important**: Showcase SVGs are **generated** from the JSON specs — never hand-edit the SVG (direct SVG edits fail CI, rule #57). Edit `docs/diagrams/<name>.json`, run `python scripts/diagram_gen.py`, and commit the JSON + regenerated SVG together. All colours, fonts, and connector styles live in `frontend/src/utils/diagram-tokens.json` so the whole set stays visually consistent.

## Diagram Registry

### Existing Diagrams (Maintained)

| # | Diagram | SVG Path | On About Page | Audience |
|---|---------|----------|---------------|----------|
| 1 | **High-Level System Architecture** | `frontend/public/showcase/system-architecture.svg` | Yes | All |
| 2 | **CI/CD Deployment Pipeline** | `frontend/public/showcase/cicd-pipeline.svg` | Yes | Technical reviewers, DevOps |
| 3 | **AI Orchestration & Resilience** | `frontend/public/showcase/ai-routing.svg` | Yes | AI/ML engineers, backend devs |
| 4 | **Container & Local Dev Architecture** | `frontend/public/showcase/container-dev.svg` | Yes | Developers |
| 5 | **Data Model / ERD** | `frontend/public/showcase/data-model.svg` | Yes | Developers, DB admins |
| 6 | **Auth & RBAC Flow** | `frontend/public/showcase/auth-rbac-flow.svg` | Yes | Security reviewers, developers |
| 7 | **CViper Light Architecture** | `frontend/public/showcase/cviper-light.svg` | Yes | All |
| 8 | **Two-Product Context** | `frontend/public/showcase/two-product-context.svg` | Yes | All |
| 9 | **Job Ingestion Data Flow** | `frontend/public/showcase/job-ingestion-flow.svg` | Yes | Developers, product |

Diagrams 7–9 were added by CV-1322 (see [ADR-011](adr/011-two-product-architecture.md)).
No diagram has a `.mmd` companion any more — see "Why there are no `.mmd`
files" below.

**One follow-up is outstanding for 7–9, and it is a deliberate omission rather
than an oversight:**

**No visual-regression baseline.** `frontend/e2e/diagram-visual-regression.spec.js`
enumerates a hardcoded `DIAGRAMS` list, so these three are currently *uncovered*
by pixel comparison. They were left out on purpose: adding a name with no
baseline PNG fails the job on the first run, and baselines are generated on the
CI runner — a locally-produced PNG renders fonts differently and would go red
anyway.

To close it: run the `diagram-visual-regression` workflow with
`update_baselines=true`, which opens a PR containing the new baselines, then add
the three entries to `DIAGRAMS` with their viewBox dimensions:

```js
{ name: 'cviper-light',        width: 920, height: 736 },
{ name: 'two-product-context', width: 920, height: 742 },
{ name: 'job-ingestion-flow',  width: 920, height: 790 },
```

Until that is done, these three are protected by the generator's geometry
guarantees and `docs_drift_check.py`, but **not** by pixel comparison.

## Diagram Content Requirements

### 1. High-Level System Architecture

**Purpose**: Big-picture overview of the entire system.

**Must show**:
- User's browser as entry point
- Cloudflare edge layer (DNS, TLS, WAF, CDN)
- Azure Container Apps environment (frontend + backend containers)
- Azure Database for PostgreSQL (production)
- AI Providers (8 cloud providers, grouped)
- Observability stack (Azure Monitor + Log Analytics — metrics, structured JSON logs, correlation IDs)
- Request flow: User -> Cloudflare -> Frontend -> Backend API -> DB / AI Providers

**Stats to keep current** (auto-synced by `generate_stats.py`):
- Endpoint count — `@stats.endpoints`
- Component count — `@stats.components`
- Provider count — `@stats.providers`

> These were written as literal numbers ("currently 307", "currently 87") until
> CV-1322. Both had drifted — by 17 and 21 respectively — inside the very
> document whose rule table says *"never hardcode counts"*. The drift check
> validates `@stats.` references inside diagram JSON, not prose in this file, so
> nothing caught it. Cite the key, not the number: the live values are in
> [`STATS.md`](STATS.md).

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
- Fallback coverage (27 AI task operations)

### 4. Container & Local Dev Architecture

**Purpose**: C4 Level 3 view for developer onboarding.

**Must show**:
- **Production (Azure)**:
  - `cviper-frontend` container (React/Vite, nginx reverse proxy)
  - `cviper-backend` container (FastAPI, Uvicorn multi-worker — no Gunicorn; see backend/entrypoint.sh)
  - Key env vars injected (DATABASE_URL, JWT_SECRET_KEY, AI provider keys)
  - Network: Frontend -> Backend API (internal), Backend -> PostgreSQL private endpoint
  - Azure Blob Storage for generated documents
  - Key Vault for secrets
- **Local Development** (`start-app.bat`):
  - Frontend: `npm run dev` on localhost:3000
  - Backend: `python main.py` on localhost:8000
  - Database: SQLite (file-based for dev, in-memory for tests)
  - No Key Vault / Blob Storage (graceful fallback)

### 5. Data Model / ERD

**Purpose**: Show database table relationships and the dual-database strategy.

**Must show**:
- Core tables: users, jobs, searches, companies, salary_estimates, salary_benchmarks, cv_analyses, cv_versions, search_profiles, seen_jobs, skill_trends, config
- Relationships and cardinality (users 1->N jobs, jobs 1->N cv_analyses, etc.)
- Dual-database note: PostgreSQL (production) vs SQLite (dev/CI) via SQLAlchemy dialect abstraction
- Alembic migration chain (`@stats.migrations`, auto-synced)
- Key constraints: partial unique index on email, user_id scoping on all user-owned tables

### 6. Auth & RBAC Flow

**Purpose**: Document the full authentication and authorization lifecycle.

**Must show**:
- Registration flow: form -> email pre-check -> account creation -> verification email -> deep-link token -> verified
- Login flow: credentials -> JWT issuance -> `completeAuthentication` state machine -> bearer token storage
- Demo login: sandbox user creation -> session expiry timer -> auto-logout
- Request authentication: `authFetch` -> bearer header -> `AuthMiddleware` -> `get_current_user` -> role check
- Header-gated 401 interceptor: `X-Auth-Status: session-expired` triggers global logout; per-endpoint 401s are local
- RBAC roles: Admin (full access), User (own data), Demo (sandbox providers, time-limited, read-heavy)
- Post-login fetch storm: `suspendAuthInterceptorFor(5000)` grace window

### 7. CViper Light Architecture

**Purpose**: Show the desktop product, which this repository's documentation did
not acknowledge at all before CV-1322.

**Must show**:
- The machine boundary — no account, no server of ours
- WebView (React + Vite + Tailwind 4) holding no key and opening no socket
- The **Tauri IPC boundary** — the load-bearing idea: every outbound request
  starts in Rust, never in the page, which is why a key cannot reach the renderer
- Rust core commands: jobs, providers, secrets, fetch_page, updater
- SQLite (`tauri-plugin-sql`) and the OS credential store (`keyring` crate)
- The shared workspace packages, and that twelve modules are ported from CViper
- Outbound calls, all user-initiated: job boards, AI provider, Ollama, one-page
  fetch, manual update check
- What is deliberately absent: accounts, sync, telemetry, cloud storage,
  background network, link tracking, crawling

**Must NOT show**: `pluribus` or `ollama_relay` as labelled boxes (LESSON-024).
Plain "Ollama" is fine and correct — Light genuinely offers a local Ollama path
and says so publicly.

### 8. Two-Product Context

**Purpose**: The relationship between CViper and CViper Light — what is shared,
what is forked, and where the boundary sits.

**Must show**:
- Both repositories side by side, named
- Each product's data store, AI key model, and identity model
- The twelve ported modules, named
- That they are **copied, not extracted into a shared package**, and why
- That `docs/port-parity-manifest.yaml` pins them and the guard fails on drift
- The deliberate divergences: duplicates merged vs flagged, day rates
  annualised vs left null, pro-rata handled only in Light, 17 nested fields vs
  9 flat ones

**Keep in step with**: [ADR-011](adr/011-two-product-architecture.md). If the
divergence table here and the manifest disagree, the manifest wins.

### 9. Job Ingestion Data Flow

**Purpose**: How something a user pastes becomes a tracked job. The diagram set
previously had an ERD (structure at rest) but no data flow.

**Must show**:
- The six stages: input → guards → AI extraction → normalise → human review → save
- That **guards run before any AI token is spent**
- That oversize pastes are truncated, never rejected
- The null-never-guess prompt rule
- The fail-open contract: every failure yields an empty reviewable form, never a 500
- That the raw pasted text is never logged
- That nothing persists before the user reviews it
- Which stages are shared with CViper Light

**Must stay honest about**: the awkward-advert table at the foot. If pro-rata
handling lands upstream, or work-mode extraction is added, that table changes.

## Why there are no `.mmd` files

Diagrams once carried a `.mmd` (Mermaid) companion, described as an optional
human-readable topology reference. **CV-1322 retired all eight of them.** The
decision is recorded here because "where did the Mermaid files go?" is a
reasonable question to ask later.

Every claim made for them had stopped being true. Measured 2026-08-21:

| Problem | Evidence |
|---------|----------|
| **Duplicated, and the copies disagreed** | `ai-routing.mmd` and `cicd-pipeline.mmd` existed in BOTH `docs/diagrams/` and `frontend/public/showcase/`, with different content |
| **The guard watched the wrong copies** | `check_diagram_sources()` asserted existence under `frontend/public/showcase/` — a location this document never mentioned. The documented copies were unguarded |
| **Only existence was ever checked** | Nothing compared a `.mmd` to its `.json`. `data-model.mmd` named **22 entities against a 48-table schema** and was four months behind its source |
| **The headers were false** | Every file opened with *"The showcase SVG is hand-crafted"*. Rule #57 made that untrue — the SVGs are generated, and hand edits fail CI |
| **Two were published** | `frontend/public/` ships verbatim in the Vite build, so `/showcase/ai-routing.mmd` served internal topology on the public site |

A `.mmd` four months behind its JSON is worse than no file: it is a confident,
readable, wrong answer sitting next to the right one.

**Regenerating them from the JSON was considered and rejected.** The
box/connection spec carries no ER semantics, so a generated `data-model.mmd`
would have been strictly worse than the hand-written one it replaced. The JSON
specs are readable, they are the source of truth, and they are the only thing
verified.

**The replacement is a forbid-list** (LESSON-033):
`backend/tests/infrastructure/test_diagram_mmd_retired.py` fails if any `.mmd`
reappears under the diagram or showcase trees, and — the wider rule the
published copies revealed — if anything unpublishable is added to
`frontend/public/showcase/`.

If a text topology reference is ever wanted again, generate it from the JSON in
the same commit as the SVG, so it cannot drift. Do not hand-write one.

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

### Update Checklist (CLAUDE.md rule #57)

When changing any diagram, edit the JSON spec (never the SVG), then:
1. Run `python scripts/diagram_gen.py` to regenerate the SVG(s)
2. Verify connector lines/paths do NOT cross through any box they are not connecting to
3. Run `python scripts/generate_stats.py` to sync numeric stats (if counts changed)
4. Run `python scripts/docs_drift_check.py` to verify no drift
5. Verify no `LOCAL_ONLY_PROVIDERS` (pluribus, ollama_relay) appear as labelled boxes
6. Commit the JSON spec + regenerated SVG together

### Creating a New Diagram

1. Write the JSON spec in `docs/diagrams/<name>.json` (reuse styles/connectors from `diagram-tokens.json`)
2. Run `python scripts/diagram_gen.py <name>` to generate `frontend/public/showcase/<name>.svg`
4. Add the SVG to `AboutProject.jsx` (if it should appear on the About page)
5. Register it in `frontend/e2e/diagram-visual-regression.spec.js` (`DIAGRAMS`) and capture baselines
6. Add a row to the Diagram Registry table above
7. Add any auto-syncable stats to `generate_stats.py` and any drift checks to `docs_drift_check.py`
8. Update this document

## Owner

**Primary**: TDA (Technical Documentation Architect)
**Contributors**: BE (backend accuracy), FE (frontend rendering), DO (infrastructure accuracy), SEC (auth flow accuracy)
