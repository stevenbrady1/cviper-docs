# CViper — Architecture

**One description covering both products.** Written 2026-08-21 (CV-1322).

Before this document existed there was no single place that described the system
as it actually is. `APPLICATION_SPEC.md` describes the hosted web app as though
it were the whole thing; CViper Light's own docs describe the desktop app and
know nothing of this repository's conventions. Neither was wrong. Together they
left a gap exactly where the interesting decisions live.

Read this first, then the diagrams, then the ADRs.

---

## 1. There are two products

| | **CViper** | **CViper Light** |
|---|---|---|
| What it is | Hosted web application | Local-first desktop application |
| Repository | `github.com/stevenbrady1/CViper` | `C:\Dev\cviper` (no remote yet) |
| Stack | FastAPI + React 18 / Vite 5 + PostgreSQL | Tauri v2 + React 19 / Vite 8 + Rust + SQLite |
| Identity | Accounts, JWT, refresh cookies | None. Nothing to log into |
| Data | PostgreSQL on Azure, encrypted fields | One SQLite file on the user's disk |
| AI keys | Admin-managed and per-user, server-side | The user's own, in the OS credential store |
| Deployment | Azure Container Apps, manual deploy gate | Signed desktop installers (unsigned today) |
| Telemetry | Opt-in analytics | None, and a test enforces it |

They are **not** a front end and a back end. They are not a fork. They are two
products that share a brand and a body of hard-won domain logic.

**Diagram:** `two-product-context.svg`

### Why two

CViper is for someone who wants their job hunt on any device, with the AI
already paid for. CViper Light is for someone who will not put their CV on
somebody else's server. Those are different people, and one product cannot be
both without lying to one of them.

The decision, its costs, and the rules that follow are recorded in
[ADR-011](adr/011-two-product-architecture.md).

---

## 2. CViper — the hosted product

**Diagrams:** `system-architecture.svg`, `container-dev.svg`, `auth-rbac-flow.svg`,
`data-model.svg`, `ai-routing.svg`, `cicd-pipeline.svg`

### Request path

```
Browser
  → Cloudflare (DNS, TLS, WAF, CDN)
    → Azure Container Apps
        cviper-frontend   nginx + React SPA
        cviper-backend    Uvicorn + FastAPI
          → PostgreSQL (Azure Flexible Server, VNet private endpoint)
          → AI providers (admin keys ∪ the user's own personal keys)
```

Live counts are in [`STATS.md`](STATS.md) and are generated, never hand-written.

### Backend layering

Requests flow down. Imports flow the same way, and a guard enforces it.

```
routes/                         HTTP adapters. May import anything below.
  |
domain/  ai/  career_search/    Business logic. May import repositories,
services/  helpers/             core, utils and each other. NEVER routes.
  |
repositories/                   Database access. May import core and utils.
  |                             NEVER domain logic, NEVER routes.
core/  utils/                   Pure primitives, no I/O. Import NOTHING
                                from the stack above.
```

Measured 2026-08-21, after CV-1323:

| Layer | Files | Lines | What it is |
|---|---|---|---|
| `routes/` | 34 | 20,796 | HTTP adapters |
| `ai/` | 38 | 17,764 | the AI gateway, prompts, providers and AI services |
| `domain/` | 40 | 12,270 | business logic |
| `repositories/` | 24 | 4,803 | data access, one module per entity |
| `career_search/` | 8 | 3,133 | career-page discovery |
| `services/` | 6 | 1,027 | background jobs, alerts, push, reminders |
| `core/` | 4 | 603 | pure primitives |
| `helpers/` | 6 | 590 | infrastructure adapters |
| `utils/` | 2 | 25 | generic utilities |

Live counts are in [`STATS.md`](STATS.md) and are generated; the table above is
a shape, not a stat, and is refreshed when the shape changes.

**What each name promises.** The previous version of this section carried an
honest note saying the layer names were aspirational and `helpers/` was where
the domain actually lived. That is no longer true, and the note is gone because
the condition it described is gone — not because it was tidied away.

- **`domain/`** is the answer to "where is the business logic?". Search, ATS
  behaviour, geography, document generation, CV analysis, scoring provenance,
  advert parsing, reminder rules, registration seeding. If it knows about CVs,
  jobs, adverts or users, it is here.
- **`core/`** exists so `repositories/` has somewhere to get shared pure logic
  from without reaching upward. Country coercion, job-title identity, the
  dedupe fingerprint. No session, no request, no I/O.
- **`helpers/`** is now what its name always promised: five adapters onto
  something outside the process — blob storage, ClamAV, diagnostic redaction,
  HTTP session identity, the cloud/desktop switch. The test for adding
  something: would this module be equally at home in an unrelated product?
- **`services/`** is background work — cron sweeps, alerts, push. It is small
  because that is genuinely all it does; it is no longer small *while*
  something else holds the logic it is named for.
- **`ai/`** and **`career_search/`** are domain-tier packages that predate
  `domain/` and were deliberately left where they are. The guard treats them as
  peers, not exceptions.

**`repositories/` is a re-export surface.** `__init__.py` is 171 lines of
`from .job_repository import ...`. It used to be 2,623 lines and 102
definitions — 60% of its own layer — which is why the seven correctly-named
sibling modules read as the exception rather than the rule. Import style across
the codebase is `import repositories as repo`, and every name is still reachable
as `repo.<name>`.

**The guard, and why it is not a convention.** A layering that lives only in a
document is a diagram, and diagrams do not fail builds — which is how the
inversion above went unnoticed until an audit measured it.
`scripts/check_import_direction.py` parses every backend module with `ast`,
including function-local imports, and fails the build on an import that points
back up the stack. Its break-on-purpose tests are in
`backend/tests/infrastructure/test_import_direction.py`; each builds a synthetic
repository under `tmp_path` rather than breaking a real file, so proving the
guard works never requires sabotaging the tree it guards.

`KNOWN_VIOLATIONS` is empty and a test fails if it grows. When CV-1323 started
there were six entries — five where `repositories/` reached up into `helpers/`,
one where a background sweep imported a FastAPI router for a date helper.

History: `ClaudeReports/audits/2026-08-20-audit-full-architecture-refactor-cviper-and-light.md`
(the measurement) and
`ClaudeReports/changes/2026-08-21-change-cv1323-backend-layering-phase4.md`
(the impact analysis and the plan).

### The AI gateway

Every AI call goes through `AIGateway.call()`. Nothing constructs a second
gateway. It provides, in order:

1. Per-task provider selection, which considers the user's own keys and not only
   the admin registry
2. Retry with backoff on rate limits; immediate failure on permanent quota errors
3. **One** truncation retry at double the token budget, then a typed error
4. Provider fallback down a chain
5. Response validation against a Pydantic schema — which returns the raw dict on
   mismatch rather than raising, so a schema change never blocks a user

Sanitisation happens at the schema layer (`ai/schemas.py`), so every call site
using `validate_response` gets HTML stripping and length caps for free.

---

## 3. CViper Light — the desktop product

**Diagram:** `cviper-light.svg`

### The one idea that shapes everything

**Network calls originate in Rust, never in the web page.**

The WebView renders screens and holds no key, opens no socket, and cannot reach
the network. Everything crosses a typed Tauri IPC boundary into the Rust core,
which owns the outbound requests. An API key therefore cannot enter the
renderer, even if the renderer is compromised.

This is why there is no official Tauri keyring plugin in use: Light writes ~30
lines of Rust over the `keyring` crate directly, because the specified plugin
does not exist.

### What it stores

- **SQLite**, one file on the user's disk, via `tauri-plugin-sql` (which needs
  `features = ["sqlite"]` — the plugin is inert without it)
- **OS credential store** for API keys, via the `keyring` crate. No command reads
  a key back out; a forgotten key is replaced, not recovered

### What leaves the machine

Only what the user starts: a job search, a CV analysis against a provider they
chose, an update check they press, and a single job advert they ask to fetch.
Nothing reaches a server of ours, because there is not one.

The fetch path is deliberately narrow: one page, no cookies, no key, no link
following, and private or loopback addresses refused before the request and
again on redirect.

### Licence and distribution (ADR 012, 2026-09-06)

Light is free to use and MIT-licensed in its own public repository,
`github.com/stevenbrady1/cviper-light`; this repository stays private. Windows
and macOS ship as signed builds, Linux as an unsigned AppImage with a checksum.
Local AI requires an Ollama install in v1 (no bundled runtime). Jobserve is a
manual-search card, never a scraper. The update check reads a static, signed
`latest.json` from GitHub Releases and is the one network call the app may make
without a button press — on by default, with a visible toggle, and named as such
in Light's privacy notice. Three invariants are tests in Light's CI, not
policy: no personal data reaches an owner server, no inference is billed to the
owner, nothing is paywalled. See
[`adr/012-cviper-light-licence-and-distribution.md`](adr/012-cviper-light-licence-and-distribution.md).

### Workspace layout

```
apps/light             Tauri v2 desktop app
apps/cloud             Empty stub — declared in the workspace, contains nothing
packages/core-types    Shared domain types
packages/ai-providers  BYO-key and local Ollama adapters
packages/job-apis      Adzuna / Reed clients, keyless search links
packages/cv-parsing    CV ingestion and extraction
packages/keyword-scoring  The no-AI scorer
packages/ui            One file, 7 lines — a placeholder
```

Packages are **source-only and never built**: no `build` script, no `dist`. Vite
compiles them from source, `tsc --noEmit` typechecks them, Vitest runs them
directly.

`apps/cloud` and `packages/ui` are honest stubs, named as such in Light's own
`FEATURE-MATRIX.md`. They are workspace members that resolve to nothing.

---

## 4. What the two products share

Twelve modules were copied from CViper into CViper Light:

| Domain | CViper | CViper Light |
|---|---|---|
| Salary wording | `backend/salary_utils.py` | `packages/ai-providers/src/salary-wording.ts` |
| Dedupe fingerprint | `backend/core/dedupe_fingerprint.py` | `packages/job-apis/src/fingerprint.ts` |
| Reed salary + contract | `backend/job_sites_api.py` | `packages/job-apis/src/reed-{salary,contract}.ts` |
| Keyword scorer | `backend/ai/keywords.py` | `packages/keyword-scoring/` |
| Prompt calibration | `backend/ai/prompts/constants.py` | `packages/ai-providers/src/prompt/constants.ts` |
| Extraction prompt | `backend/ai/prompts/search_helpers.py` | `.../prompt/build-extraction-prompt.ts` |
| Extraction schema | `backend/ai/schemas.py` | `packages/core-types/src/job-extraction.ts` |
| Extraction pipeline | `backend/ai/services/search_helpers.py` | `packages/ai-providers/src/extract-job.ts` |
| JSON repair + sanitise | `backend/ai/gateway.py` | `.../extract-json.ts`, `packages/cv-parsing/src/sanitize.ts` |
| Awkward-advert fixtures | two test files | `packages/ai-providers/src/test/advert-fixtures.ts` |

### Copied, not extracted — and why

A shared package would have to be published and versioned across a Python
service and a Rust/TypeScript desktop app. That is a build and release burden
out of all proportion to twelve files.

**Copies are accepted. Undetected copies are not.**

- [`port-parity-manifest.yaml`](port-parity-manifest.yaml) pins each source's
  content hash, names the downstream file, and records whether the relationship
  is `fidelity` (must behave identically) or `divergent` (deliberately differs)
- `scripts/check_port_parity.py` fails when a pin drifts, naming what depends on it
- Each ported source carries a `PORTED-TO:` marker, so a developer sees the
  warning in the file itself
- Three modules carry **measured parity tests** — expectations produced by
  running the Python, not by predicting it

The guard proves *notification*, not correctness. It cannot check that the
TypeScript is still right; it checks that somebody was told to look.

### Where they deliberately diverge

Divergence is legitimate. `salary-wording.ts` narrows patterns that are safe on
a five-word salary field but not on a whole pasted advert — unanchored,
`excellent` matches "excellent communication skills" and `\bdoe\b` matches "John
Doe" in a signature block. It also **adds pro-rata handling that CViper does not
have at all**, and returns null for day rates rather than annualising them,
because Light's schema has no `salary_period` field and a wrong number is worse
than an empty one.

The full divergence list is in the manifest, and in the table on
`two-product-context.svg`.

---

## 5. The ingestion flow — the clearest shared path

**Diagram:** `job-ingestion-flow.svg`

Both products turn a pasted advert into a tracked job through the same six
stages:

1. **Input** — paste, upload, or (Light only) fetch one page from a link
2. **Guards** — extension, magic bytes, size cap, malware scan. All of it
   **before any AI token is spent**, so an attacker cannot spend the user's
   budget. Oversize pastes are truncated at 50,000 characters, never rejected
3. **AI extraction** — the prompt rule is *null, never guess*; the gateway
   handles retry, truncation retry and fallback; the response is schema-validated
4. **Normalise** — empty string and missing collapse into one null, so the form
   renders "the advert didn't say" identically every time
5. **Human review** — every field editable, nothing saved yet. This is the point
   of the feature
6. **Save** — now treated as ordinary *user* input, because the user may have
   edited it. Salary normalised, dedupe fingerprint computed, row written

Two invariants worth stating plainly:

- **Fail-open.** Every failure mode — no AI configured, no provider, any gateway
  exception, timeout, truncation, JSON decode error — resolves to an empty
  reviewable form and an honest banner. Never a 500. A manual save still works.
- **The pasted text is never logged.** It is third-party personal data and often
  client-confidential. Length and outcome only, in every path including failures.

---

## 6. Known gaps

Stated here so nobody has to rediscover them.

| Gap | Where |
|---|---|
| `helpers/` holds the domain logic that `services/` is named for | §2 above; audit Phase 4 |
| `App.jsx` is 5,191 lines in one function with 87 hooks | audit Phase 5 |
| `repositories/__init__.py` holds 60% of its own layer | audit Phase 4 |
| Pro-rata pay is unhandled in CViper (Light has it) | manifest, `salary_utils.py` |
| Hybrid/remote wording is captured but never parsed | `job-ingestion-flow.svg` |
| `.mmd` diagram companions are unverified and some are months stale | [`DIAGRAM_SCOPE.md`](DIAGRAM_SCOPE.md) |
| Light's `apps/cloud` and `packages/ui` are empty workspace members | §3 above |
| Light's updater is wired and tested but the signing key is a placeholder | Light's `RELEASE-SIGNING.md` |
| Nine stale agent worktrees hold 85,243 files and break local source-scanning tests | audit Phase 3 hygiene |

---

## 7. Where to look next

| Question | File |
|---|---|
| Why two products? | [`adr/011-two-product-architecture.md`](adr/011-two-product-architecture.md) |
| How Light is licensed, signed and updated | [`adr/012-cviper-light-licence-and-distribution.md`](adr/012-cviper-light-licence-and-distribution.md) |
| What is shared, and is it still in sync? | [`port-parity-manifest.yaml`](port-parity-manifest.yaml) |
| What do the diagrams have to show? | [`DIAGRAM_SCOPE.md`](DIAGRAM_SCOPE.md) |
| Every other architecture decision | [`adr/README.md`](adr/README.md) |
| Live counts | [`STATS.md`](STATS.md) |
| What CViper Light does and does not do | Light's `docs/FEATURE-MATRIX.md` |
| The full structural audit | `ClaudeReports/audits/2026-08-20-audit-full-architecture-refactor-cviper-and-light.md` |

---

# Appendix — Hosted product deep-dive

_Folded from APPLICATION_SPEC §1–§3 (AUDIT-2026-09 Phase 8). Scope is
the HOSTED product only; sections 1–7 above cover the two-product
picture. The schema inventory lives in generated
`docs/03-DATA-MODEL.md`; the API inventory in generated
`docs/02-API.md`; workflows and UI in `docs/01-PRODUCT-SPEC.md`._

## A.1 High-Level Architecture

```
                    ┌──────────────┐
                    │  Cloudflare  │  DNS/TLS (cviper.ai)
                    └──────┬───────┘
                           ▼
┌─────────────────────────────────────┐
│         React 18 SPA (Vite)         │  Port 3000
│  App.jsx + components + tabs         │
│  State-based tab navigation         │
│  SSE streaming for search results   │
└─────────────┬───────────────────────┘
              │ REST + SSE
              ▼
┌─────────────────────────────────────┐
│     FastAPI Backend (Python)        │  Port 8000
│  API endpoints, AI facade            │
│  (see docs/STATS.md for counts)     │
│  9 job board integrations           │
│  Rate limiting, JWT auth, RBAC      │
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│  PostgreSQL (Azure Flexible Server) │  Production
│  SQLite (local dev/CI)              │
│  Alembic migrations, RLS policies   │
│  JSON columns for nested data       │
└─────────────────────────────────────┘
```

**No React Router** — navigation is a single `activeTab` state variable switching between views. Tabs are registered in `config/tabs.js` with progressive disclosure tiers (focused/standard/full). Desktop TopNav groups secondary tabs into "Insights" and "More" dropdown menus. `useTabTier` hook derives the current tier from onboarding progress. WizardMode provides guided onboarding that constrains visible tabs to one at a time.

**No external UI library** — custom CSS in `theme.css` with a banking-style professional design (navy/blue primary palette).

**Fully offline-capable** — every AI feature has a keyword/template-based fallback that works without any API keys.

**Async Task Queue** — AI operations (scoring, CV analysis, document generation) are processed via an async task queue with status tracking. Tasks are queued, processed in order, and report progress. The frontend displays progress indicators and queue management UI for long-running AI operations.

**Tier System (Free / Pro / Sandbox)** — every user has a `tier` field on the `User` model with three values: `free` (default), `pro` (paid), and `sandbox` (demo). Tier drives per-operation daily quotas (`UsageLimitMiddleware`), AI token budgets (Pro 5×, Sandbox 0.5× — CV-244), and feature gating. Promotion paths: (1) **Phase 1 admin path** (CV-236) — admin promotes a user via `PATCH /api/admin/users/{id}/tier`; (2) **Phase 2 Stripe path** (scaffolding from v0.6.1, CV-238/239/240/241) — user-driven checkout via `POST /api/billing/create-checkout-session`, webhook receiver at `POST /api/billing/webhook`, nightly demotion job sweeps `tier='pro'` users whose `tier_expires_at` has passed back to `free`, plus a per-request read-time expiry guard so a paused subscription stops conferring Pro features even before the nightly run.

---

## A.2 Tech Stack

### Frontend
| Concern | Technology |
|---------|-----------|
| Framework | React 18.2 (no class components except ErrorBoundary) |
| Build | Vite 5 (port 3000, auto-open) |
| Styling | Single `theme.css` file (~4,600 lines), no CSS-in-JS |
| Tests | Vitest + @testing-library/react + jsdom |
| HTTP | Native `fetch()` with `async/await` |
| State | `useState` / `useRef` / `useMemo` (no Redux/Zustand) |

### Backend
| Concern | Technology |
|---------|-----------|
| Framework | FastAPI + Uvicorn |
| ORM | SQLAlchemy 2.0 (declarative models) |
| Database | PostgreSQL (production), SQLite (local dev/CI) |
| Migrations | Alembic with dialect-aware operations |
| AI | OpenAI, Anthropic, Google Gemini, Grok (xAI), Mistral, OpenRouter, Ollama + Pluribus (local only) |
| Scraping | requests, BeautifulSoup4 (plain self-identifying HTTP — no browser impersonation, CV-1306) |
| Documents | python-docx (DOCX), PyPDF2 (PDF read), custom PDF writer |
| Rate Limiting | slowapi |
| Auth | JWT (access + refresh tokens), bcrypt, RBAC (admin/standard/sandbox) |
| Monitoring | Prometheus custom metrics, Grafana dashboards, Loki structured logging |
| Deploy | Azure Container Apps, Bicep IaC, Cloudflare |

### Python Dependencies (requirements.txt)
```
fastapi>=0.115.0
uvicorn>=0.30.0
pydantic>=2.5.0
python-multipart>=0.0.6
PyPDF2==3.0.1
python-docx==1.1.0
openai==1.12.0
python-dotenv==1.2.2
tiktoken==0.6.0
anthropic>=0.25.0
google-genai>=1.0.0
mistralai>=1.0.0
cryptography>=42.0.0
requests>=2.31.0
beautifulsoup4>=4.12.0
lxml>=4.9.0
openpyxl>=3.1.0
slowapi>=0.1.9
sqlalchemy>=2.0.0
```

---

## A.3 Project Structure

```
cviper/
├── backend/
│   ├── main.py                    # FastAPI app setup, middleware, startup
│   ├── database.py                # SQLAlchemy models + engine setup
│   ├── repositories.py            # CRUD layer (all DB operations return dicts)
│   ├── ai_service.py              # Thin facade (~200 lines) → ai/ package
│   ├── ai/
│   │   ├── providers.py           # ProviderRegistry — 8 cloud + 2 local AI provider clients + 2 sandbox
│   │   ├── router.py              # TaskRouter — priority-based routing with fallback
│   │   ├── gateway.py             # AIGateway — universal dispatcher + JSON repair
│   │   ├── keywords.py            # KeywordService — synonym lookup, expansion
│   │   ├── fallbacks.py           # FallbackService — 18+ template fallbacks
│   │   └── services/              # 6 domain services (CV, matching, docs, scoring, etc.)
│   ├── routes/                    # Route modules (see docs/STATS.md)
│   │   ├── search.py              # Job search + SSE streaming
│   │   ├── jobs.py                # Saved jobs CRUD + scoring
│   │   ├── cv_analysis.py         # CV analysis + AI keys
│   │   ├── documents.py           # Document generation + download
│   │   ├── auth.py                # JWT auth + registration
│   │   ├── admin.py               # Admin-only endpoints (incl. PATCH /api/admin/users/{id}/tier — CV-236)
│   │   ├── billing.py             # Stripe checkout + webhook + Pro demotion (CV-238/239/240/241)
│   │   ├── companies.py           # Companies + salary estimates
│   │   ├── ai_insights.py         # Deep analysis, portfolio review
│   │   ├── feedback.py            # User feedback system
│   │   ├── gdpr.py                # Data export/deletion
│   │   ├── monitoring.py          # Health, metrics, diagnostics
│   │   └── ...                    # config, health, misc, notifications, etc.
│   ├── migrations/                # Alembic migrations (44)
│   ├── job_sites_api.py           # JobSearchAggregator + 6 board integrations
│   ├── salary_utils.py            # normalize_salary() parser
│   ├── adzuna_client.py           # Adzuna API for live salary benchmarks
│   ├── document_generator.py      # DOCX/PDF generation
│   ├── url_validator.py           # SSRF prevention
│   ├── salary_benchmarks_seed.py  # ~32 curated London IT/Finance benchmarks
│   ├── requirements.txt
│   ├── Dockerfile
│   └── tests/                     # See docs/STATS.md for current counts
│       ├── conftest.py            # In-memory SQLite + fixtures
│       └── test_*.py              # Organised by domain (ai, api, security, etc.)
├── frontend/
│   ├── src/
│   │   ├── main.jsx               # React.StrictMode → <App />
│   │   ├── App.jsx                # Main component (orchestration + state)
│   │   ├── theme.css              # Full design system
│   │   ├── components/            # Extracted UI components (see docs/STATS.md)
│   │   │   ├── ErrorBoundary.jsx, Toast.jsx, TopNav.jsx
│   │   │   ├── LoginScreen.jsx, StatusBar.jsx, TabAIIndicator.jsx
│   │   │   ├── AIProviderCard.jsx, AIMultiProviderCard.jsx
│   │   │   ├── WizardMode.jsx, CollapsibleSection.jsx
│   │   │   ├── AboutProject.jsx, CompanyDetail.jsx, Icon.jsx
│   │   │   └── ... (see docs/STATS.md for count)
│   │   ├── tabs/                  # Tab-level components
│   │   │   ├── CVAnalysisTab.jsx  # CV upload + analysis
│   │   │   ├── SearchTab.jsx      # Job search + results
│   │   │   ├── ApplicationsTab.jsx # Application tracking + Kanban
│   │   │   ├── CompaniesTab.jsx   # Company + salary management
│   │   │   ├── ConfigTab.jsx      # Settings
│   │   │   ├── CareerInsightsTab.jsx # Skills gap + strategy
│   │   │   ├── PromptsLabTab.jsx  # Multi-provider AI comparison
│   │   │   ├── FAQTab.jsx         # Searchable FAQ
│   │   │   ├── AdminTab.jsx       # Admin panel
│   │   │   ├── FeedbackAdmin.jsx  # Feedback triage
│   │   │   └── search/            # Search sub-components
│   │   ├── hooks/                 # Custom hooks
│   │   │   ├── useApi.js          # API_BASE + authFetch wrapper
│   │   │   ├── useAIProviders.js  # Provider state + retry logic
│   │   │   ├── useWizard.js       # Guided onboarding wizard state + auto-advance
│   │   │   ├── useTabTier.js      # Progressive tab disclosure tier derivation
│   │   │   ├── useOnboarding.js   # First-visit state + step tracking
│   │   │   ├── useTaskTracker.js  # Running/completed task management
│   │   │   └── useToast.js        # Toast message state
│   │   └── context/
│   │       └── AppContext.jsx     # Auth + settings + jobs context
│   ├── package.json
│   ├── vite.config.js
│   ├── Dockerfile
│   └── nginx.conf
├── azure/                         # Infrastructure as Code
│   ├── container-apps.bicep       # Container Apps, ACR, VNet, PostgreSQL
│   └── deploy.sh                  # Deployment script with secret management
├── data/                          # Local dev data (Docker volume mount)
│   └── config/
│       ├── keywords.json          # 85 skills + synonyms + job levels
│       ├── job_sites.json         # 6 job boards with enabled flags
│       ├── settings.json          # cv_folder, location, output_folder
│       └── cv_format.json         # 5 CV format presets
├── docs/
│   ├── runbooks/                  # 13 operational runbooks
│   ├── adr/                       # 9 architecture decision records
│   ├── BACKLOG.yaml               # Product backlog (synced with GitHub Issues)
│   └── ...                        # BRD, FSD, Testing Strategy, SLO, etc.
├── docker-compose.yml
├── .github/workflows/
│   ├── ci.yml                     # CI: tests, lint, security, schema drift, Docker smoke
│   └── deploy.yml                 # Manual deploy to Azure
└── .env                           # API keys (not committed)
```

---
