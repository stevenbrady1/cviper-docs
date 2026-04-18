# Functional Specification Document (FSD)

## CViper - AI-Powered Job Search & Application Platform

---

| Field | Value |
|-------|-------|
| **Document ID** | FSD-CVIPER-001 |
| **Version** | 0.5.2 |
| **Status** | Pre-Release |
| **Author** | CViper Project Team |
| **Date** | 2026-04-17 |
| **Classification** | Internal |
| **Related BRD** | BRD-CVIPER-001 v0.5.2 |

### Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.5.2 | 2026-04-17 | CViper Project Team | Version bump (commit: 7baf4b18). Update this description with change details. |
| 0.5.1 | 2026-04-13 | CViper Project Team | Career Intelligence UI, UK salary comparison, legal markdown rendering, provider settings redesign, progressive disclosure, wizard mode, E2E guards. All doc versions aligned to 0.5.1. |
| 0.3.2 | 2026-04-10 | CViper Project Team | CV Optimisation Pipeline (keyword injection, ATS format validator, one-click optimise). Training Provider Foundation (8 providers, certifications, skill progress). AI Ethics & Fairness (prompt guardrails, confidence scores, Challenge This Score). Growth Readiness (OG/Twitter meta, robots.txt, sitemap.xml, PWA install, Plausible). Cross-user Data Isolation (3-layer: backend scoping, frontend reset, userStorage). API Contract Tests (14 tests, shared schema). P0 security verified. |
| 0.3.1 | 2026-04-07 | CViper Project Team | Phase 0 security hardening: encrypted `.env` secrets, SecurityHeadersMiddleware, fatal guards for SECURE_COOKIES + CORS wildcards, all `shell=True` removed from backend (AST ratchet test). OAuth extended to LinkedIn + Google + Microsoft. `?tab=` deep links for external URLs. Terms of Service published. Public-route registry (single source of truth for auth middleware). Stale chunk prevention (nginx cache headers + lazyRetry + Cloudflare purge). PR-based backlog sync. News Feed full-width fix. |
| 0.2.3 | 2026-03-27 | CViper Project Team | Sandbox Gemini quota fix (permanent quota error detection, force_open circuit breaker). Alembic migration 013 batch fix (SQLite batch_alter_table compatibility). CI PostgreSQL service container for schema drift checks. |
| 0.2.2 | 2026-03-27 | CViper Project Team | Version reset to align with application semver (pre-release). Consolidates all prior work (formerly v1.0–v2.2). Full history archived in `docs/Archive/`. |

> **Note:** Versions prior to 0.2.2 used an independent numbering scheme (v1.0–v2.2). Those documents are preserved in `docs/Archive/` for reference. From this version onward, document versions track the application version in `package.json`.

---

### What's New in v0.3.2

- **CV Optimisation Pipeline (FR-033)**: Keyword injection suggestions identify missing ATS keywords from job descriptions. ATS format validator checks CV structure against compatibility rules. One-click optimise-for-job automates CV tailoring for a specific role.
- **Training Provider Foundation (FR-034)**: 8 curated providers (4 free: freeCodeCamp, Coursera Audit, edX Audit, Khan Academy; 4 paid: Udemy, Pluralsight, LinkedIn Learning, Codecademy Pro). Certification mapping to skills. Skill progress tracking via Skills & Training tab.
- **AI Ethics & Fairness (FR-035)**: Fairness guardrails injected into all AI scoring prompts (no bias on name, age, gender, ethnicity). Confidence scores displayed on all AI outputs. "Challenge This Score" button for user-initiated re-evaluation. AI transparency disclosure explaining scoring methodology.
- **Growth Readiness (FR-036)**: Open Graph and Twitter Card meta tags for social sharing. robots.txt and sitemap.xml for SEO. PWA install prompt for mobile users. Plausible analytics for privacy-respecting usage tracking.
- **Cross-user Data Isolation (FR-037)**: 3-layer prevention — backend `user_id` scoping on all repo functions, frontend state reset on user switch, `userStorage` namespacing (`cviper:u:<userId>:<key>`) with purge on switch.
- **API Contract Tests (FR-038)**: 14 contract tests with shared schema file validating response shapes across major endpoints. Hard CI gate.
- **Security Verification (FR-039)**: All P0 security items verified — CSP headers, CORS guards, secret encryption, Terms of Service, subprocess hardening (AST ratchet test).

### What's New in v0.3.1

- **Demo Mode UX Redesign**: Landing page reordered (demo first, API key guide, login below). Guided tour available on demo launch. Prompt Lab visible in read-only mode for demo users.
- **API Key Guide**: Step-by-step walkthrough for Google Gemini, GitHub Models, Ollama, OpenAI, and Anthropic — in FAQ, Settings, and on the landing page.
- **AI Disclaimer**: All AI results attributed to the user's chosen model with accuracy warnings. Privacy Policy updated to v1.1 with Section 5 (AI-Generated Content Disclaimer).
- **Infrastructure Resilience**: Pre-deploy config validation gate (33 checks), infrastructure dependency map, server-side path redaction, custom domain binding verification.
- **ATS Discovery**: iCIMS and Taleo handlers added. Concurrent discovery pipeline with board health monitoring.
- **PWA**: Service worker with stale-while-revalidate caching. SVG diagram sentinel protection prevents Mermaid overwrite regression.
- **Test Suite**: 3,540 backend + 817 frontend + 41 E2E tests (4,400+ total). Icon completeness test prevents missing icon text fallback.
- **17 Lessons Learned**: Automated prevention guards for all infrastructure failure modes (LESSON-001 through LESSON-017).

### What's New in v0.2.3

- **Sandbox Gemini Quota Fix**: AIGateway now detects permanent quota errors (Google RESOURCE_EXHAUSTED with "quota") and skips retries. Circuit breaker `force_open()` immediately opens on quota exhaustion, bypassing the normal failure-count threshold. Concise user-facing error message guides sandbox users toward account creation.
- **Migration 013 Batch Fix**: Alembic migration 013 updated to use `batch_alter_table()` context manager for SQLite compatibility (copy-and-move strategy instead of direct ALTER TABLE).
- **CI PostgreSQL Service**: GitHub Actions CI now runs a PostgreSQL 16 service container for schema drift validation (`alembic upgrade head` + `alembic check`) instead of relying on SQLite for migration testing.

### What's New in v0.2.2

This version consolidates all prior pre-release work (formerly documented as v1.0–v2.2) under a single semver-aligned version. Key capabilities included in this baseline:

- **RBAC Audit & Hardening (FR-032)**: Service control endpoints enforce admin-only access; full permission matrix in Section 3.1
- **Guided Base CV Bullet Optimization (FR-031)**: AI-powered CAR-pattern rewriting via `POST /api/cv/optimize-bullets`
- **Hybrid JWT/OAuth Authentication (FR-027–FR-030)**: JWT access tokens + httpOnly refresh cookies, LinkedIn/Google OAuth2, per-session sandbox with Alex Morgan persona, AuthContext provider, SandboxBanner/ExampleBadge/SignUpPrompt UI components
- **Full feature set**: 10 enhancement phases, Azure deployment, GDPR compliance, sandbox abuse prevention, mobile rendering, multi-provider AI routing, salary benchmarks, document generation (see `docs/STATS.md` for current test counts)

> For detailed change-by-change history, see the archived documents in `docs/Archive/`.

---

## 1. Introduction

### 1.1 Purpose

This document provides the complete functional specification for CViper, detailing system architecture, functional requirements, API specifications, data models, security architecture, UI specifications, and test strategy. It is written for architects, developers, and QA engineers.

### 1.2 System Overview

CViper is a full-stack web application consisting of:

- **Frontend**: React 18 SPA (Vite + SWC) — 7 main tabs + 1 admin tab (2 hidden by default via Advanced Mode), components including ProgressStepper (7-step registration / 4-step workflow). See `docs/STATS.md` for current counts.
- **Backend**: Python FastAPI server — route modules with REST endpoints, PostgreSQL (production) / SQLite (dev/CI). See `docs/STATS.md` for current counts.
- **AI Layer**: Multi-provider AI service with 9 supported providers, priority-based routing with automatic failover
- **Scraping Layer**: 11 ATS handler classes + Adzuna affiliate API for career page and job board extraction
- **Monitoring**: Structured JSON logging, Prometheus metrics, Grafana dashboard integration, OpenTelemetry distributed tracing

### 1.3 References

| Document | Description |
|----------|-----------|
| BRD-CVIPER-001 | Business Requirements Document |
| CLAUDE.md | Development guidelines and coding standards |
| .github/workflows/ci.yml | CI/CD pipeline configuration |
| azure/container-apps.bicep | Azure production infrastructure (Bicep IaC) |
| azure/deploy.sh | Azure deployment automation script |
| .github/workflows/deploy.yml | Manual production deployment workflow |

---

## 2. System Architecture

### 2.1 High-Level Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        FRONTEND (React 18 + Vite)               │
│  ┌──────┐ ┌──────────────┐ ┌──────────┐ ┌─────────┐ ┌────────────┐  │
│  │TopNav│ │ProgressStep │ │SearchTab │ │AppsTab  │ │CompanyTab  │  │
│  └──────┘ └──────────────┘ │  Form    │ │  Cards  │ │  Salaries  │  │
│                      │  Filters │ │  Kanban │ │  Benchmarks│  │
│                      │  Results │ │  Docs   │ │            │  │
│                      └──────────┘ └─────────┘ └────────────┘  │
│  ┌──────────┐ ┌───────────┐ ┌────────────┐ ┌──────────────┐   │
│  │ConfigTab │ │Monitoring │ │PromptLab  │ │  AdminTab    │   │
│  │ Settings │ │  Panel    │ │  Compare  │ │  Users/DB    │   │
│  │ AI Keys  │ │  Health   │ │  Templates│ │  (Root only) │   │
│  └──────────┘ └───────────┘ └────────────┘ └──────────────┘   │
│                                                                 │
│  Hooks: useApi (cache/dedup) | useAIProviders | useTaskTracker │
└───────────────────────────┬─────────────────────────────────────┘
                            │ HTTP/SSE (fetch + credentials)
                            │ Base: VITE_API_BASE (default localhost:8000/api)
                            v
┌─────────────────────────────────────────────────────────────────┐
│                     BACKEND (FastAPI + Uvicorn)                  │
│                                                                  │
│  ┌─── Middleware ────────────────────────────────────────────┐   │
│  │ CORS | Auth | Rate Limiting | Correlation ID | Logging   │   │
│  └──────────────────────────────────────────────────────────-┘   │
│                                                                  │
│  ┌─── Routes (12 modules) ──────────────────────────────────┐   │
│  │ health | auth | jobs | search | cv_analysis | documents  │   │
│  │ ai_insights | companies | config | monitoring | prompts  │   │
│  │ admin | misc                                             │   │
│  └────────────────────────┬─────────────────────────────────┘   │
│                           │                                      │
│  ┌─── Business Logic ────-┤─────────────────────────────────┐   │
│  │                        v                                  │   │
│  │  ┌──────────────────────────────────┐                    │   │
│  │  │        AIService (Facade)        │                    │   │
│  │  │  50+ methods, backward compat    │                    │   │
│  │  └──────────┬───────────────────────┘                    │   │
│  │       ┌─────┴──────┬──────────┬──────────┬──────────┐    │   │
│  │       v            v          v          v          v    │   │
│  │  CVAnalysis  JobMatching  DocumentGen  Scoring  Career   │   │
│  │  Service     Service      Service      Service  Insights │   │
│  │       └─────┬──────┴──────────┴──────────┴──────────┘    │   │
│  │             v                                             │   │
│  │  ┌──────────────────┐  ┌──────────────┐                  │   │
│  │  │   AIGateway      │  │  TaskRouter  │                  │   │
│  │  │   (Dispatcher)   │  │  (Selection) │                  │   │
│  │  └────────┬─────────┘  └──────┬───────┘                  │   │
│  │           └──────────┬────────┘                           │   │
│  │                      v                                    │   │
│  │           ┌─────────────────────┐                        │   │
│  │           │  ProviderRegistry   │                        │   │
│  │           │  8 AI Clients       │                        │   │
│  │           └─────────────────────┘                        │   │
│  │                                                           │   │
│  │  ┌──────────────────────────────────────────────────┐    │   │
│  │  │  Career Search Engine                            │    │   │
│  │  │  11 ATS Handlers + HTML Scraper + Adaptive Parse │    │   │
│  │  └──────────────────────────────────────────────────┘    │   │
│  └───────────────────────────────────────────────────────────┘   │
│                                                                  │
│  ┌─── Data Layer ───────────────────────────────────────────────┐│
│  │ SQLAlchemy ORM | PostgreSQL (prod) / SQLite (dev) | Repos  ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
                            │
              ┌─────────────┼─────────────┐
              v             v             v
     ┌──────────────┐ ┌─────────┐ ┌─────────────────────────────────┐
     │ AI Providers │ │ Job     │ │ Infrastructure Services         │
     │ OpenAI       │ │ Boards  │ │                                 │
     │ Anthropic    │ │ Reed    │ │ ┌─────────────────────────────┐ │
     │ Google       │ │ Indeed  │ │ │ Grafana (Docker :3001)      │ │
     │ OpenRouter   │ │ LinkedIn│ │ │ - Infinity datasource       │ │
     │ Mistral      │ │ CWJobs  │ │ │ - cviper-unified dashboard  │ │
     │ Grok (xAI)   │ │ etc.   │ │ │ - Auto-provisioned on start │ │
     │ GitHub       │ └─────────┘ │ └─────────────────────────────┘ │
     │ Pluribus *   │             │  * local gateway only           │
     └──────────────┘             │ ┌─────────────────────────────┐ │
              │                   │ │ Ollama (Docker :11434)      │ │
              │                   │ │ - Model: llama3.2 (default) │ │
              └───────────────────│ │ - OpenAI-compatible API     │ │
               (Ollama is both    │ │ - Auto-pull on first start  │ │
                an AI provider    │ │ - CPU/GPU configurable      │ │
                AND infra svc)    │ └─────────────────────────────┘ │
                                  └─────────────────────────────────┘
```

### 2.2 Technology Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| **Frontend Framework** | React | 18.2.0 |
| **Frontend Build** | Vite + SWC | 5.0.8 |
| **Frontend Testing** | Vitest + React Testing Library | 2.1.9 |
| **Frontend Linting** | ESLint + Prettier | 8.56.0 / 3.2.0 |
| **Backend Framework** | FastAPI | Latest* |
| **Backend Runtime** | Python | 3.12 |
| **ASGI Server** | Uvicorn | Latest* |
| **Database (Production)** | Azure Database for PostgreSQL Flexible Server | Burstable B1ms, UK South |
| **Database (Local/Test)** | SQLite (WAL mode / in-memory) | 3.x |
| **Database (CI Schema Drift)** | PostgreSQL (service container) | 16 |
| **ORM** | SQLAlchemy | Latest* |
| **Backend Testing** | pytest + pytest-xdist | Latest* |
| **Rate Limiting** | slowapi | Latest* |
| **AI SDKs** | openai, anthropic, google-generativeai, mistral | Latest* |
| **Monitoring** | Grafana + Infinity Datasource | 12.x |
| **CI/CD** | GitHub Actions | Latest |
| **Deployment** | Azure Container Apps (Bicep IaC) | N/A |
| **Local AI** | Ollama | Latest* |

*\* "Latest" = latest stable release as of March 2026; pinned versions managed via requirements.txt and package.json*

---

## 3. Security Architecture

### 3.1 Authentication & Authorization

#### Authentication Flow (Hybrid JWT + Session)

```
[User] ──> POST /api/auth/login {username, password}
               │
               v
       [AuthMiddleware]
       Extract token from:
       1. Header: Authorization: Bearer <JWT>  → decode JWT claims (stateless)
       2. Header: Authorization: Bearer <session_token> → DB lookup (legacy fallback)
       3. Cookie: session_token (httponly) → DB lookup (legacy fallback)
               │
               v
       [Inject request.state.current_user]
       User object available to all route handlers
```

#### OAuth2 Social Login Flow

```
[User] ──> GET /api/auth/linkedin (or /google)
               │
               v
       [Server redirects to provider authorization page]
               │
               v
       [User authenticates with provider]
               │
               v
       [Provider redirects to /api/auth/linkedin/callback]
               │
               v
       [Server exchanges code for token, extracts profile]
       [Creates/updates user, issues JWT + refresh token]
               │
               v
       [Redirect to frontend: /?oauth_token=<JWT>]
       [Set httpOnly refresh_token cookie]
```

#### Password Security

| Aspect | Implementation |
|--------|---------------|
| Hashing | bcrypt with per-user salt |
| Storage | `password_hash` + `salt` columns on User table |
| Verification | `bcrypt.checkpw(password, stored_hash)` |
| Session Token | `secrets.token_urlsafe(32)` — 43-char random |
| Session TTL | 24 hours (default), 30 days (remember_me) |
| Session Storage | `UserSession` table (DB-backed, not in-memory) |
| Idle Timeout | Configurable via `SESSION_IDLE_TIMEOUT_MINUTES` (default: 60); sessions inactive beyond threshold are deleted |
| Secure Cookies | `Secure` flag enabled via `SECURE_COOKIES` env var for HTTPS deployments |
| Account Lockout | 10 failed login attempts within 15 minutes triggers temporary lockout (HTTP 429). Tracked per-username (case-insensitive). Clears on successful authentication |
| Password Reset | Forgot password flow via email token: `POST /api/auth/forgot-password` generates token, `POST /api/auth/reset-password` validates token and sets new password |
| JWT Access Token | `python-jose` HS256 signed, 15-min expiry (1-hr for sandbox) |
| JWT Refresh Token | `secrets.token_urlsafe(64)`, SHA-256 hashed before DB storage, 7-day expiry, single-use (rotated) |
| OAuth | authlib with OpenID Connect (LinkedIn) and OAuth2 (Google); server-side token exchange |

#### Role-Based Access

| Role | Identity Flags | Permissions |
|------|---------------|------------|
| `root` (Admin) | `role="root"`, `is_admin=True` | Full access: CRUD users, view all data, admin endpoints, service control, system config, global AI metrics |
| `user` (Standard) | `role="user"`, `is_sandbox=False` | Own data only: own jobs, searches, configs, CV analyses, documents, own AI metrics. No admin or service control access |
| `user` (Sandbox) | `role="user"`, `is_sandbox=True`, `provider="sandbox"` | Ephemeral (1-hour TTL). Truncated AI output, no DELETE operations, no admin/service control, no password/privacy settings, restricted AI providers (sandbox_google, sandbox_openrouter, keyword) |

**RBAC Enforcement**: Admin endpoints use router-level `dependencies=[Depends(require_root)]` on `APIRouter`. Service control endpoints (`/api/service/*`) also use route-level `Depends(require_root)`. The `require_root` dependency in `auth.py` validates `user.is_admin == True` or `user.role == "root"` and emits a structured `rbac_denial` log event on 403. Sandbox users are additionally blocked by `_is_sandbox_restricted` middleware for `/api/admin/*`, `/api/service/*`, `/api/ollama/*`, `/api/pluribus/*`, and all DELETE operations.

#### RBAC Permission Matrix

| Feature / Endpoint | Admin (root) | Standard User | Sandbox User | Enforcement Layer |
|---|:---:|:---:|:---:|---|
| **Screens** | | | | |
| CV Analysis | Full | Full | Truncated output | Backend middleware |
| Job Search | Full | Full | Truncated output | Backend middleware |
| Applications | Full | Full | No delete | Frontend + backend MW |
| Company Salaries | Full | Full | Full | Data isolation |
| Settings (Config) | Full | Full | Partial (no password, privacy, interface) | Frontend + backend 403 |
| Monitoring tab | Always visible | Advanced Mode toggle | Advanced Mode toggle | Frontend |
| Prompt Lab tab | Always visible | Advanced Mode toggle | Advanced Mode toggle | Frontend |
| Admin tab | Visible | Hidden | Hidden | Frontend + backend router |
| **Admin Sub-sections** | | | | |
| User Management | Full CRUD | 403 | 403 | `require_root` router dependency |
| Service Control | Full | 403 | 403 | `require_root` route + MW |
| Job Board Defaults | Full | 403 | 403 | `require_root` router dependency |
| Career Page Defaults | Full | 403 | 403 | `require_root` router dependency |
| Database Browser | Full | 403 | 403 | `require_root` router dependency |
| **Data Operations** | | | | |
| DELETE operations | Full | Full | Blocked | Backend middleware |
| Cross-user data access | Full (all users) | Own data only | Own data only | Repository `user_id` filter |
| **AI Provider Access** | | | | |
| All providers | Visible | All except local-only and sandbox_ prefixed | sandbox_google, sandbox_openrouter, keyword only | `provider_visibility.py` |
| Ollama / Pluribus (local only) | Full | Full | Blocked | Backend MW + frontend |
| **Infrastructure** | | | | |
| `/api/service/*` | Full | 403 | 403 | `require_root` route dependency |
| `/api/ollama/*` | Full | Full | Blocked | Backend middleware |
| `/api/admin/*` | Full | 403 | 403 | `require_root` router dependency |

#### Per-User API Key Resolution

Users can configure their own AI provider API keys stored in the `Config` table under `user_provider_keys`. The `UserKeyResolver` (`ai/user_keys.py`) resolves keys per-request:

```
User Key (encrypted in DB) → Decrypt → Build ephemeral client
  OR
Server Default Key → Use shared client from ProviderRegistry
```

Resolution is cached per `{user_id}:{provider}` with a 5-minute TTL.

#### Token Budget Enforcement

Per-user daily AI token limits enforced at the `AIGateway.call()` level:

| Setting | Default | Description |
|---------|---------|-------------|
| `USER_DAILY_TOKEN_LIMIT` | 100,000 | Daily token cap per user (0 = disabled) |
| Root Exemption | Yes | Root users bypass budget checks |
| Reset | UTC midnight | Budget resets based on `AIPromptLog.created_at` |
| Warning | 80%+ usage | Structured `token_budget_warning` event emitted |
| Exceeded | 100%+ usage | `TokenBudgetExceeded` exception → 429 response |

#### Public Endpoints (No Auth Required)

```
GET  /api/health
GET  /api/health/detailed
POST /api/auth/login
POST /api/auth/register
POST /api/auth/setup
GET  /api/auth/status
GET  /api/job-categories
GET  /api/onboarding/career-pages
POST /api/auth/forgot-password
POST /api/auth/reset-password
GET  /api/version
POST /api/auth/sandbox
GET  /api/auth/providers
GET  /api/auth/linkedin
GET  /api/auth/linkedin/callback
GET  /api/auth/google
GET  /api/auth/google/callback
POST /api/auth/refresh
GET  /api/auth/me
GET  /api/auth/check-username/{username}
```

### 3.2 Data Encryption

| Data | At Rest | In Transit |
|------|---------|-----------|
| Passwords | bcrypt hash + salt | HTTPS |
| Server AI API Keys | AES encryption (key_encryption.py) for `.env` values | HTTPS to providers |
| User AI API Keys | Fernet symmetric encryption (`ENC:` prefix) with server master key; supports key rotation via `rotate_user_keys()` | HTTPS to providers |
| CV Profiles in Search | encrypt_json() / decrypt_json() | HTTPS |
| Session Tokens | SHA-256 hashed before DB storage (cryptographically random plaintext in cookie) | httponly cookie |
| Database | SQLite file on disk (OS-level encryption if needed) | N/A (local) |

### 3.3 SSRF Prevention

**Module**: `url_validator.py`

```python
validate_url(url, strict_allowlist=False) -> (bool, str)
```

| Check | Details |
|-------|---------|
| Scheme | Only `http://` or `https://` |
| Hostname | Must be non-empty |
| Blocked IPs | 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16, 127.0.0.0/8, ::1, 169.254.0.0/16, fe80::/10, fc00::/7 |
| DNS Resolution | Resolve hostname, check resolved IP against blocked ranges |
| Allowlist | Optional KNOWN_JOB_BOARDS set for strict validation |

### 3.4 Rate Limiting

**Library**: slowapi

| Endpoint Group | Limit | Key |
|---------------|-------|-----|
| Job Search | 10/minute | user_id or IP |
| AI Scoring/Analysis | 30/minute | user_id or IP |
| Login | 5/minute | IP |
| Document Generation | 30/minute | user_id or IP |
| Health Checks | Exempt | N/A |

### 3.5 Prompt Injection Prevention

**Module**: `ai/gateway.py` — `sanitize_for_prompt(text)`

Patterns stripped before AI calls:
- `ignore|disregard|forget previous|above|prior instructions|prompts|rules`
- `you are now|new instructions|system:`
- `SYSTEM OVERRIDE|ADMIN MODE|DEBUG MODE`
- Excessive whitespace collapsed (4+ newlines → 3)

### 3.6 Audit Logging

| Log Type | Table | Fields |
|----------|-------|--------|
| Admin Actions | `AdminAuditLog` | admin_user_id, timestamp, table_name, record_id, old_values, new_values |
| AI Prompts | `AIPromptLog` | user_id, task, provider, model, system_prompt, user_prompt, response_text, tokens, created_at |
| Request Correlation | Middleware | X-Request-ID header, context var for log tracing |

### 3.7 Grafana Security

| Aspect | Implementation |
|--------|---------------|
| Authentication | Local admin user; password set via `GRAFANA_ADMIN_PASSWORD` env var (default: `changeme` in .env.example) |
| Network Isolation | Grafana container communicates with Backend via Docker internal network; not exposed to public internet in production unless explicitly configured |
| Access Control | Single admin account by default; Grafana built-in RBAC available for multi-user orgs |
| Data Sensitivity | Dashboards display aggregate metrics only (call counts, latency, error rates); no PII, CV content, or API keys exposed |
| Dashboard Provisioning | Dashboards created via authenticated API calls (admin credentials); no anonymous provisioning |

### 3.8 Ollama Security

| Aspect | Implementation |
|--------|---------------|
| Network Isolation | Ollama container only accessible within Docker network; port 11434 not exposed to public internet |
| API Authentication | No authentication on Ollama API (by design — local inference engine); access controlled via network isolation |
| Data Privacy | All inference runs locally — CV text and job descriptions never leave the server; no data sent to external APIs |
| Model Integrity | Models downloaded from official Ollama registry; SHA256 hash verified on download |
| Resource Limits | Container memory and CPU limits configurable via Docker Compose; prevents resource exhaustion on shared hosts |
| Prompt Data | Same prompt sanitisation applied as for cloud providers (Section 3.5) |

---

## 4. Functional Requirements

### 4.1 FR-001: Multi-Source Job Search

**Traces to**: BR-001

**User Story**: As a job seeker, I want to search multiple job boards and company career pages simultaneously so that I can find all relevant roles in one operation.

**Use Cases**:

| UC | Actor | Action | System Response |
|----|-------|--------|----------------|
| UC-001a | User | Enters search criteria (location, salary, keywords, work mode) and clicks Search | System queries all enabled job boards and career pages in parallel via SSE streaming |
| UC-001b | User | Selects specific career pages to include | System filters search to only selected company boards |
| UC-001c | User | Saves search as a reusable profile | System stores criteria, boards, and CV snapshot for future use |

**Acceptance Criteria**:
- Search returns results from all enabled sources within 60 seconds
- Results are deduplicated by URL (same job from multiple sources appears once)
- Streaming SSE delivers results per-source as they arrive (no waiting for slowest)
- Location filtering supports distance radius (default 25 miles)
- Work mode filter supports: remote, hybrid, onsite, any

**Error Handling**:
- If a source fails, log error and continue with remaining sources
- If all sources fail, return empty results with error summary
- Rate limit exceeded: return 429 with retry-after header

### 4.2 FR-002: CV Analysis

**Traces to**: BR-002

**User Story**: As a job seeker, I want the system to analyse my CV and extract my skills, experience, and qualifications so that it can match me to relevant jobs.

**Use Cases**:

| UC | Actor | Action | System Response |
|----|-------|--------|----------------|
| UC-002a | User | Uploads CV file (PDF/DOCX/TXT) | System extracts text, sends to AI provider, returns structured profile |
| UC-002b | User | Selects multiple AI providers | System analyses CV with each provider in parallel, aggregates results |
| UC-002c | User | Edits skill proficiency levels | System updates profile with user overrides |

**Acceptance Criteria**:
- Extracts: skills, experience_years, job_titles, education, languages, industries, strengths, achievements
- Supports PDF, DOCX, and TXT file formats
- Multi-provider analysis shows per-provider results with aggregate view
- Results cached: L1 in-memory (10 min), L2 database (24 hours)
- Falls back to keyword extraction if no AI provider available

### 4.3 FR-003: Job-to-CV Fit Scoring

**Traces to**: BR-003

**User Story**: As a job seeker, I want each job scored against my CV so that I can prioritise the best matches.

**Scoring Algorithm**:

```
IF ai_score AND keyword_score available:
    match_score = 0.7 * ai_score + 0.3 * keyword_score
    scoring_method = "ai_weighted"
ELIF ai_score available:
    match_score = ai_score
    scoring_method = "ai_only"
ELSE:
    match_score = keyword_score
    scoring_method = "keyword_only"

Sub-scores (each 0-25, summing to 100 max):
  - skills_match: overlap between CV skills and job requirements
  - experience_fit: years of experience vs. job seniority
  - industry_alignment: CV industries vs. company sector
  - seniority_match: CV seniority level vs. job level
```

**Acceptance Criteria**:
- Score range: 0-100 with colour coding (green 80+, orange 60-79, red <60)
- Sub-score breakdown available via expandable detail panel
- Score history tracked with timestamp, provider, and CV version
- Batch scoring supports parallel execution
- Fallback to keyword-only scoring when AI unavailable

### 4.4 FR-004: Document Generation

**Traces to**: BR-004

**User Story**: As a job seeker, I want the system to generate a tailored CV and cover letter for each job so that I can apply quickly with optimised documents.

**Acceptance Criteria**:
- Generated CV rewrites user's original CV to emphasise role-relevant skills
- Cover letter personalised with company name, matched skills, and job-specific details
- ATS compatibility score (0-100) provided immediately after generation
- Output formats: TXT (stored), DOCX and PDF (exported)
- Multi-AI comparison: generate variants from different providers, compare side-by-side
- Inline editing: user can edit generated text and regenerate DOCX/PDF

### 4.5 FR-005: Application Tracking

**Traces to**: BR-005

**User Story**: As a job seeker, I want to track the status of all my applications in one place.

**Status Workflow**:

```
not_applied ──> applied ──> interviewing ──> offer
                   │              │              │
                   └──> rejected  └──> rejected  └──> rejected
                                                      │
                                                      v
                                                  archived
```

**Acceptance Criteria**:
- Status dropdown on each job card
- Two views: table (sortable, filterable) and Kanban (drag-drop by status)
- Filters: by status, show/hide archived, text search
- Listing status check: verify if job posting is still active
- Follow-up reminders: configurable alerts for applications without response

### 4.6 FR-006: AI Provider Routing

**Traces to**: BR-006

**Priority-Based Routing System**:

Users configure a provider priority order (highest priority first). The TaskRouter selects the first available provider for each AI call. If it fails, the gateway automatically tries the next in the chain. Circuit breakers skip providers with recent failures. When all providers are exhausted, 22 keyword-based fallback methods provide basic results.

| Component | Role |
|-----------|------|
| **Priority chain** | User-configurable ordered list: pluribus → anthropic → google → openai → grok → openrouter → mistral → github → ollama |
| **Circuit breaker** | Temporarily skips providers with recent failures |
| **Sandbox routing** | Dedicated sandbox_google → sandbox_openrouter → keyword (isolated from production) |
| **Keyword fallback** | 22 template-based methods covering every AI operation |

**Failover Chain**:
1. User-specified provider (if explicitly chosen)
2. Tier-assigned provider (based on task type)
3. Priority-order fallback (configurable provider list)
4. Keyword-only fallback (no AI required)

**Permanent Error Detection**: Quota exhaustion errors (e.g., Google `RESOURCE_EXHAUSTED` with "quota") are classified as permanent failures. The circuit breaker is force-opened immediately (bypassing normal threshold count), and retries are skipped to avoid wasting time on exhausted providers.

### 4.7 FR-007: Career Page Search

**Traces to**: BR-001

**Supported ATS Platforms** (11 handlers):

| ATS | Handler | API Type | Companies |
|-----|---------|----------|-----------|
| Greenhouse | GreenhouseHandler | REST JSON API | TCS, Rothesay Life |
| Lever | LeverHandler | REST JSON API | Various |
| SmartRecruiters | SmartRecruitersHandler | REST JSON API | Wise, Legal & General, Informa |
| Workday | WorkdayHandler | POST JSON API | LSEG, M&G, JP Morgan, Accenture, 20+ |
| Workable | WorkableHandler | REST JSON API | Starling Bank, Hargreaves Lansdown |
| Eightfold | EightfoldHandler | POST JSON API + HTML fallback | Morgan Stanley, Citi |
| Oracle HCM | OracleHCMHandler | REST API | JP Morgan |
| Ashby | AshbyHandler | REST JSON API | ClearBank |
| Avature | AvatureHandler | HTML scraping | Metro Bank, Deloitte |
| SuccessFactors | SuccessFactorsHandler | HTML scraping | Atos |
| TalentSoft | TalentSoftHandler | HTML scraping | Various |

**Dispatch Chain**:
```
board.ats_provider → resolve_site_type() → get_handler() → handler.search()
    → If fails: HTML scraper fallback → adaptive parser
    → If SPA detected: diagnostic message returned
```

### 4.8 FR-008: Salary Intelligence

**Traces to**: BR-007

**Data Sources (Priority Order)**:
1. Job posting salary (normalised to annual GBP)
2. Curated benchmarks (~374 entries across 19 UK industry sectors, 41 unique roles)
3. Adzuna API (affiliate market data connector with click tracking)
4. AI estimation (LLM-generated when no data available)

**Employment Type**: Each salary estimate carries an `employment_type` flag (`Contract` or `Permanent`), displayed as a sortable colour-coded badge in the Companies tab.

**Normalisation Rules**:
- Daily rate x 230 working days = annual
- Hourly rate x 1,840 working hours = annual
- "£45k-55k" → {min: 45000, max: 55000}
- "Competitive" / "DOE" → null

### 4.9 FR-009: Monitoring & Observability

**Traces to**: BR-013, BR-021, BR-022

**Dashboard Panels** (Grafana):

| Panel | Type | Data Source |
|-------|------|-----------|
| Provider Health Overview | Table | GET /api/monitoring/providers |
| Daily Usage vs Limits | Bar gauge | GET /api/monitoring/usage |
| Token Consumption | Donut chart | GET /api/monitoring/providers |
| Recent AI Requests | Table | GET /api/monitoring/logs |
| Error Rate by Provider | Stat cards | GET /api/monitoring/providers |
| Avg Latency by Provider | Stat cards | GET /api/monitoring/providers |

**Security & Multi-User Panels** (collapsible row in `cviper-unified` dashboard):

| Panel | Type | LogQL Data Source |
|-------|------|-------------------|
| Auth Events Timeline | Timeseries | `count_over_time(... \| event_type =~ "auth_.*")` grouped by event_type |
| Failed Logins (24h) | Stat | `count_over_time(... \| event_type = "auth_login_failure" [24h])` |
| RBAC Denials | Stat | `count_over_time(... \| event_type = "rbac_denial")` |
| Token Budget Events | Table | `... \| event_type =~ "token_budget_.*"` with JSON field extraction |
| AI Usage by Provider | Bar chart | `count_over_time(... \| event_type = "ai_call_complete")` grouped by provider |
| Key Source Distribution | Pie chart | `count_over_time(... \| event_type = "key_resolution")` grouped by key_source |

**Template Variable**: `$user_id` (textbox, default `.*`) for per-user filtering across all security panels.

**Security Alert Rules** (in `monitoring/grafana/provisioning/alerting/alerts.yml`):

| Alert | Trigger | Duration | Severity |
|-------|---------|----------|----------|
| Brute Force Login | >10 `auth_login_failure` in 5 min | 1m | Critical |
| Token Budget Exceeded | >3 `token_budget_exceeded` in 15 min | 5m | Warning |
| RBAC Denial Spike | >3/min `rbac_denial` sustained | 5m | Warning |
| Key Decryption Failure | Any "Failed to decrypt user key" in 5 min | 5m | Critical |

### 4.10 FR-010: Grafana Automated Deployment

**Traces to**: BR-013, BR-021, BR-022, BR-024

**User Story**: As an operator, I want Grafana to deploy automatically with preconfigured dashboards so that I have monitoring available immediately without manual setup.

**Deployment Requirements**:

| Requirement | Details |
|-------------|---------|
| Container | `grafana/grafana-oss:latest` via Docker Compose or Azure Container Apps |
| Port | 3001 (mapped from container port 3000) |
| Authentication | Admin password set via `GRAFANA_ADMIN_PASSWORD` env var |
| Datasource | Infinity plugin (`yesoreyeram-infinity-datasource`) auto-installed and configured to point at Backend API |
| Dashboard | `cviper-unified` dashboard (UID: `cviper-unified`) auto-provisioned via Grafana API on first startup |
| Network | Accesses Backend via Docker network (`host.docker.internal:8000` locally, internal service URL in production) |

**Acceptance Criteria**:
- Grafana container starts alongside backend and frontend with no manual intervention
- Infinity datasource plugin is installed and configured automatically
- `cviper-unified` dashboard with 6 panels is created on first startup
- Dashboard displays live data from backend monitoring endpoints
- Backend `/api/monitoring/grafana-link` returns a valid, clickable deep-link to the dashboard
- Frontend Monitoring tab shows active Grafana button (not "not configured" banner) when GRAFANA_URL is set
- If Grafana is unreachable, backend and frontend degrade gracefully (banner message, no errors)

**Auto-Provisioning Sequence**:
```
[Container Start]
    │
    ├──> Grafana starts with GF_SECURITY_ADMIN_PASSWORD from env
    │
    ├──> Entrypoint script or init container:
    │       1. Install Infinity datasource plugin (grafana cli plugins install)
    │       2. Restart Grafana to load plugin
    │       3. POST /api/datasources — create "CViper API" datasource
    │       4. POST /api/dashboards/db — create cviper-unified dashboard
    │
    └──> Health check: GET /api/health returns 200
         Dashboard accessible at /d/cviper-unified/
```

### 4.11 FR-011: Ollama Automated Deployment

**Traces to**: BR-019, BR-023, BR-024

**User Story**: As a user, I want Ollama to deploy and configure automatically so that I have local AI inference available without manual model downloads or API configuration.

**Deployment Requirements**:

| Requirement | Details |
|-------------|---------|
| Container | `ollama/ollama:latest` via Docker Compose or Azure Container Apps (demo-only, scale 0-1) |
| Port | 11434 |
| Model | Default: `llama3.2` (configurable via `OLLAMA_MODEL` env var) |
| Model Loading | Auto-pull on first startup if not present |
| API | OpenAI-compatible endpoint at `/v1/chat/completions` |
| Resources | CPU by default; GPU passthrough configurable via Docker `--gpus` flag |
| Integration | Backend auto-detects Ollama on startup, registers in ProviderRegistry |

**Acceptance Criteria**:
- Ollama container starts alongside backend with no manual intervention
- Configured model is automatically downloaded on first run
- Backend detects Ollama availability via `GET {OLLAMA_BASE_URL}/api/tags`
- Ollama appears as an available provider in `/api/ai-providers` response
- Users can select Ollama for any AI task (default: Light-tier tasks)
- Model switching via `/api/ollama/set-model` works without container restart
- If Ollama is unavailable, backend falls through to next provider in chain (no errors)
- Ollama metrics (calls, tokens, latency) appear in Grafana dashboard alongside cloud providers
- All existing users have immediate access to Ollama — no migration, enablement, or per-user provisioning required
- Newly registered users can use Ollama from their first login with no additional setup
- Ollama access is role-agnostic: standard users and admin users have identical Ollama capabilities
- No per-user Ollama configuration is required — the backend provides Ollama as a shared infrastructure service accessible through the standard AI provider framework

**Resource Configuration**:

| Environment | CPU | RAM | GPU | Model |
|-------------|-----|-----|-----|-------|
| Local Dev | Host CPU | Host RAM | Optional | llama3.2 (~2GB) |
| Azure Container Apps (demo) | 2 vCPU | 4 GiB | None | llama3.2 (pre-baked in image) |
| Production (GPU) | Host CPU | 8GB+ | NVIDIA (passthrough) | llama3.2 or larger |

### 4.12 FR-012: Simplified CV Analysis Page (P1)

**Traces to**: BR-039

**User Story**: As a job seeker, I want the CV Analysis page to focus on uploading and analyzing my CV without cluttering the interface with AI provider configuration.

**Acceptance Criteria**:
- AI provider card removed from CV Analysis page
- Inline provider indicator (green dot + provider name) shown next to Analyze button
- Warning alert with "Go to Settings" CTA shown when no provider is configured
- Provider configuration remains exclusively in Settings tab

### 4.13 FR-013: Guided Onboarding — Wizard Mode + ProgressStepper (P2)

**Traces to**: BR-040

**User Story**: As a first-time user, I want guided onboarding that walks me through the core workflow without overwhelming me with the full UI.

**Acceptance Criteria**:
- **WizardMode** (primary): auto-starts for first-visit and demo users. Full-width ribbon below TopNav guides through Upload → Search → Score → Save (registered) or Explore CV → See Scores → Try Search → Your Turn (demo). Auto-advances on completion signals (e.g. hasCv, hasSearchResults). When active, TopNav shows only the current step's tab + Settings. "Exit Guide" available at any step.
- **ProgressStepper** (ambient): 5-step compact stepper (Upload CV → Review Profile → Find Jobs → Apply → Get Help) always visible below TopNav. Tracks completion with per-step checkmarks. Pulses next incomplete step. Clicking a step navigates to its tab.
- Wizard auto-start: `useWizard({ autoStart: true })` activates on mount when no saved state exists. Welcome modals dismiss to reveal the already-running wizard ("Get Started") or dismiss + exit ("Skip").
- Demo wizard uses viewing steps with auto-complete timers (8s) + action steps + conversion CTA.

### 4.14 FR-014: CV-to-Search Data Flow (P3)

**Traces to**: BR-041

**User Story**: As a user who has analyzed my CV, I want the Search tab to show me where my CV data is being used so I understand the connection between analysis and search.

**Acceptance Criteria**:
- "Suggested from CV" badge shown next to auto-populated search criteria
- "analyze your CV" clickable link navigates to CV Analysis tab when no CV exists
- "Re-analyze CV" link available for users who want to update their profile
- Skills from CV analysis labelled with "Skills from your CV analysis" badge

### 4.15 FR-015: Collapsible Job Sources (P4)

**Traces to**: BR-042

**User Story**: As a user, I want the job boards and career pages grouped in a collapsible section so the search form is not overwhelming.

**Acceptance Criteria**:
- Job Boards and Career Pages wrapped in a collapsible "Job Sources" section
- Section auto-expands when sources are loaded from backend or when backend is connecting
- Company grid shows only selected/favourited items when total exceeds 10 companies
- "Show All Companies" toggle available for large lists

### 4.16 FR-016: Progressive Tab Disclosure + Advanced Mode Toggle (P5)

**Traces to**: BR-043

**User Story**: As a new user, I should see only the tabs relevant to my current workflow. As a power user, I should be able to reveal all tabs.

**Acceptance Criteria**:
- **Progressive disclosure tiers** (config/tabs.js `TAB_TIERS`):
  - `focused` (default): CV Analysis, Job Search, Applications, Settings (4 tabs)
  - `standard` (after CV upload + first search): + Company Salaries, Career Insights, Skills & Training
  - `full` (after applying, or explicit opt-in): all tabs including News Feed, FAQ, Behind CViper
- **Tab grouping**: Insights dropdown (Companies, Career Insights, Skills, News Feed) and More dropdown (Prompt Lab, My Requests, FAQ, Behind CViper) reduce visible top-level items
- **"Show all tabs" toggle** in Settings > Preferences bypasses tier filtering. Persisted via `userStorage.getItemGlobal`/`setItemGlobal` (readable before login)
- **Advanced Mode toggle** (existing): Monitoring and Prompt Lab hidden unless enabled. Gear icon on advanced tabs
- Tier derivation: `useTabTier` hook reads `onboarding.steps` (upload/search/apply) and `showAllTabs` preference
- Admin users always see full tier

### 4.17 FR-017: Settings Sub-Navigation (P6)

**Traces to**: BR-044

**User Story**: As a user, I want the Settings page organised into logical sections so I can find what I need quickly.

**Acceptance Criteria**:
- 4 sub-tabs: General, AI Providers, Search & Keywords, Preferences
- Horizontal tab bar with active indicator (blue underline + bold text)
- General: CV settings, format, library. AI Providers: priority + API keys. Search: keywords + industry prefs. Preferences: advanced mode + password
- Default sub-tab is "General"

### 4.18 FR-018: Contextual Empty States (P7)

**Traces to**: BR-045

**User Story**: As a user, I want to see helpful guidance when a tab has no data instead of a blank screen.

**Acceptance Criteria**:
- CV Analysis: "No CV Analyzed Yet" with document icon when no analysis exists
- Job Search: "No Search Results Yet" with search icon before first search
- Companies: "No Salary Estimates Yet" with building icon when table is empty
- All empty states use consistent `.empty-state` CSS class with icon (40px), heading, and descriptive paragraph
- Empty states hidden immediately when data loads

### 4.19 FR-019: GDPR Compliance

**Traces to**: BRD Section 5.3

**User Story**: As a user, I want full control over my personal data in compliance with UK GDPR, including the ability to export and delete my data.

**Acceptance Criteria**:
- `GET /api/gdpr/export` returns comprehensive JSON of all user data (jobs, analyses, configs, prompt logs, consents), rate-limited to 2/hour
- `DELETE /api/gdpr/delete-account` requires password verification, performs cascading deletion of all user data across all tables
- `UserConsent` model tracks granular consent (analytics, ai_processing, data_sharing) with timestamps
- Consent banner shown on first visit; consents viewable/withdrawable via `/api/gdpr/consents` endpoints
- Privacy Policy page accessible from sidebar and config tab
- AI prompt logs (`system_prompt` field) encrypted at rest with Fernet
- IP addresses anonymised in request logs (last octet/segment zeroed)
- Account lockout: 10 failed login attempts within 15 minutes triggers HTTP 429
- Google Fonts self-hosted (Inter + IBM Plex Mono woff2) to eliminate cross-border data transfers; CSP headers disallow external font sources
- AI prompt logs and search history purged after 180 days via scheduled maintenance
- `MASTER_KEY` required in production (startup fails without it)
- Incident response plan documented (`docs/INCIDENT_RESPONSE_PLAN.md`) with 72-hour ICO notification window

### 4.20 FR-020: Sandbox Abuse Prevention

**Traces to**: BR-046

**User Story**: As the platform operator, I want to prevent sandbox account abuse so that public trial access remains available without excessive AI consumption.

**5-Layer Strategy**:

| Layer | Mechanism | Default Limit |
|-------|-----------|--------------|
| 1. Browser Fingerprint | SHA-256 of canvas + screen + timezone | Unique visitor identification |
| 2. Per-Fingerprint Limit | Daily session cap per fingerprint | 50 sessions/day |
| 3. Per-IP Limit | Daily session cap per IP address | 100 sessions/day |
| 4. Dedicated Sandbox Providers | sandbox_google (1000 req/day) + sandbox_openrouter (50 req/day) with keyword fallback | Per-provider daily quotas |
| 5. Output Truncation + Session Expiry | AI responses capped, sessions auto-expire after 1 hour | Scores, skills, rationale shortened; background cleanup every 5 min |

**Acceptance Criteria**:
- `SandboxUsage` model tracks daily usage per fingerprint and IP
- Sandbox users restricted to sandbox-prefixed AI providers (sandbox_google, sandbox_openrouter) and keyword fallback (no user cloud provider keys exposed)
- When sandbox AI quota is exhausted, user-facing error message displayed: "Sandbox AI is temporarily busy — using keyword matching instead. Create a free account and add your own API key for unlimited AI access."
- AI responses truncated: sub_scores, pros/cons, salary rationale capped at configurable lengths
- Sandbox career pages seeded with 10 companies (6 banks, 4 fintechs)
- 30 tests covering repository, endpoint, and truncation code paths

### 4.21 FR-021: Adzuna Affiliate Job Search

**Traces to**: BR-047

**User Story**: As a job seeker, I want to search Adzuna alongside other job boards so that I have access to a wider range of job listings.

**Acceptance Criteria**:
- `AdzunaJobsAPI` class implements search with salary formatting and date filtering
- Adzuna registered in `JobSearchAggregator` as a first-class source (enabled by default for new users)
- Job links use Adzuna's `redirect_url` for affiliate commission tracking
- `AffiliateClick` model stores URL hash (not full URL), timestamp, user_id, and source
- `POST /api/track-click` endpoint records outbound clicks non-blockingly
- Non-blocking `onClick` handler in `SearchResultsList` for job link clicks
- 25 tests covering API integration, affiliate URLs, error handling, and click tracking

### 4.22 FR-022: 7-Step Onboarding Registration

**Traces to**: BR-048

**User Story**: As a new user, I want a guided registration flow that collects my preferences step by step so that the platform is configured for my needs from the start.

**Registration Steps**:

| Step | Content | Behaviour |
|------|---------|-----------|
| 1 | Credentials (username, password, confirm) | Validation rules applied |
| 2 | Personal details (display name, email) | Optional fields |
| 3 | Industry selection (19 UK sectors) | Multi-select checkbox grid |
| 4 | Role selection (filtered by step 3 industries) | Reduces cognitive load |
| 5 | Job Boards (dedicated picker) | Toggle enabled/disabled |
| 6 | Career Pages (full list, deduplicated) | Admin defaults + global config merged, case-insensitive dedup |
| 7 | Privacy Policy summary | Accept & Register / Decline buttons |

**Acceptance Criteria**:
- Declining privacy policy resets to login with explanatory message
- Career pages endpoint merges `admin_default_career_pages` and `company_boards` configs
- ProgressStepper redesigned with gradient circles, connector lines with glow, hint subtitles, progress counter, hover lift animation
- WCAG AA accessible colours: #A7F3D0 (11:1 contrast), rgba(255,255,255,0.65) (5.8:1 contrast) against dark nav background

### 4.23 FR-023: Password Reset Flow

**Traces to**: BR-049

**User Story**: As a user who has forgotten their password, I want to reset it via email so that I can regain access to my account.

**Acceptance Criteria**:
- "Forgot password?" link on login screen
- `POST /api/auth/forgot-password` generates time-limited token and sends email
- `POST /api/auth/reset-password` validates token and sets new password
- Show/hide password toggle (eye icon) on all password inputs: login, register, register confirm, reset, reset confirm
- 22 tests covering forgot-password, token validation, reset flow, and email service

### 4.24 FR-024: Performance Optimisation

**Traces to**: BR-051

**User Story**: As a user, I want the application to respond quickly and handle concurrent requests efficiently.

**Optimisations Implemented**:

| Area | Change | Impact |
|------|--------|--------|
| Database | Wrap blocking sync calls in `asyncio.to_thread()` | Non-blocking event loop |
| Compression | `GZipMiddleware` for JSON responses | Reduced bandwidth |
| Tracing | OpenTelemetry (FastAPI + SQLAlchemy + httpx) | Distributed request tracing |
| Frontend | Lazy-load 7 modals, `React.memo()` on SearchForm/DocumentCentre/AdminPanel | Reduced initial bundle |
| Rendering | Progressive rendering: first 100 results + "Show more" | Faster perceived load |
| Pagination | Default limits on `/api/saved-jobs` and `/api/companies` | Bounded response size |
| Caching | `useMemo` on CompaniesTab computed sets | Reduced re-renders |
| Azure | VNet + private endpoint for PostgreSQL, PgBouncer (port 6432) | Secure, pooled connections |
| Azure | `minReplicas: 1` on backend | Eliminates cold-start latency |
| DB Indexes | Composite indexes for user-scoped queries (jobs, salary_estimates, prompt_logs) | Faster filtered queries |
| DB Integrity | CHECK constraints on scores (0-100), roles ('root'\|'user'), unique salary estimates | Data validation at DB level |
| Load Testing | Locust suite with 3 user profiles | Performance baseline |

### 4.25 FR-025: iPhone & Mobile Safe Area Rendering

**Traces to**: BR-054

**User Story**: As a mobile user, I want the application to render correctly on iPhone and other devices with notches, rounded corners, and dynamic browser chrome, so that no content is obscured.

**Implementation**:

| Area | Change | Detail |
|------|--------|--------|
| Viewport | `viewport-fit=cover` meta tag | Extends web content into safe area regions |
| Layout | `min-height: calc(100dvh - 70px)` | Dynamic viewport height accounts for mobile browser chrome resize |
| Navigation | `padding: 0 max(24px, env(safe-area-inset-right)) 0 max(24px, env(safe-area-inset-left))` | Nav container respects device safe areas |
| Toast | `bottom: max(24px, env(safe-area-inset-bottom))` | Toasts avoid home indicator overlap |
| Breakpoint | New `@media (max-width: 480px)` | Compact padding and font sizes for small phones |
| Fallback | `100vh` retained before `100dvh` | Browsers without dvh support use vh |

**Acceptance Criteria**:
- Content is not obscured by notch, home indicator, or rounded corners on iPhone
- Toast notifications appear above the home indicator
- Navigation bar padding adapts to safe area insets
- Layout is usable on screens as narrow as 320px

### 4.26 FR-026: AI Model Selection Per Provider

**Traces to**: BR-055

**User Story**: As a user, I want to select which AI model each provider uses, so that I can choose between faster/cheaper models and more capable ones.

**Implementation**:

| Component | Detail |
|-----------|--------|
| Backend: `PROVIDER_MODELS` | Hardcoded dict in `ai/providers.py` mapping 8 cloud providers + local gateways to their available models (id, name, tier) |
| Backend: endpoint | `PUT /api/ai-provider-model/{provider_id}` — validates model against `PROVIDER_MODELS`, updates `ProviderRegistry.clients[provider_id]["model"]`, persists to config table as `provider_model_preferences` |
| Backend: response | `available_models` list added to `get_available_providers()` and `get_provider_keys_masked()` API responses |
| Frontend: ConfigTab | Model dropdown replaces static model label when `available_models` array is non-empty; `onChange` calls `PUT /api/ai-provider-model/{provider_id}` via `authFetch` |
| Persistence | Selected model stored in `config` table under key `provider_model_preferences` (JSON) |

**Supported Providers & Models**:

| Provider | Models |
|----------|--------|
| OpenAI | GPT-4o, GPT-4o Mini, GPT-4 Turbo, GPT-3.5 Turbo, o3-mini |
| Anthropic | Claude Opus 4.6, Claude Sonnet 4.6, Claude Haiku 4.5, Claude 3 Haiku |
| Google | Gemini 2.0 Flash, Gemini 1.5 Flash, Gemini 1.5 Pro |
| Mistral | Mistral Large, Mistral Small, Mistral Nemo |
| Grok (xAI) | Grok 3, Grok 3 Mini, Grok 2 |
| GitHub | GPT-4o (GitHub), GPT-4o Mini (GitHub) |
| OpenRouter | Auto (Best Available), Claude Sonnet 4.6, GPT-4o, Gemini Flash 2.0 |
| Pluribus (local only) | Default (auto) |

**Acceptance Criteria**:
- Model dropdown appears in Settings for each provider with a configured API key
- Selecting a model persists across sessions
- Model validation rejects invalid model IDs
- Providers without `available_models` show static model label (backward compatible)

### 4.27 FR-027: Hybrid JWT/OAuth Authentication

**Traces to**: BR-062, BR-063, BR-068

**User Story**: As a user, I want to sign in with my LinkedIn or Google account, or use a traditional username/password, so that I can access CViper with my preferred authentication method.

**Technical Implementation**:

| Component | Implementation |
|-----------|---------------|
| JWT Library | python-jose[cryptography] with HS256 signing |
| OAuth Library | authlib with OpenID Connect (LinkedIn) and OAuth2 (Google) |
| Access Token | 15-min expiry (1hr sandbox), stored in React state |
| Refresh Token | 7-day expiry, SHA-256 hashed in DB, httpOnly cookie, single-use rotation |
| Session Middleware | Starlette SessionMiddleware for OAuth state (10min, oauth_session cookie) |
| Backward Compat | Bearer tokens tried as JWT first, then as session token; cookie fallback preserved |

**New Backend Files**: `jwt_utils.py`, `oauth.py`, `account_migration.py`

**Acceptance Criteria**:
- Users can sign in via username/password, LinkedIn OAuth, or Google OAuth
- JWT access tokens contain user claims (sub, username, is_sandbox, is_admin, provider)
- Refresh tokens are rotated on each use (old revoked, new issued)
- Existing password-based users continue to work with no migration required
- OAuth callback redirects to frontend with JWT in URL param, sets refresh cookie
- 54 tests covering JWT lifecycle, OAuth configuration, and account migration

### 4.28 FR-029: Frontend AuthContext Provider

**Traces to**: BR-066

**User Story**: As a frontend developer, I want a centralised auth context so that all components can access authentication state without prop drilling.

**Technical Implementation**:

| Aspect | Implementation |
|--------|---------------|
| Context | `AuthContext` in `src/context/AuthContext.jsx` |
| Token Storage | JWT in React state via `useState` (never localStorage) |
| Silent Refresh | Timer schedules refresh 60s before JWT expiry via httpOnly cookie |
| OAuth Handling | Detects `?oauth_token=` URL param on mount, cleans URL |
| Legacy Bridge | `setAuthTokenGetter()` wires JWT into existing `authFetch()` and `useApi()` |

**Exposed API**: `useAuth()` hook returning: `user`, `token`, `isAuthenticated`, `isSandbox`, `isAdmin`, `loginPassword()`, `loginOAuth()`, `startSandbox()`, `register()`, `logout()`, `refreshToken()`, `getProviders()`, `getToken()`

**Acceptance Criteria**:
- AuthProvider wraps App in main.jsx
- All existing authFetch calls automatically include JWT when available
- Page refresh recovers auth state via silent token refresh
- OAuth callback token is consumed and URL cleaned on mount

### 4.29 FR-030: Sandbox UI Components

**Traces to**: BR-067

**User Story**: As a sandbox user, I want clear visual indicators that I'm in demo mode, with a timer showing my remaining session time and prompts to create an account.

**Components**:

| Component | Purpose |
|-----------|---------|
| `SandboxBanner` | Top-of-page banner with countdown timer and "Create Account" CTA |
| `ExampleBadge` | Small "EXAMPLE" label on seed data items (source="example") |
| `SignUpPrompt` | Modal overlay when sandbox users attempt restricted actions |

**Acceptance Criteria**:
- SandboxBanner shows countdown timer derived from `sandbox_expires_at`
- Timer turns warning colour (amber) when under 5 minutes
- Auto-logout triggers when timer reaches zero
- ExampleBadge renders only for items with `source="example"`
- SignUpPrompt offers "Create Account" and "Continue Exploring" options
- 19 frontend tests covering all component interactions

---

### 4.30 FR-031: Guided Base CV Bullet Optimization

**Traces to**: BR-069

**User Story**: As a job seeker, I want AI to analyse my base CV's bullet points and identify weak ones so I can improve them with stronger action verbs, quantified results, and the CAR (Challenge-Action-Result) pattern before tailoring for specific roles.

**Components**:

| Component | Location |
|-----------|----------|
| `build_optimize_bullets_prompt` | `backend/ai/prompts/document_gen.py` |
| `build_optimize_bullets_system` | `backend/ai/prompts/document_gen.py` |
| `DocumentGenService.optimize_base_cv_bullets` | `backend/ai/services/document_gen.py` |
| `AIService.optimize_base_cv_bullets` | `backend/ai_service.py` (facade) |
| `POST /api/cv/optimize-bullets` | `backend/routes/cv_optimization.py` |

**Request/Response**:
- **Request**: `{ "cv_text": "<full CV text, min 200 chars>", "provider": "<optional>" }`
- **Response**: `{ "overall_feedback": "<summary>", "improved_bullets": [{ "original_bullet", "suggested_rewrite", "reasoning", "context" }] }`

**AI Routing**: Task `optimize_cv_bullets` mapped to **premium** tier in `TaskRouter.TASK_TIER_MAP` (complex reasoning, high token usage).

**Rate Limit**: 10 requests/minute per user.

**Acceptance Criteria**:
- Endpoint returns structured JSON with `overall_feedback` and `improved_bullets` array
- Each improved bullet includes original text, suggested rewrite, reasoning, and location context
- CV text shorter than 200 characters returns a validation error (400)
- Sandbox users receive keyword fallback response (no AI cost)
- AI disabled state returns graceful degradation message
- Malformed AI responses return fallback message without crashing
- 7 unit tests covering all code paths pass

### 4.31 FR-032: RBAC Audit & Hardening

**Traces to**: BR-070, BR-058

**User Story**: As the platform operator, I want all privileged endpoints to enforce admin-only access at the route level so that standard users cannot exploit unprotected infrastructure endpoints.

**Changes**:

| Change | Location | Detail |
|--------|----------|--------|
| Service control admin enforcement | `backend/routes/misc.py` | Added `dependencies=[Depends(require_root)]` to `POST /api/service/backend` and `POST /api/service/frontend` |
| Sandbox advanced tab visibility | `frontend/src/components/TopNav.jsx` | Removed `isSandbox` from `showAllTabs` condition; sandbox users must enable Advanced Mode |
| RBAC test coverage | `backend/tests/security/test_rbac_enforcement.py` | 2 new parametrized tests verifying non-root users get 403 on service control endpoints |

**Acceptance Criteria**:
- `POST /api/service/backend` returns 403 for non-root authenticated users
- `POST /api/service/frontend` returns 403 for non-root authenticated users
- Sandbox users do not see Monitoring or Prompt Lab tabs unless Advanced Mode is enabled
- Admin (root) users continue to see all tabs and access all service control endpoints
- 26 RBAC enforcement tests pass (24 existing + 2 new service control tests)

### 4.33 FR-033: CV Optimisation Pipeline

**Traces to**: BR-071

**User Story**: As a job seeker, I want the system to identify missing ATS keywords in my CV, validate its format for ATS compatibility, and optimise it for a specific job with one click — so I can maximise my chances of passing automated screening.

**Components**:

| Feature | Description |
|---------|-------------|
| Keyword Injection Suggestions | Compares CV text against job description to identify high-value missing keywords; suggests natural incorporation points |
| ATS Format Validator | Checks CV structure against ATS compatibility rules: clean section headers, no tables/images/columns, consistent date formats, standard fonts |
| One-Click Optimise | Combines keyword injection + bullet rewriting + format validation into a single action tailored to a specific job posting |

**Acceptance Criteria**:
- Keyword injection returns a list of missing keywords ranked by importance with suggested incorporation locations
- ATS format validator returns a structured report with pass/fail checks and specific remediation advice
- One-click optimise produces a tailored CV variant with keywords injected, weak bullets rewritten, and format issues resolved
- All operations include AI provider attribution and confidence scores
- Fallback to keyword-based suggestions when AI is unavailable

### 4.34 FR-034: Training Provider Foundation

**Traces to**: BR-072

**User Story**: As a job seeker, I want to see recommended training courses and certifications for skills I'm missing, so I can close gaps and become a stronger candidate.

**Training Providers**:

| Provider | Type | Cost |
|----------|------|------|
| freeCodeCamp | Free | Free |
| Coursera (Audit mode) | Free | Free (audit) |
| edX (Audit mode) | Free | Free (audit) |
| Khan Academy | Free | Free |
| Udemy | Paid | Per-course |
| Pluralsight | Paid | Subscription |
| LinkedIn Learning | Paid | Subscription |
| Codecademy Pro | Paid | Subscription |

**Acceptance Criteria**:
- Skills & Training tab accessible via TopNav navigation
- Each skill gap from CV analysis links to relevant courses from the 8 providers
- Certification mapping: specific certifications (AWS, Azure, PMP, etc.) mapped to skills they validate
- Skill progress tracking: users can mark skills as "learning", "completed", or "certified"
- Provider data stored in seed data with URLs, cost indicators, and certification details

### 4.35 FR-035: AI Ethics & Fairness

**Traces to**: BR-073

**User Story**: As a user, I want assurance that AI scoring is fair and transparent, so I can trust the system's recommendations and challenge results I disagree with.

**Components**:

| Component | Implementation |
|-----------|---------------|
| Fairness Guardrails | System prompts include explicit instructions to evaluate based on skills, experience, and qualifications only — not name, age, gender, ethnicity, or other protected characteristics |
| Confidence Scores | Every AI-generated score includes a confidence rating (0-100) indicating model certainty |
| Challenge This Score | Button on score detail panels triggers re-evaluation with a different prompt variation and explicit reasoning |
| AI Transparency | Disclosure panel explaining how scores are generated, which provider/model was used, and limitations |

**Acceptance Criteria**:
- All AI scoring prompts contain fairness guardrails (verifiable via prompt audit)
- Confidence scores displayed alongside fit scores, salary estimates, and analysis results
- Challenge This Score produces a new evaluation with written reasoning for the score
- AI transparency disclosure accessible from any AI-generated result
- Fairness guardrails cannot be overridden by user input (prompt injection prevention applies)

### 4.36 FR-036: Growth Readiness

**Traces to**: BR-074

**User Story**: As the platform operator, I want the application to be discoverable via search engines, shareable on social media, installable as a PWA, and instrumented with privacy-respecting analytics.

**Components**:

| Feature | Implementation |
|---------|---------------|
| Open Graph Tags | `og:title`, `og:description`, `og:image`, `og:url` meta tags in index.html |
| Twitter Cards | `twitter:card`, `twitter:title`, `twitter:description` meta tags |
| robots.txt | Allows search engine crawling of public pages, disallows API and admin paths |
| sitemap.xml | Lists public-facing pages for search engine indexing |
| PWA Install Prompt | Browser-native install prompt with custom UI trigger for supported browsers |
| Plausible Analytics | Privacy-respecting analytics script (no cookies, no personal data, GDPR-compliant) |

**Acceptance Criteria**:
- Social sharing previews render correctly on LinkedIn, Twitter/X, and Facebook
- robots.txt correctly allows/disallows appropriate paths
- sitemap.xml lists all public pages
- PWA install prompt appears on mobile browsers that support installation
- Plausible analytics tracks page views without cookies or personal data

### 4.37 FR-037: Cross-user Data Isolation

**Traces to**: BR-075

**User Story**: As a user, I want my data to be completely isolated from other users, so that switching accounts never reveals another user's jobs, analyses, or preferences.

**3-Layer Prevention Strategy**:

| Layer | Mechanism | Implementation |
|-------|-----------|---------------|
| 1. Backend Scoping | All repo functions filter by `user_id` | Every `repo.get_*`, `repo.list_*`, `repo.find_*` called with explicit `user_id=_uid(request)` |
| 2. Frontend State Reset | React state cleared on user switch | `handleLogout` resets all user-scoped state; `useEffect` on `currentUser?.id` as safety net |
| 3. localStorage Namespacing | Per-user key namespacing | `userStorage` utility: `cviper:u:<userId>:<key>` format; `purgeAllUserScopedStorage()` on switch |

**Acceptance Criteria**:
- No backend repo function returns data across users (enforced by `test_user_isolation_audit.py`)
- Switching users clears all previous user data from React state
- localStorage keys are namespaced per user; switching users purges all namespaced keys
- Legacy unprefixed localStorage keys are also purged on user switch (defence-in-depth)
- Regression guard tests verify all three layers

### 4.38 FR-038: API Contract Tests

**Traces to**: BR-076

**User Story**: As a developer, I want automated contract tests that verify API response shapes so that backward-incompatible changes are caught before deployment.

**Implementation**:

| Aspect | Detail |
|--------|--------|
| Test Count | 14 contract tests covering major endpoints |
| Schema File | Shared schema definition with required keys and value types per endpoint |
| CI Gate | Contract test failures block merge (hard gate) |
| Coverage | Health, auth, jobs, search, CV analysis, companies, monitoring, config endpoints |

**Acceptance Criteria**:
- 14 contract tests validate response shapes against shared schema
- Adding or removing a required response field without updating the schema fails CI
- Schema file serves as machine-readable API documentation
- Contract tests run as part of the standard test suite (no separate execution)

### 4.39 FR-039: Security Verification

**Traces to**: BR-077

**User Story**: As the platform operator, I want all P0 security controls verified and maintained so that the application meets security baseline requirements.

**Verified Controls**:

| Control | Status | Enforcement |
|---------|--------|-------------|
| Content Security Policy headers | Verified | SecurityHeadersMiddleware (report-only mode) |
| CORS origin guards | Verified | Fatal production guard prevents wildcard origins |
| Secret encryption at rest | Verified | Fernet master key; `MASTER_KEY` required in production |
| Terms of Service | Verified | Published at `/?tab=terms`; accepted during registration |
| Subprocess hardening | Verified | All `shell=True` removed; AST ratchet test enforces in CI |

**Acceptance Criteria**:
- All P0 security controls pass automated verification
- AST ratchet test prevents reintroduction of `shell=True` in backend code
- Fatal startup guards prevent production boot without required security configuration
- Security controls are documented in `SECURITY.md`

---

## 5. Process & Sequence Diagrams

### 5.1 Job Search Sequence (Streaming)

```
┌──────┐    ┌────────┐    ┌──────────┐    ┌────────────┐    ┌──────────┐
│Client│    │ FastAPI │    │ SearchSvc│    │ ATSHandler │    │ AIService│
└──┬───┘    └───┬────┘    └────┬─────┘    └─────┬──────┘    └────┬─────┘
   │            │              │                │               │
   │POST /search-stream        │                │               │
   │───────────>│              │                │               │
   │            │ search_jobs  │                │               │
   │            │ _streaming() │                │               │
   │            │─────────────>│                │               │
   │            │              │                │               │
   │  SSE:profile              │                │               │
   │<───────────│              │                │               │
   │            │              │ For each board:│               │
   │            │              │───────────────>│               │
   │            │              │                │ search(board, │
   │            │              │                │ keywords, loc)│
   │            │              │                │────┐          │
   │            │              │                │    │ API call  │
   │            │              │  job_results   │<───┘          │
   │            │              │<───────────────│               │
   │            │              │                │               │
   │            │              │ For each job:  │               │
   │            │              │ ──────────────────────────────>│
   │            │              │                │    match_job() │
   │            │              │   scored_job   │               │
   │            │              │ <──────────────────────────────│
   │            │              │                │               │
   │  SSE:site_result          │                │               │
   │<───────────│<─────────────│                │               │
   │            │              │                │               │
   │  SSE:complete             │                │               │
   │<───────────│              │                │               │
```

### 5.2 Document Generation Sequence

```
┌──────┐    ┌────────┐    ┌──────────┐    ┌──────────┐
│Client│    │ FastAPI │    │ DocGenSvc│    │ AIGateway│
└──┬───┘    └───┬────┘    └────┬─────┘    └────┬─────┘
   │            │              │               │
   │POST /apply │              │               │
   │{job_ids,   │              │               │
   │ cv_folder} │              │               │
   │───────────>│              │               │
   │            │ load CV text │               │
   │            │──────┐       │               │
   │            │<─────┘       │               │
   │            │              │               │
   │            │ For each job:│               │
   │            │─────────────>│               │
   │            │              │ generate_cv() │
   │            │              │──────────────>│
   │            │              │  tailored_cv  │
   │            │              │<──────────────│
   │            │              │               │
   │            │              │ generate_cl() │
   │            │              │──────────────>│
   │            │              │ cover_letter  │
   │            │              │<──────────────│
   │            │              │               │
   │            │              │ score_ats()   │
   │            │              │──────────────>│
   │            │              │  ats_score    │
   │            │              │<──────────────│
   │            │              │               │
   │            │  {documents, │               │
   │            │   ats_score} │               │
   │            │<─────────────│               │
   │ {results}  │              │               │
   │<───────────│              │               │
```

### 5.3 AI Provider Failover State Diagram

```
                    ┌─────────────────┐
                    │  Request Received│
                    └────────┬────────┘
                             │
                    ┌────────v────────┐
                    │ Select Provider  │
                    │ (TaskRouter)     │
                    └────────┬────────┘
                             │
                    ┌────────v────────┐
              ┌─────│ Circuit Breaker │─────┐
              │     │ Open?           │     │
              │Yes  └─────────────────┘  No │
              │                             │
              v                    ┌────────v────────┐
     ┌────────────────┐           │ Call Provider    │
     │ Skip to Next   │           │ via AIGateway    │
     │ Provider       │           └────────┬────────┘
     └────────┬───────┘                    │
              │              ┌─────────────┴──────────────┐
              │              │                            │
              │     ┌────────v────────┐         ┌────────v────────┐
              │     │    Success      │         │    Failure      │
              │     │                 │         │                 │
              │     └────────┬────────┘         └────────┬────────┘
              │              │                           │
              │              v                  ┌────────v────────────┐
              │     ┌─────────────────┐        │ Permanent Quota     │
              │     │ Return Response │        │ Error?              │
              │     │ Log Prompt      │        └───┬────────────┬───┘
              │     └─────────────────┘         Yes│            │No
              │                                    │            │
              │                           ┌────────v──────┐ ┌───v─────────────┐
              │                           │ force_open()  │ │ Log Failure     │
              │                           │ Circuit       │ │ Update Breaker  │
              │                           │ Breaker       │ │ (normal count)  │
              │                           │ (skip retries)│ └────────┬────────┘
              │                           └───────┬───────┘          │
              │                                   │                  │
              │                                   v                  │
              │                                ┌────────v────────┐   │
              └───────────────────────────────>│ More Providers? │<──┘
                                               └────────┬────────┘
                                                   │         │
                                                Yes│         │No
                                                   │         │
                                          ┌────────v──┐  ┌───v──────────┐
                                          │Try Next   │  │Use Keyword   │
                                          │Provider   │  │Fallback      │
                                          └───────────┘  └──────────────┘
```

**Permanent Quota Error Detection**: The `AIGateway._is_permanent_quota_error()` method identifies Google's `RESOURCE_EXHAUSTED` errors containing "quota" as permanent failures. When detected, retries are skipped and `force_open()` immediately opens the circuit breaker for that provider (bypassing the normal failure-count threshold). This prevents wasting time retrying a provider whose quota is exhausted.

### 5.4 Authentication State Diagram

```
                    ┌──────────────┐
                    │  Anonymous   │
                    └──────┬───────┘
                           │
              ┌────────────┴────────────┐
              │                         │
     ┌────────v────────┐     ┌─────────v─────────┐
     │ AUTH_ENABLED=    │     │ AUTH_ENABLED=      │
     │ false            │     │ true               │
     └────────┬────────┘     └─────────┬──────────┘
              │                        │
              v               ┌────────v────────┐
     ┌─────────────────┐     │  Login Screen   │
     │ Auto-inject     │     │  Username/Pass  │
     │ default user    │     │  OAuth Buttons  │
     │ (no login UI)   │     │  Try Sandbox    │
     └────────┬────────┘     └───────┬─────────┘
              │                      │
              │         ┌────────────┼────────────┐
              │         │            │            │
              │    ┌────v─────┐ ┌───v────┐ ┌────v──────┐
              │    │ Password │ │ OAuth  │ │ Sandbox   │
              │    │ Login    │ │ Flow   │ │ (per-     │
              │    │ (JWT +   │ │ (JWT + │ │ session)  │
              │    │ session) │ │ cookie)│ │ 1hr JWT   │
              │    └────┬─────┘ └───┬────┘ └────┬──────┘
              │         │           │            │
              │    ┌────v───────────v────────────v────┐
              │    │         Authenticated            │
              │    │  JWT access token (15m/1h)       │
              │    │  + httpOnly refresh cookie (7d)   │
              └────┤  + legacy session cookie          │
                   └──────────────┬───────────────────┘
                                  │
                   ┌──────────────┼──────────────────┐
                   │              │                   │
              ┌────v─────┐  ┌────v──────┐  ┌────────v────────┐
              │ Silent   │  │ Session   │  │ Sandbox Expired │
              │ Refresh  │  │ Expired   │  │ (1hr → logout)  │
              │ (auto)   │  │ → Login   │  └─────────────────┘
              └──────────┘  └───────────┘
```

### 5.5 App-to-Ollama Inference Sequence

```
┌──────┐    ┌────────┐    ┌──────────┐    ┌──────────┐    ┌────────┐
│Client│    │ FastAPI │    │AIGateway │    │TaskRouter│    │ Ollama │
└──┬───┘    └───┬────┘    └────┬─────┘    └────┬─────┘    └───┬────┘
   │            │              │               │              │
   │ Score Job  │              │               │              │
   │ (provider: │              │               │              │
   │  ollama)   │              │               │              │
   │───────────>│              │               │              │
   │            │ select_      │               │              │
   │            │ provider()   │               │              │
   │            │──────────────────────────────>│              │
   │            │              │  "ollama"      │              │
   │            │<──────────────────────────────│              │
   │            │              │               │              │
   │            │ call(ollama, │               │              │
   │            │  prompt)     │               │              │
   │            │─────────────>│               │              │
   │            │              │ POST /v1/     │              │
   │            │              │ chat/         │              │
   │            │              │ completions   │              │
   │            │              │──────────────────────────────>│
   │            │              │               │              │
   │            │              │               │   LLM        │
   │            │              │               │   inference  │
   │            │              │               │   (local)    │
   │            │              │               │              │
   │            │              │  response     │              │
   │            │              │<──────────────────────────────│
   │            │              │               │              │
   │            │              │ Log: provider │              │
   │            │              │ =ollama,      │              │
   │            │              │ tokens, ms    │              │
   │            │              │────┐          │              │
   │            │              │<───┘          │              │
   │            │  score       │               │              │
   │            │<─────────────│               │              │
   │  result    │              │               │              │
   │<───────────│              │               │              │
```

### 5.6 App-to-Grafana Monitoring Data Flow

```
┌────────────┐    ┌────────┐    ┌──────────┐    ┌─────────┐
│ Frontend   │    │Backend │    │ Grafana  │    │Infinity │
│ Monitoring │    │ API    │    │ Server   │    │Datasrc  │
│ Panel      │    │        │    │          │    │         │
└─────┬──────┘    └───┬────┘    └────┬─────┘    └────┬────┘
      │               │              │               │
      │ GET /api/     │              │               │
      │ monitoring/   │              │               │
      │ grafana-link  │              │               │
      │──────────────>│              │               │
      │               │ GET /api/    │               │
      │               │ health       │               │
      │               │─────────────>│               │
      │               │  200 OK      │               │
      │               │<─────────────│               │
      │  {url, avail: │              │               │
      │   true}       │              │               │
      │<──────────────│              │               │
      │               │              │               │
      │ User clicks   │              │               │
      │ "Open Grafana"│              │               │
      │───────────────────────────────>              │
      │               │              │               │
      │               │         [Dashboard loads]    │
      │               │              │               │
      │               │              │ GET /api/     │
      │               │              │ monitoring/   │
      │               │              │ providers     │
      │               │              │──────────────>│
      │               │              │               │
      │               │              │               │ GET http://
      │               │              │               │ backend:8000
      │               │              │               │ /api/monitoring/
      │               │              │               │ providers
      │               │<────────────────────────────-│
      │               │  {providers: │               │
      │               │   [...]}     │               │
      │               │─────────────────────────────>│
      │               │              │               │
      │               │              │ [Panels       │
      │               │              │  render with  │
      │               │              │  live data]   │
      │               │              │               │
      │               │         [Auto-refresh 30s]   │
```

---

## 6. Data Model

### 6.1 Entity-Relationship Overview

```
User ──1:N──> UserSession
User ──1:N──> Job ──1:N──> ScoreHistory
User ──1:N──> Search
User ──1:N──> Company ──1:N──> SalaryEstimate
User ──1:N──> CvAnalysis
User ──1:N──> CvVersion
User ──1:N──> SearchProfile
User ──1:N──> Config
User ──1:N──> SalaryBenchmark
User ──1:N──> AIPromptLog
User ──1:N──> SeenJob
User ──1:N──> PromptTemplateOverride
User ──1:N──> SkillTrend
User ──1:N──> UserConsent
User ──1:N──> AffiliateClick
         ──1:N──> AdminAuditLog (admin_user_id)
         ──0:N──> SandboxUsage (fingerprint/IP tracking, no FK)
User ──1:N──> RefreshToken
User ──1:N──> SandboxEvent
```

### 6.2 Data Dictionary

#### User Table

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | VARCHAR (UUID) | PK | Unique user identifier |
| username | VARCHAR | UNIQUE, NOT NULL, INDEXED | Login username |
| display_name | VARCHAR | NULLABLE | User display name |
| email | VARCHAR | NULLABLE | User email |
| password_hash | VARCHAR | NULLABLE | bcrypt password hash (NULL for OAuth/sandbox users) |
| salt | VARCHAR | NULLABLE | Per-user password salt (NULL for OAuth/sandbox users) |
| role | VARCHAR | NOT NULL, DEFAULT 'user' | 'root' or 'user' |
| is_active | BOOLEAN | DEFAULT TRUE | Account active flag |
| is_admin | BOOLEAN | DEFAULT FALSE | Admin flag (separate from role for flexibility) |
| provider | VARCHAR | DEFAULT 'local' | Auth provider: 'local', 'linkedin', 'google', 'sandbox' |
| provider_id | VARCHAR | NULLABLE | Provider-specific user ID (for OAuth dedup) |
| avatar_url | TEXT | NULLABLE | Profile picture URL from OAuth provider |
| is_sandbox | BOOLEAN | DEFAULT FALSE | Per-session sandbox user flag |
| sandbox_expires_at | VARCHAR | NULLABLE | ISO timestamp when sandbox session expires |
| api_key | TEXT | NULLABLE | Encrypted per-user AI key |
| created_at | VARCHAR | NOT NULL | ISO timestamp |
| updated_at | VARCHAR | NULLABLE | ISO timestamp |
| last_login_at | VARCHAR | NULLABLE | ISO timestamp |

#### Job Table

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | VARCHAR (UUID) | PK | Unique job identifier |
| user_id | VARCHAR | FK(User.id), INDEXED | Owner |
| title | VARCHAR | | Job title |
| company | VARCHAR | | Company name |
| location | VARCHAR | | Job location |
| salary | VARCHAR | | Raw salary string |
| salary_normalized | JSON | | {annual_min, annual_max, currency, period} |
| url | VARCHAR | INDEXED | Job posting URL |
| description | TEXT | | Full job description |
| status | VARCHAR | INDEXED | Application status |
| applied | BOOLEAN | INDEXED | Has user applied? |
| applied_date | VARCHAR | | When applied |
| match_score | FLOAT | | Composite fit score (0-100) |
| ai_score | FLOAT | | AI-generated score component |
| keyword_score | FLOAT | | Keyword overlap score component |
| scoring_method | VARCHAR | | ai_weighted / keyword_only / etc. |
| scored_at | VARCHAR | INDEXED | When last scored |
| scored_by | VARCHAR | | Provider that scored |
| matched_skills | JSON | | Skills found in both CV and job |
| missing_skills | JSON | | Job skills not in CV |
| essential_skills | JSON | | Must-have skills from job |
| desirable_skills | JSON | | Nice-to-have skills from job |
| sub_scores | JSON | | {skills_match, experience_fit, industry_alignment, seniority_match} |
| deep_analysis | JSON | | Consolidated salary/culture/interview analysis |
| output_folder | VARCHAR | | Path to generated documents |
| ai_provider | VARCHAR | | Provider used for document generation |
| listing_active | BOOLEAN | | Is job posting still live? |
| archived | BOOLEAN | INDEXED | Soft delete |
| created_at | VARCHAR | INDEXED | When job was saved |
| _extra | JSON | | Legacy catch-all field |

#### Config Table

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | INTEGER | PK, AUTO | Row ID |
| name | VARCHAR | NOT NULL, INDEXED | Config key (e.g., 'keywords', 'settings') |
| user_id | VARCHAR | NULLABLE, INDEXED | NULL = global, non-NULL = per-user |
| data | JSON | NOT NULL | Config data object |
| updated_at | VARCHAR | | Last modified |

**Unique Constraint**: `(name, user_id)`

#### AIPromptLog Table

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | INTEGER | PK, AUTO | Log ID |
| user_id | VARCHAR | FK(User.id), INDEXED | Who made the request |
| job_id | VARCHAR | FK(Job.id), NULLABLE | Related job (if applicable) |
| task | VARCHAR | INDEXED | Task type (match_job, analyze_cv, etc.) |
| provider | VARCHAR | | AI provider used |
| model | VARCHAR | | Model name |
| system_prompt | TEXT | | System prompt sent |
| user_prompt | TEXT | | User prompt sent |
| response_text | TEXT | | AI response received |
| temperature | FLOAT | | Temperature parameter |
| max_tokens | INTEGER | | Max tokens parameter |
| input_tokens | INTEGER | | Tokens in request |
| output_tokens | INTEGER | | Tokens in response |
| total_tokens | INTEGER | | Total tokens used |
| created_at | VARCHAR | INDEXED | Timestamp |

**Composite Index**: `(task, created_at)`

#### UserConsent Table

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | INTEGER | PK, AUTO | Row ID |
| user_id | VARCHAR | FK(User.id), INDEXED | Owning user |
| consent_type | VARCHAR | NOT NULL | Type: analytics, ai_processing, data_sharing |
| granted | BOOLEAN | NOT NULL | Whether consent is granted |
| granted_at | VARCHAR | NOT NULL | ISO timestamp when granted/withdrawn |

#### AffiliateClick Table

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | INTEGER | PK, AUTO | Row ID |
| user_id | VARCHAR | FK(User.id), INDEXED | User who clicked |
| url_hash | VARCHAR | NOT NULL | SHA-256 hash of destination URL (privacy) |
| source | VARCHAR | | Click source (e.g., "adzuna") |
| created_at | VARCHAR | NOT NULL, INDEXED | Click timestamp |

#### SandboxUsage Table

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | INTEGER | PK, AUTO | Row ID |
| fingerprint | VARCHAR | INDEXED | Browser fingerprint hash |
| ip_address | VARCHAR | INDEXED | Client IP address |
| date | VARCHAR | NOT NULL | Usage date (YYYY-MM-DD) |
| request_count | INTEGER | DEFAULT 0 | Daily request count |

**Unique Constraint**: `(fingerprint, date)` and `(ip_address, date)`

#### RefreshToken Table

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| token_hash | VARCHAR | PK | SHA-256 hash of the raw refresh token |
| user_id | VARCHAR | FK(User.id), INDEXED | Token owner |
| created_at | VARCHAR | NOT NULL | ISO timestamp |
| expires_at | VARCHAR | NOT NULL | ISO timestamp (7 days from creation) |
| revoked | BOOLEAN | DEFAULT FALSE | Whether token has been revoked |

**Index**: `(user_id, revoked)` for efficient token lookup and cleanup

#### SandboxEvent Table

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | INTEGER | PK, AUTO | Row ID |
| user_id | VARCHAR | FK(User.id), INDEXED | Sandbox user |
| event_type | VARCHAR | NOT NULL, INDEXED | Event type (sandbox_started, feature_used, conversion_started) |
| event_data | JSON | NULLABLE | Additional event context |
| created_at | VARCHAR | NOT NULL | ISO timestamp |

**Index**: `(user_id, event_type)` for conversion funnel queries

#### SalaryEstimate Table (updated columns)

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| employment_type | VARCHAR | NULLABLE | `Contract` or `Permanent` |

**Unique Constraint**: `(company_id, role, user_id)`

*(Additional tables: ScoreHistory, Search, Company, SalaryBenchmark, CvAnalysis, CvVersion, SearchProfile, SeenJob, SkillTrend, PromptTemplateOverride, AdminAuditLog, UserSession — all follow the same FK pattern to User with appropriate indexes.)*

---

## 7. API Specifications

### 7.1 Base URL

- **Development**: `http://localhost:8000/api`
- **Production**: `https://cviper.uk/api` (proxied via frontend nginx to internal backend)
- **Frontend Config**: `VITE_API_BASE` environment variable

### 7.2 Authentication

All requests include credentials:
- **Header**: `Authorization: Bearer <JWT>` (preferred — stateless JWT verification)
- **Header**: `Authorization: Bearer <session_token>` (legacy fallback — DB lookup)
- **Cookie**: `session_token` (httponly, samesite=lax) — legacy fallback
- **Cookie**: `refresh_token` (httponly, samesite=lax, path=/api/auth/refresh) — silent JWT renewal
- **Public endpoints**: No auth required (see Section 3.1)

### 7.3 Core Endpoints

#### Search

| Method | Path | Rate Limit | Description |
|--------|------|-----------|-------------|
| POST | `/api/search` | 10/min | Standard job search |
| POST | `/api/search-stream` | 10/min | Streaming search (SSE) |

**POST /api/search-stream**

Request:
```json
{
  "location": "London",
  "salary_min": 60000,
  "cv_folder": "/path/to/cv",
  "selected_sites": ["reed", "indeed"],
  "selected_career_pages": ["JP Morgan", "Goldman Sachs"],
  "search_keywords": "python developer",
  "search_titles": ["Software Engineer", "Backend Developer"],
  "search_skills": ["Python", "FastAPI", "SQL"],
  "work_mode": "hybrid",
  "distance": 25,
  "posted_after": "2026-03-01",
  "selected_providers": ["google"],
  "profile_id": "uuid-optional"
}
```

Response (SSE stream):
```
event: profile
data: {"skills": [...], "job_titles": [...], "experience_years": 5}

event: site_result
data: {"site": "reed", "jobs": [{...}], "count": 15}

event: site_result
data: {"site": "JP Morgan", "jobs": [{...}], "count": 8}

event: complete
data: {"total_jobs": 23, "sources_queried": 2}
```

#### Jobs

| Method | Path | Rate Limit | Description |
|--------|------|-----------|-------------|
| POST | `/api/apply` | None | Generate CVs/cover letters for jobs |
| POST | `/api/save-jobs` | None | Save search results to tracking |
| GET | `/api/saved-jobs` | None | List all tracked jobs |
| GET | `/api/saved-jobs/{job_id}` | None | Get single job details |
| PUT | `/api/saved-jobs/{job_id}` | None | Update job fields |
| DELETE | `/api/saved-jobs/{job_id}` | None | Delete a saved job |
| POST | `/api/score-job` | 30/min | AI scoring for single job |
| POST | `/api/bulk-score` | None | Score multiple jobs |
| GET | `/api/score-history/{job_id}` | None | Historical scores |
| POST | `/api/clear-scores` | None | Clear scores from jobs |

**POST /api/apply**

Request:
```json
{
  "job_ids": ["uuid-1", "uuid-2"],
  "cv_folder": "/path/to/cv",
  "provider": "anthropic"
}
```

Response:
```json
{
  "message": "2 applications processed",
  "applications": [
    {
      "job_id": "uuid-1",
      "status": "success",
      "documents": {
        "cv_path": "/output/job1/cv.txt",
        "cover_letter_path": "/output/job1/cover_letter.txt",
        "ats_score": 82
      }
    }
  ]
}
```

#### CV Analysis

| Method | Path | Description |
|--------|------|-------------|
| POST | `/api/analyze-cv` | Analyse CV from folder |
| POST | `/api/analyze-cv-upload` | Analyse uploaded files |
| POST | `/api/analyze-cv-text` | Analyse raw CV text |
| POST | `/api/analyze-cv-multi` | Multi-provider analysis |
| POST | `/api/compare-cvs` | Compare multiple CVs |
| POST | `/api/cv/optimize-bullets` | AI bullet point optimization (FR-031, 10/min) |
| GET | `/api/cv-analyses` | List saved analyses |

#### AI Insights

| Method | Path | Rate Limit | Description |
|--------|------|-----------|-------------|
| POST | `/api/deep-analysis/{job_id}` | 30/min | Salary + culture + interview analysis |
| POST | `/api/portfolio-review` | 30/min | Career pattern analysis |
| POST | `/api/interview-prep/{job_id}` | 30/min | Interview preparation materials |
| GET | `/api/skills-gap` | 30/min | Skills gap identification |
| GET | `/api/skills-trending` | None | Trending skills over time |
| GET | `/api/analytics` | 30/min | Search analytics dashboard |

#### Companies & Salaries

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/companies` | List companies |
| POST | `/api/companies/{id}/salary-estimate` | Estimate salary for role |
| POST | `/api/companies/bulk-salary-estimate` | Bulk estimates |
| GET | `/api/salary-benchmarks` | Curated benchmark data |
| GET | `/api/salary-benchmarks/for-role` | Benchmarks for specific role |
| GET | `/api/company-boards` | Career page configs |
| GET | `/api/company-boards/industries` | Industry filter options |

#### Configuration

| Method | Path | Description |
|--------|------|-------------|
| GET/PUT | `/api/keywords` | User keywords config |
| GET/PUT | `/api/config/settings` | User settings |
| GET/PUT | `/api/job-sites` | Job board configuration |
| GET | `/api/ai-providers` | List AI providers |
| PUT | `/api/ai-keys/{provider_id}` | Update provider API key |
| PUT | `/api/ai-provider-model/{provider_id}` | Update provider model selection (FR-026) |
| POST/GET | `/api/ai-routing` | Provider routing config |

#### Monitoring

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/monitoring/providers` | Provider health & metrics |
| GET | `/api/monitoring/usage` | Daily usage vs limits |
| GET | `/api/monitoring/logs` | Unified log stream |
| GET | `/api/monitoring/grafana-link` | Grafana dashboard deep-link |
| POST | `/api/frontend-errors` | Frontend error ingestion |
| POST | `/api/frontend-metrics` | Frontend metrics (Core Web Vitals) |

#### GDPR & Privacy

| Method | Path | Rate Limit | Description |
|--------|------|-----------|-------------|
| GET | `/api/gdpr/export` | 2/hour | Export all user data as JSON |
| DELETE | `/api/gdpr/delete-account` | None | Password-verified account erasure |
| GET | `/api/gdpr/consents` | None | List user's consent records |
| POST | `/api/gdpr/consents` | None | Record or withdraw consent |

#### Authentication (Additional)

| Method | Path | Rate Limit | Description |
|--------|------|-----------|-------------|
| POST | `/api/auth/forgot-password` | 5/min | Generate password reset token and send email |
| POST | `/api/auth/reset-password` | 5/min | Validate token and set new password |

#### Authentication (JWT/OAuth)

| Method | Path | Rate Limit | Description |
|--------|------|-----------|-------------|
| GET | `/api/auth/providers` | — | List configured OAuth providers |
| GET | `/api/auth/linkedin` | — | Redirect to LinkedIn OAuth authorization |
| GET | `/api/auth/linkedin/callback` | — | Handle LinkedIn OAuth callback, issue JWT |
| GET | `/api/auth/google` | — | Redirect to Google OAuth authorization |
| GET | `/api/auth/google/callback` | — | Handle Google OAuth callback, issue JWT |
| POST | `/api/auth/refresh` | 10/min | Refresh access token via httpOnly cookie (rotates refresh token) |
| GET | `/api/auth/me` | — | Get current user profile (requires auth) |
| POST | `/api/auth/sandbox` | 5/min | Create per-session sandbox user with seed data |

#### Affiliate & Tracking

| Method | Path | Description |
|--------|------|-------------|
| POST | `/api/track-click` | Record affiliate outbound click (URL hash, source) |

#### System

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/version` | Return app version and git commit hash |

#### Admin (Root Only)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/admin/users` | List all users |
| POST | `/api/admin/users` | Create user |
| PUT | `/api/admin/users/{id}` | Update user |
| DELETE | `/api/admin/users/{id}` | Delete user |
| GET/PUT | `/api/admin/defaults/job-boards` | Default job boards |
| GET/PUT | `/api/admin/defaults/career-pages` | Default career pages |
| POST | `/api/admin/cleanup` | Clean old records |
| GET | `/api/admin/backup` | Trigger DB backup |

#### Service Control (Root Only)

| Method | Path | Description |
|--------|------|-------------|
| POST | `/api/service/backend` | Stop or restart the backend process (admin only, `require_root`) |
| POST | `/api/service/frontend` | Stop, start, or restart the frontend process (admin only, `require_root`) |

### 7.4 Error Codes

| Code | Meaning | Example |
|------|---------|---------|
| 200 | Success | Standard response |
| 201 | Created | New resource created |
| 400 | Bad Request | Invalid request body |
| 401 | Unauthorized | Missing or invalid session |
| 403 | Forbidden | Insufficient role permissions |
| 404 | Not Found | Resource doesn't exist |
| 422 | Validation Error | Pydantic schema violation |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Unhandled exception |

**Error Response Format**:
```json
{
  "detail": "Human-readable error message"
}
```

---

## 8. User Interface Specifications

### 8.1 Navigation Structure

```
┌──────────────────────────────────────────────────────────────────────┐
│ TopNav: Logo + Version + Git Hash | AI Provider Indicator | User Menu │
├──────────────────────────────────────────────────────────────────────┤
│ Tab Bar (progressive disclosure — new users see fewer tabs):          │
│   Focused:  CV Analysis | Job Search | Applications | Settings       │
│   + Standard: [▼ Insights: Companies, Career Insights, Skills]       │
│   + Full:     [▼ More: Prompt Lab*, My Requests, FAQ, Behind CViper] │
│               [Admin]** | [Monitoring]* (in More dropdown)            │
├──────────────────────────────────────────────────────────────────────┤
│ WizardMode ribbon (if active): Step N of M — instruction — [Next]    │
│ ProgressStepper: Upload CV → Review Profile → Find Jobs → Apply      │
├──────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  Main Content Area                                                    │
│  (Active tab renders here)                                           │
│                                                                       │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │  Tab-specific content                                        │    │
│  │  (or empty state with icon + CTA if no data)                │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                                                                       │
└──────────────────────────────────────────────────────────────────────┘

*  Monitoring and Prompt Lab visible only when Advanced Mode is enabled
** Admin tab visible only to root users
```

**Layout**: Horizontal TopNav with tab buttons and dropdown groups (no sidebar). Ungrouped tabs render directly; grouped tabs render inside "Insights" and "More" dropdown menus (click to open, click-outside to close). WizardMode ribbon appears below tabs when wizard is active. ProgressStepper appears as ambient progress tracker. Git commit hash displayed beneath version number — injected at build time via Vite define.

**Progressive Disclosure**: Tabs have a `tier` field (focused/standard/full). The `useTabTier` hook derives the current tier from onboarding progress (`upload` + `search` → standard; + `apply` → full) or from the "Show all tabs" user preference. Admin users always see full tier. TopNav filters tabs by `tierLevel` before applying advanced mode filtering.

**Advanced Mode**: Defaults to `false`. Toggled via Settings > Preferences. Persisted via `userStorage.getItemGlobal` (readable before login) and backend config (`advanced_mode`). When off, Monitoring and Prompt Lab tabs are hidden from the More dropdown.

### 8.2 Tab Specifications

#### CV Analysis Tab

| Section | Components | Behaviour |
|---------|-----------|-----------|
| Provider Alert | Warning card with "Go to Settings" CTA | Shown only when no AI provider configured |
| Provider Indicator | Green dot + provider name inline next to Analyze button | Compact status, replaces full provider card (P1) |
| File Upload | Drag-and-drop zone, file list, folder browser | Multi-file upload, supported: PDF/DOCX/TXT |
| Empty State | Icon + "No CV Analyzed Yet" + guidance text | Shown when no analysis, no files, not analyzing |
| Analysis Results | Skills, titles, experience, strengths, achievements | Expandable sections, editable skill overrides |
| Search Profiles | Saved profile list, compare, load into search | Profile manager with CRUD operations |

#### Search Tab

| Section | Components | Behaviour |
|---------|-----------|-----------|
| CV Data Flow | "Suggested from CV" badge, "analyze your CV" link, "Re-analyze CV" link | Links navigate to CV Analysis tab (P3) |
| Job Sources (collapsible) | Collapsible section wrapping Job Boards + Career Pages | Auto-expands when sources load; collapsed by default (P4) |
| Job Boards | Checkbox grid (200px min columns), toggle enabled/disabled, quick-add | Auto-save on toggle (1s debounce) |
| Career Pages | Checkbox grid with star favourites, industry/ATS labels, industry filter dropdown, delete button | Fixed height scroll (320px), no layout shift. Companies filtered to selected/favourited when >10 items |
| Search Criteria | Location, salary min, keywords, job titles, work mode, distance, posted after | Auto-populated from CV analysis |
| Empty State | Icon + "No Search Results Yet" + guidance text | Shown before first search, hidden during loading |
| Results | Table with fit score badges, company, title, location, salary | Sortable, filterable, colour-coded scores |

#### Applications Tab

| Section | Components | Behaviour |
|---------|-----------|-----------|
| Empty State | Mailbox icon + "No Applications Yet" + "Start Job Search" CTA | Navigates to Search tab |
| Filters | Status dropdown, show archived toggle, text search | Real-time filtering |
| Job Cards | Score badge, status dropdown, action menu | Expandable score detail |
| Actions | Apply, Multi-Generate, Interview Prep, Follow-up Email, Delete | Provider selection per action |
| Views | Table (default) or Kanban (drag-drop) | Toggle button |

#### Companies Tab

| Section | Components | Behaviour |
|---------|-----------|-----------|
| Empty State | Building icon + "No Salary Estimates Yet" + guidance text | Shown when no estimates exist; filter-specific message when filters hide all results |
| Estimates Table | Company, role, min/max/median salary, confidence, provider | Column sorting |
| Filters | By company, by role, grouping mode | Real-time filtering |
| Bulk Actions | Select, re-estimate, delete | Multi-select checkboxes |
| Benchmarks | Reference salary data by sector | Expandable section |

#### Settings Tab

| Section | Sub-Tab | Content |
|---------|---------|---------|
| Sub-Navigation | 4 tabs: General, AI Providers, Search & Keywords, Preferences | Horizontal tab bar with active indicator (P6) |
| General | CV folder, location, output folder, base CV selector, CV editor, CV format & layout, CV library | Auto-save on change (1s debounce) |
| AI Providers | Provider priority (drag-reorder), API keys management, signup links, model selection | Active provider badge, masked keys, set/change/remove. "Get Key" link to provider signup page when unconfigured; green hint for free-tier providers (Ollama, GitHub, Google, OpenRouter, Mistral). Model dropdown per provider when `available_models` present (FR-026) |
| Search & Keywords | Skills & technologies, job titles, keyword tags, industry preferences (target/avoided) | Auto-save on change |
| Preferences | Advanced Mode toggle, Change Password (auth-enabled only) | Persists to localStorage + backend config |

### 8.3 Validation Rules

| Field | Rule |
|-------|------|
| Location | Required for search; free text |
| Salary Min | Optional; positive integer |
| Keywords | Optional; comma-separated |
| Job Titles | Optional; array of strings |
| CV Folder | Must exist on filesystem |
| API Key | Format validated per provider (e.g., OpenAI starts with `sk-`) |
| Username | 3-30 chars, alphanumeric + underscore, unique |
| Password | 8+ chars minimum |
| URL | Must pass SSRF validation (scheme, hostname, IP check) |

### 8.4 Empty States

All primary tabs display contextual empty states when no data is present. Empty states use the `.empty-state` CSS class (defined in theme.css) and follow a consistent pattern:

| Tab | Condition | Icon | Heading | CTA |
|-----|-----------|------|---------|-----|
| CV Analysis | No analysis, no files uploaded, not analyzing | `document` (40px) | "No CV Analyzed Yet" | Guidance text: upload CV above |
| Job Search | `searchResults.length === 0 && !loading` | `search` (40px) | "No Search Results Yet" | Guidance text: configure criteria and search |
| Applications | `savedJobs.length === 0` | `mailbox` (40px) | "No Applications Yet" | "Start Job Search" button → Search tab |
| Companies (no data) | `allEstimates.length === 0` | `building` (40px) | "No Salary Estimates Yet" | Guidance text: add company above |
| Companies (filtered) | `filtered.length === 0 && allEstimates.length > 0` | None | "No Matching Estimates" | Guidance text: broaden filters |
| Search Results (filtered) | `filteredAndSortedResults.length === 0 && searchResults.length > 0` | None | "No jobs match your current filters" | "Clear all filters" link |

**Design Principles**:
- Icon size: 40px using the Icon component
- Heading: `<h3>` element
- Body: `<p>` with actionable guidance
- CTA buttons navigate to the relevant tab or action
- Empty states are hidden immediately when data loads or user starts an action

### 8.5 Shared Job Table Components

Job data appears in two views (Job Search results table and Applications tracking table). To prevent visual drift, shared rendering logic is extracted into single-source components:

| Component | Path | Purpose | Used by |
|-----------|------|---------|---------|
| `SalaryDisplay` | `src/components/SalaryDisplay.jsx` | Contract day/hourly rates with annual equivalent, raw salary, AI estimates | ApplicationCard, SearchResultsList |
| `SourceBadge` | `src/components/SourceBadge.jsx` | Source name with career page icon, truncated with ellipsis | ApplicationCard, SearchResultsList |
| `DescriptionPreview` | `src/components/DescriptionPreview.jsx` | Truncated job description (80 char default) for collapsed rows | ApplicationCard, SearchResultsList |
| `StatusBadge` | `src/components/StatusBadge.jsx` | Coloured pill badges (SCORED, Active, Expired, NEW, SAVED, contract type) with variant/size/outline props | ApplicationCard, SearchResultsList |
| `FitScoreBadge` | `src/components/FitScoreBadge.jsx` | Score badge with AI/keyword method indicator, click-to-expand analysis panel | ApplicationCard, SearchResultsList |

**Rule**: Any rendering logic that appears in both job table views must be extracted into a shared component. Inline duplication between these files is a CI-time code review flag.

---

## 9. Non-Functional Requirements

### 9.1 Performance

| Metric | Target |
|--------|--------|
| Search response (first results via SSE) | < 5 seconds |
| Search completion (all sources) | < 60 seconds |
| Single job scoring | < 10 seconds |
| Document generation (CV + cover letter) | < 30 seconds |
| Page load (frontend) | < 3 seconds |
| API response (non-AI endpoints) | < 500ms |

### 9.2 Scalability

| Aspect | Current | Notes |
|--------|---------|-------|
| Concurrent users | 1-50+ | PostgreSQL with connection pooling in production; SQLite single-writer in local dev |
| Database size | Up to 32GB | Azure PostgreSQL Burstable B1ms tier (expandable) |
| Job listings | 10,000+ per user | No hard limit |
| AI providers | 9 supported | Extensible via ProviderRegistry |
| Career pages | 80+ configured | Extensible via seed data |

### 9.3 Availability

| Component | Target | Mechanism |
|-----------|--------|-----------|
| Backend API | 99.5% | Azure Container Apps auto-restart, liveness probes |
| AI Service | 99.9% | 9-provider failover chain + keyword fallback |
| Frontend | 99.9% | Static files served via nginx |
| Database | 99.5% | Azure PostgreSQL managed backups (7-day retention), admin backup endpoint |

### 9.4 Ollama Universal Access (NFR-OLL-ACCESS)

**Traces to**: BR-025

Ollama is a shared infrastructure service requiring zero per-user provisioning. All users (existing, new, any role) have immediate, equal access with no configuration. When Ollama is unavailable, the system silently falls through to the next provider. Full acceptance criteria are defined in FR-011 (Section 4.11).

**Implementation Note**: Ollama access is inherently universal because `ProviderRegistry` detects availability at startup and exposes it to all AI routing decisions. No user-level Ollama settings, entitlements, or feature flags exist or should be introduced.

### 9.5 Logging & Monitoring

| Type | Implementation |
|------|---------------|
| Structured Logging | JSON format via `JsonLogFormatter`, stdout + optional rotating file |
| Request Correlation | X-Request-ID header, `request_id_var` context var |
| User Attribution | `user_id` field added to every HTTP request log via `RequestLoggingMiddleware` |
| Sensitive Data | Passwords, API keys, tokens redacted in logs |
| Frontend Errors | Reported to /api/frontend-errors |
| Web Vitals | Reported to /api/frontend-metrics |
| AI Prompt Audit | AIPromptLog table (full prompt + response) |
| Grafana Dashboard | Unified dashboard with 8 base panels + 6 security panels (collapsible row) |
| Security Event Logging | 12 structured `event_type` values emitted from auth, RBAC, token budget, AI gateway, and key resolution modules |

**Structured Log Extra Fields** (allowed in `JsonLogFormatter`):

```
method, path, status_code, duration_ms, client_ip, user_agent,
source, error_type, component, url, stack_trace,
user_id, event_type, provider, tokens_used, tokens_limit,
key_source, username, role
```

**Security Event Types**:

| Event Type | Source | Level | Trigger |
|-----------|--------|-------|---------|
| `auth_login_success` | routes/auth.py | INFO | Successful login |
| `auth_login_failure` | routes/auth.py | WARNING | Invalid credentials |
| `auth_logout` | routes/auth.py | INFO | User logout |
| `auth_register` | routes/auth.py | INFO | New user registration |
| `auth_sandbox_login` | routes/auth.py | INFO | Sandbox session started |
| `auth_password_change` | routes/auth.py | INFO | Password changed |
| `auth_session_expired` | auth.py | INFO | Token expired or idle timeout (includes `reason` field) |
| `rbac_denial` | auth.py | WARNING | Non-root user attempted admin action |
| `token_budget_warning` | ai/token_budget.py | WARNING | User at 80%+ of daily budget |
| `token_budget_exceeded` | ai/gateway.py | WARNING | User exceeded daily budget |
| `ai_call_complete` | ai/gateway.py | INFO | AI call succeeded (includes provider, tokens_used) |
| `key_resolution` | ai/user_keys.py | INFO | Key resolved (includes key_source: user_key or server_default) |

### 9.6 Disaster Recovery

| Scenario | Recovery |
|----------|---------|
| Database corruption | Azure PostgreSQL automated backups (7-day retention); Admin backup endpoint |
| AI provider outage | Automatic failover to next provider in chain |
| All AI providers down | Keyword-only mode (no AI dependency) |
| Deployment failure | Azure Container Apps revision rollback; CI gate + manual deploy approval prevents broken deploys |
| Data loss | CSV/XLSX export; admin archive endpoint |

---

## 10. Test Strategy & Test Cases

### 10.1 Test Plan Overview

| Layer | Framework | Test Count | Execution |
|-------|----------|-----------|-----------|
| Backend Unit/Integration | pytest + pytest-xdist | 1968+ tests, 36 files | Parallel (`-n auto --dist worksteal`) |
| Frontend Unit/Component | Vitest + React Testing Library | 527+ tests, 30 files | Threaded pool |
| CI Pipeline | GitHub Actions | 9 jobs (with paths-filter for PR skipping) | Push/PR to main/develop |
| Security Scanning | Snyk + Aikido + Custom | 4 scan types | Every CI run |

### 10.2 Test Environment

| Aspect | Configuration |
|--------|--------------|
| Backend DB | In-memory SQLite with StaticPool (session-scoped) |
| Data Isolation | `clean_db_between_tests` autouse fixture (bulk deletes 15+ tables) |
| Network Isolation | `_mock_outbound_http` blocks all real HTTP calls |
| AI Isolation | `_block_ai_gateway` blocks all AI calls (opt-out: `@pytest.mark.allow_ai_gateway`); `FakeAIGateway` enforces pure fakes over mocks for LLM tests |
| Data Cleanup | Conditional: skip 15-table DB wipe for tests that don't use database fixtures; always clear in-memory caches |
| Rate Limiter | Reset between tests to prevent 429 false positives |
| Frontend DOM | jsdom environment |
| Frontend Fetch | Globally mocked via `vi.fn()` |

### 10.3 Test Categories & Key Test Cases

#### Category 1: Search & Scraping (22 test files)

| Test Case | File | Purpose | Traces To |
|-----------|------|---------|-----------|
| Search returns results from multiple boards | test_search_comprehensive.py | End-to-end search workflow | FR-001 |
| Deduplication by URL prevents duplicates | test_search_comprehensive.py | URL-based dedup | BR-001 |
| Location filter respects distance radius | test_location_filter.py | Geo filtering | FR-001 |
| Career page handler dispatch chain | test_career_search.py | ATS handler selection | FR-007 |
| Workday URL corrections validated | test_grey_box_fixes.py | Seed data correctness | FR-007 |
| Industry tag on every seed company | test_industry_coverage.py | Data completeness | BRL-007 |
| Date parsing handles all formats | test_date_parsing.py | Posted date extraction | FR-001 |
| Keywords expanded with synonyms | test_search_keywords.py | Keyword service | FR-001 |

#### Category 2: AI & Scoring (30 test files)

| Test Case | File | Purpose | Traces To |
|-----------|------|---------|-----------|
| Hybrid scoring: 70% AI + 30% keyword | test_fit_score_weighted.py | Scoring formula | FR-003 |
| Fallback to keyword-only when AI fails | test_fallback_subscores.py | Resilience | BR-003 |
| Priority-based routing selects correct provider | test_tiered_routing.py | Provider selection | FR-006 |
| Provider priority order respected | test_priority_routing.py | Selection ordering | FR-006 |
| Circuit breaker skips failing providers; force_open on permanent quota errors | test_circuit_breaker.py | Fault tolerance | FR-006 |
| Prompt sanitisation strips injections | test_prompt_sanitization.py | Security | 3.5 |
| Score history tracked per scoring run | test_score_history.py | Audit trail | FR-003 |
| Batch scoring runs in parallel | test_batch_scoring.py | Performance | FR-003 |

#### Category 3: Documents (10 test files)

| Test Case | File | Purpose | Traces To |
|-----------|------|---------|-----------|
| CV generation produces valid output | test_cv_generation_quality.py | Quality gate | FR-004 |
| DOCX/PDF export creates valid files | test_document_generator.py | File format | FR-004 |
| ATS score calculation matches expectations | test_ats_scoring.py | ATS scoring | FR-004 |
| Document editing persists changes | test_document_editing.py | Edit workflow | FR-004 |
| Multi-provider generation creates variants | test_multi_generate.py | Comparison | FR-004 |

#### Category 4: Security (13 test files)

| Test Case | File | Purpose | Traces To |
|-----------|------|---------|-----------|
| bcrypt password hashing verified | test_auth.py | Password security | 3.1 |
| Session expiry enforced (24h/30d) | test_auth.py | Session TTL | 3.1 |
| Rate limits enforced per endpoint | test_rate_limiting.py | Abuse prevention | 3.4 |
| SSRF blocked for private IPs | test_url_validation.py | SSRF prevention | 3.3 |
| Path traversal attacks prevented | test_path_traversal.py | File system security | 3.3 |
| User data isolation verified | test_user_data_isolation_ac.py | Multi-tenant | 3.6 |
| API key isolation per user | test_api_key_isolation.py | Credential security | 3.2 |
| Registration validation rules | test_registration.py | Input validation | 8.3 |
| RBAC enforcement on admin + service control endpoints (26 tests) | test_rbac_enforcement.py | Router-level RBAC | BR-035, BR-070 |
| Key encryption at rest + rotation (6 tests) | test_key_encryption_at_rest.py | Credential encryption | BR-036 |
| Token budget enforcement (9 tests) | test_token_budget.py | Budget limits | BR-037 |
| Data isolation on new tables (SkillTrend, ScoreHistory, SalaryEstimate) | test_data_isolation_new_tables.py | Multi-tenant completeness | BR-032 |
| Auth event structured logging (9 tests) | test_auth_event_logging.py | Observability | BR-038 |
| GDPR data export (comprehensive JSON) | test_gdpr.py | Data subject rights | FR-019 |
| GDPR account erasure (cascading delete) | test_gdpr.py | Right to erasure | FR-019 |
| GDPR consent management (grant/withdraw) | test_gdpr.py | Consent tracking | FR-019 |
| GDPR security (IP anonymisation, lockout, prompt encryption) | test_gdpr_security.py | GDPR hardening | FR-019 |
| Sandbox abuse prevention — repository (10 tests) | test_sandbox_abuse.py | FP/IP tracking CRUD | FR-020 |
| Sandbox abuse prevention — endpoints (5 tests) | test_sandbox_abuse.py | Daily limit enforcement | FR-020 |
| Sandbox abuse prevention — truncation (15 tests) | test_sandbox_abuse.py | Output capping | FR-020 |
| Password reset flow (22 tests) | test_password_reset.py | Forgot/reset/token/email | FR-023 |
| Adzuna integration (25 tests) | test_adzuna.py | API, affiliate URLs, tracking | FR-021 |

#### Category 4b: Cross-user Data Isolation

| Test Case | File | Purpose | Traces To |
|-----------|------|---------|-----------|
| Backend repo functions filter by user_id | test_user_isolation_audit.py | Verify all repo functions scope by user | FR-037, BR-075 |
| Frontend userStorage namespaces per user | userStorage.test.js | Verify key namespacing and purge | FR-037, BR-075 |
| API contract response shapes validated | test_api_contracts.py | Response shape regression | FR-038, BR-076 |

#### Category 5: Data Layer (9 test files)

| Test Case | File | Purpose | Traces To |
|-----------|------|---------|-----------|
| CRUD operations for all entities | test_repositories.py | Data integrity | 6.2 |
| Config scoping (global vs per-user) | test_configuration.py | Multi-tenant config | 6.2 |
| Salary benchmark CRUD | test_salary_benchmarks.py | Salary data | FR-008 |
| Search profile save/load | test_saved_searches.py | Profile persistence | FR-001 |

#### Category 6: Frontend (30 test files, 527+ tests)

| Test Case | File | Purpose | Traces To |
|-----------|------|---------|-----------|
| Tab navigation switches views | App.test.jsx | Navigation | 8.1 |
| Advanced mode hides/shows dev tabs | App.test.jsx | P5 toggle | FR-016 |
| Empty states render on CV/Search/Companies | App.test.jsx | P7 empty states | FR-018 |
| ProgressStepper renders and navigates | ProgressStepper.test.jsx | P2 onboarding | FR-013 |
| Error boundary catches render errors | App.test.jsx | Resilience | 8.1 |
| API hook caches and deduplicates | useApi.test.js | Performance | 7.2 |
| AI providers load with retry | useAIProviders.test.js | Provider management | FR-006 |
| Fit score badge renders correctly | FitScoreBadge.test.jsx | Score display | FR-003 |
| Login form validates input | LoginScreen.test.jsx | Auth UI | 3.1 |
| Monitoring panel shows providers | MonitoringPanel.test.jsx | Monitoring UI | FR-009 |
| Board layout: 3-zone structure | SearchForm tests | Career page grid | 8.2 |
| Selection and deletion workflow | SearchForm tests | Board management | 8.2 |

#### Category 7: JWT/OAuth/Migration

| Test Case | File | Purpose | Traces To |
|-----------|------|---------|-----------|
| JWT access token creation and claims | test_jwt_utils.py | Token lifecycle | FR-027 |
| JWT decode rejects expired/tampered tokens | test_jwt_utils.py | Token security | FR-027 |
| Refresh token rotation (create/validate/revoke) | test_jwt_utils.py | Rotation pattern | BR-062 |
| Cleanup expired refresh tokens | test_jwt_utils.py | Maintenance | BR-062 |
| OAuth provider configuration detection | security/test_oauth.py | Provider registry | FR-027 |
| OAuth user info extraction (LinkedIn/Google) | security/test_oauth.py | Profile parsing | BR-063 |
| Account migration transfers all user data | test_account_migration.py | Data migration | BR-065 |
| SandboxBanner renders countdown timer | SandboxBanner.test.jsx | Timer UI | BR-067 |
| ExampleBadge renders for example source | ExampleBadge.test.jsx | Badge UI | FR-030 |
| SignUpPrompt modal interactions | SignUpPrompt.test.jsx | Prompt UI | FR-030 |

#### Category 8: Grafana Deployment Tests

| Test ID | Test Case | Purpose | Expected Result | Traces To |
|---------|-----------|---------|-----------------|-----------|
| TC-GRAF-001 | Grafana container starts and is healthy | Verify automated deployment | `GET /api/health` returns `{"database":"ok"}` within 30s of container start | FR-010, BR-013 |
| TC-GRAF-002 | Infinity datasource auto-provisioned | Verify data source connectivity | `GET /api/datasources` includes "CViper API" entry with type `yesoreyeram-infinity-datasource` | FR-010, BR-022 |
| TC-GRAF-003 | cviper-unified dashboard exists on startup | Verify dashboard provisioning | `GET /api/dashboards/uid/cviper-unified` returns 200 with 6 panels | FR-010, BR-021 |
| TC-GRAF-004 | Dashboard panels display live data | Verify end-to-end data flow | Provider Health Overview table shows at least 1 row when backend has configured providers | FR-009, BR-021 |
| TC-GRAF-005 | Backend grafana-link endpoint returns valid URL | Verify integration | `GET /api/monitoring/grafana-link?provider=all&timeRange=1h` returns `{available: true, url: "http://..."}` | FR-009, BR-013 |
| TC-GRAF-006 | Frontend shows active Grafana button | Verify UI integration | Monitoring tab "Open Grafana" button is enabled (not greyed out) when GRAFANA_URL is set | FR-009 |
| TC-GRAF-007 | Grafana admin password set from env var | Verify security | Login with `admin` / `${GRAFANA_ADMIN_PASSWORD}` succeeds; default `admin/admin` does not | 3.7 |
| TC-GRAF-008 | Graceful degradation when Grafana unavailable | Verify resilience | Backend returns `{available: false}` when Grafana is down; frontend shows info banner, no errors | FR-010 |

#### Category 9: Ollama Deployment Tests

| Test ID | Test Case | Purpose | Expected Result | Traces To |
|---------|-----------|---------|-----------------|-----------|
| TC-OLL-001 | Ollama container starts and API is reachable | Verify automated deployment | `GET {OLLAMA_BASE_URL}/api/tags` returns 200 within 60s of container start | FR-011, BR-019 |
| TC-OLL-002 | Configured model auto-downloaded | Verify model loading | `GET /api/tags` response includes model matching `OLLAMA_MODEL` env var (default: llama3.2) | FR-011, BR-023 |
| TC-OLL-003 | Backend detects Ollama as available provider | Verify provider registration | `GET /api/ai-providers` includes entry with `{id: "ollama", configured: true}` | FR-011, BR-019 |
| TC-OLL-004 | Ollama responds to inference request | Verify model works | `POST /v1/chat/completions` with simple prompt returns non-empty response with valid JSON | FR-011, BR-023 |
| TC-OLL-005 | App-to-Ollama integration: scoring works | Verify end-to-end | `POST /api/score-job` with `provider=ollama` returns valid match_score (0-100) | FR-011, FR-003 |
| TC-OLL-006 | Model switch via API | Verify runtime config | `POST /api/ollama/set-model {model_name: "mistral"}` succeeds; subsequent calls use new model | FR-011 |
| TC-OLL-007 | Ollama metrics visible in Grafana | Verify monitoring integration | Grafana Provider Health table includes row with provider="ollama", showing totalCalls and avgLatencyMs | FR-011, FR-009 |
| TC-OLL-008 | Graceful fallback when Ollama unavailable | Verify resilience | When Ollama container is stopped, AI requests fall through to next provider in chain; no 500 errors | FR-011, FR-006 |
| TC-OLL-009 | Ollama status endpoint works | Verify health check | `POST /api/ollama/status` returns `{status: "available", model: "llama3.2"}` when running | FR-011 |
| TC-OLL-010 | Universal Ollama access (parameterised: existing user, new user, standard role, admin role) | Verify all user types have immediate, equal Ollama access with no provisioning | For each variant: `GET /api/ai-providers` includes `{id: "ollama", configured: true}` and `POST /api/score-job` with `provider=ollama` returns a valid score. No migration, enablement, or role-based restriction applies | FR-011, BR-025 |

#### Category 10: Integration Tests (Cross-Component)

| Test ID | Test Case | Purpose | Expected Result | Traces To |
|---------|-----------|---------|-----------------|-----------|
| TC-INT-001 | Full stack deployment (backend + frontend + Grafana + Ollama) | Verify all services start | All 4 containers healthy within 120s of `docker compose up` | BR-024 |
| TC-INT-002 | Ollama inference logged and visible in Grafana | Verify monitoring pipeline | After Ollama scoring, Grafana "Recent AI Requests" table shows entry with provider=ollama | FR-009, FR-011 |
| TC-INT-003 | Provider failover: Ollama → cloud | Verify resilience chain | When Ollama is stopped mid-session, next scoring request automatically uses cloud provider | FR-006, FR-011 |
| TC-MON-001 | Backend metrics appear in all Grafana panels | Verify full dashboard | After 5+ AI calls, all 6 dashboard panels show non-zero data | FR-009, FR-010 |

### 10.4 BRD Feature ID → FSD Functional Requirement Mapping

| BRD F-ID | Description | FSD FR-ID |
|----------|-------------|-----------|
| F-001 | Multi-source job search | FR-001 |
| F-002 | AI-powered CV analysis | FR-002 |
| F-003 | Job-to-CV fit scoring | FR-003 |
| F-004 | Tailored CV/cover letter generation | FR-004 |
| F-005 | ATS compatibility scoring | FR-004 (subsection) |
| F-006 | Skills gap analysis & dashboard | FR-003, FR-008 |
| F-007 | Interview preparation | FR-004 (AI insights) |
| F-008 | Application tracking & reminders | FR-005 |
| F-009 | Saved search profiles & dedup | FR-001 (subsection) |
| F-010 | Job comparison & Kanban | FR-005 (subsection) |
| F-011 | Multi-user auth with RBAC | Section 3.1 |
| F-012 | Rate limiting, SSRF, URL validation | Sections 3.3, 3.4 |
| F-013 | Company salary estimation | FR-008 |
| F-014 | AI provider routing (priority-based) | FR-006 |
| F-015 | Monitoring dashboard (Grafana) | FR-009 |
| F-016 | Prompt Lab | FR-009 (subsection) |
| F-017 | Admin panel | Section 7.3 (Admin endpoints) |
| F-018 | Automated Grafana deployment | FR-010 |
| F-019 | Automated Ollama deployment | FR-011 |
| F-020 | Simplified CV Analysis page | FR-012 |
| F-021 | Guided onboarding stepper | FR-013 |
| F-022 | CV-to-Search data flow | FR-014 |
| F-023 | Collapsible Job Sources | FR-015 |
| F-024 | Advanced Mode toggle | FR-016 |
| F-025 | Settings sub-navigation | FR-017 |
| F-026 | Contextual empty states | FR-018 |
| F-027 | Adzuna affiliate job search | FR-021 |
| F-028 | 7-step onboarding registration | FR-022 |
| F-029 | Password reset flow | FR-023 |
| F-030 | Employment type on salary estimates | FR-008 (updated) |
| F-031 | Industry sector expansion (19 sectors) | FR-008 (updated) |
| F-032 | Provider signup links | FR-006 (updated) |
| F-033 | Git commit hash display | Section 8.1 |
| F-034 | iPhone mobile safe area rendering | FR-025 |
| F-035 | AI model selection per provider | FR-026 |
| F-036 | Onboarding bulk selection (Select All) | FR-022 (updated) |
| F-037 | Admin CLI tools | Section 9.7 |
| F-038 | Hybrid JWT authentication | FR-027 |
| F-039 | LinkedIn/Google OAuth2 | FR-027 (OAuth subsection) |
| F-040 | Per-session sandbox with Alex Morgan | FR-020 (updated) |
| F-041 | Frontend AuthContext provider | FR-029 |
| F-042 | Sandbox UI components | FR-030 |
| F-043 | Guided CV bullet optimization | FR-031 |
| F-044 | RBAC audit & hardening (service control admin-only, sandbox tab visibility) | FR-032 |
| F-054 | CV Optimisation Pipeline | FR-033 |
| F-055 | Training Provider Foundation | FR-034 |
| F-056 | AI Ethics & Fairness | FR-035 |
| F-057 | Growth Readiness (OG tags, PWA, analytics) | FR-036 |
| F-058 | Cross-user Data Isolation | FR-037 |
| F-059 | API Contract Tests | FR-038 |
| F-060 | Security hardening verification | FR-039 |

### 10.5 Traceability Matrix (BR -> FR -> Test)

| Business Req | Functional Req | Test Files | Coverage |
|-------------|---------------|-----------|----------|
| BR-001 (Multi-source search) | FR-001, FR-007 | test_search_*.py, test_career_search*.py, test_grey_box_fixes.py | 22 files |
| BR-002 (CV analysis) | FR-002 | test_cv_analysis.py, test_cv_*.py | 10 files |
| BR-003 (Fit scoring) | FR-003 | test_fit_score_*.py, test_match_scoring.py, test_unified_scoring.py | 8 files |
| BR-004 (Document gen) | FR-004 | test_document_*.py, test_multi_generate.py, test_ats_scoring.py | 10 files |
| BR-005 (App tracking) | FR-005 | test_saved_jobs.py, test_apply_endpoint.py | 3 files |
| BR-006 (AI failover) | FR-006 | test_tiered_routing.py, test_priority_routing.py, test_circuit_breaker.py | 6 files |
| BR-007 (Salary) | FR-008 | test_salary_*.py, test_company_salary.py | 4 files |
| BR-010 (Multi-user) | 3.1, 3.6 | test_auth.py, test_*isolation*.py, test_registration.py | 9 files |
| BR-011 (Rate limiting) | 3.4 | test_rate_limiting.py | 1 file |
| BR-012 (SSRF) | 3.3 | test_url_validation.py, test_path_traversal.py | 2 files |
| BR-013 (Grafana deployment) | FR-009, FR-010 | test_monitoring_endpoints.py, MonitoringPanel.test.jsx, TC-GRAF-001 to TC-GRAF-008 | 2 files + 8 deployment tests |
| BR-016 (Dedup) | FR-001 | test_search_comprehensive.py | 1 file |
| BR-019 (Ollama deployment) | FR-011 | TC-OLL-001 to TC-OLL-009 | 9 deployment tests |
| BR-020 (Auto-save) | 8.2 | Frontend tests (auto-save debounce) | 2 files |
| ~~BR-021~~ (→ BR-013) | FR-009, FR-010 | TC-GRAF-003, TC-GRAF-004, TC-MON-001 | 3 deployment tests |
| ~~BR-022~~ (→ BR-013) | FR-010 | TC-GRAF-002 | 1 deployment test |
| BR-023 (Ollama model loading) | FR-011 | TC-OLL-002, TC-OLL-004 | 2 deployment tests |
| BR-024 (Automated deployment) | FR-010, FR-011 | TC-INT-001 | 1 integration test |
| BR-025 (Ollama universal access) | FR-011 | TC-OLL-010 (parameterised: existing, new, all roles) | 1 parameterised test |
| BR-032 (Data isolation) | 3.6 | test_data_isolation_new_tables.py, test_user_data_isolation_ac.py | 2 files |
| BR-033 (Session security) | 3.1 | test_session_security.py | 1 file |
| BR-034 (Per-user keys) | 3.2 | test_user_key_resolver.py, test_api_key_isolation.py | 2 files |
| BR-035 (RBAC enforcement) | 3.1 | test_rbac_enforcement.py (26 tests) | 1 file |
| BR-036 (Key encryption) | 3.2 | test_key_encryption_at_rest.py (6 tests) | 1 file |
| BR-037 (Token budgets) | 3.1, FR-009 | test_token_budget.py (9 tests) | 1 file |
| ~~BR-038~~ (→ BR-013) | FR-009 | test_auth_event_logging.py (9 tests), test_monitoring.py | 2 files |
| BR-039 (Simplified CV Analysis) | FR-012 | App.test.jsx (CV Analysis Provider Indicator tests) | 6 tests |
| BR-040 (Onboarding stepper) | FR-013 | ProgressStepper.test.jsx | 6 tests |
| BR-041 (CV-to-Search data flow) | FR-014 | App.test.jsx (P3: CV-to-Search Data Flow) | 3 tests |
| BR-042 (Collapsible job sources) | FR-015 | App.test.jsx (P4: Job Sources Collapsible Section) | 3 tests |
| BR-043 (Advanced mode toggle) | FR-016 | App.test.jsx (P5: Advanced Mode Toggle) | 3 tests |
| BR-044 (Settings sub-navigation) | FR-017 | App.test.jsx (config-section navigation) | 2 tests |
| BR-045 (Contextual empty states) | FR-018 | App.test.jsx (P7: Empty States) | 3 tests |
| BR-046 (Sandbox abuse prevention) | FR-020 | test_sandbox_abuse.py (30 tests) | 1 file |
| BR-047 (Adzuna affiliate) | FR-021 | test_adzuna.py (25 tests) | 1 file |
| BR-048 (7-step onboarding) | FR-022 | ProgressStepper.test.jsx, LoginScreen.test.jsx | 2 files |
| BR-049 (Password reset) | FR-023 | test_password_reset.py (22 tests) | 1 file |
| BR-050 (Employment type) | FR-008 | test_salary_*.py | 1 file |
| BR-051 (Performance) | FR-024 | Locust load test suite | 1 file |
| BR-052 (Industry sectors) | FR-008 | test_salary_benchmarks.py | 1 file |
| BR-053 (Git commit hash) | Section 8.1 | App.test.jsx | 1 file |
| BR-054 (iPhone rendering) | FR-025 | Visual testing (manual) | N/A |
| BR-055 (AI model selection) | FR-026 | ConfigTab model dropdown tests | 1 file |
| BR-056 (Onboarding bulk select) | FR-022 | LoginScreen.test.jsx, App.test.jsx | 2 files |
| BR-057 (Display name removal) | FR-022 | LoginScreen.test.jsx, App.test.jsx | 2 files |
| BR-058 (Service control admin-only) | 3.1 | test_rbac_enforcement.py (service control tests), test_sandbox_abuse.py, AdminTab tests | 3 files |
| BR-059 (Admin CLI tools) | Section 9.7 | Manual testing | N/A |
| BR-060 (CSS variables refactor) | Section 8.2 | Visual testing (manual) | N/A |
| BR-061 (Compact ProgressStepper) | FR-013 | ProgressStepper.test.jsx | 1 file |
| BR-062 (Hybrid JWT auth) | FR-027 | test_jwt_utils.py (32 tests) | 1 file |
| BR-063 (OAuth2 social login) | FR-027 | security/test_oauth.py (22 tests) | 1 file |
| BR-064 (Per-session sandbox) | FR-020 | test_sandbox.py, SandboxBanner.test.jsx | 2 files |
| BR-065 (Account migration) | FR-027 | test_account_migration.py (6 tests) | 1 file |
| BR-066 (Frontend AuthContext) | FR-029 | App.test.jsx (auth integration) | 1 file |
| BR-067 (Sandbox timer) | FR-030 | SandboxBanner.test.jsx (9 tests) | 1 file |
| BR-068 (Backward compat) | FR-027, 3.1 | test_auth.py (existing 383 tests) | 1 file |
| BR-069 (CV bullet optimization) | FR-031 | test_cv_optimization.py (7 tests) | 1 file |
| BR-070 (RBAC hardening) | FR-032, 3.1 | test_rbac_enforcement.py (2 service control tests), TopNav.test.jsx (18 tests) | 2 files |
| BR-071 (CV Optimisation Pipeline) | FR-033 | test_cv_optimization.py | 1 file |
| BR-072 (Training Providers) | FR-034 | Skills & Training tab tests | 1 file |
| BR-073 (AI Ethics & Fairness) | FR-035 | Fairness guardrail prompt tests, confidence score tests | 2 files |
| BR-074 (Growth Readiness) | FR-036 | Meta tag tests, PWA manifest tests | 2 files |
| BR-075 (Cross-user Data Isolation) | FR-037 | test_user_isolation_audit.py, userStorage.test.js | 2 files |
| BR-076 (API Contract Tests) | FR-038 | test_api_contracts.py (14 tests) | 1 file |
| BR-077 (Security Verification) | FR-039 | test_shell_ratchet.py, security header tests | 2 files |

### 10.6 CI/CD Pipeline Test Gates

| Gate | Type | Blocking? |
|------|------|-----------|
| Backend tests (Python 3.11 + 3.12) | pytest -n auto --dist worksteal | **Hard gate** — must pass |
| Frontend tests (Node 20 + 22) | vitest run | **Hard gate** — must pass |
| Frontend build | vite build | **Hard gate** — must succeed |
| Schema drift check | alembic upgrade head + alembic check (PostgreSQL 16 service) | **Hard gate** — must pass |
| Flake8 (critical errors) | E9, F63, F7, F82 | **Hard gate** |
| Flake8 (style warnings) | max-line=150, max-complexity=25 | Soft gate |
| ESLint | --max-warnings=0 | **Hard gate** |
| pip-audit | --strict | Soft gate (warnings) |
| npm audit | --omit dev | Soft gate (warnings) |
| Snyk SCA | severity-threshold=high | Soft gate (warnings) |
| Snyk SAST | severity-threshold=high | Soft gate (warnings) |
| Aikido Security | fail on CRITICAL | Soft gate |
| Secret Detection | Hardcoded keys, .env files | Soft gate |
| SSRF Detection | requests.* without validation | Soft gate |
| Integration Smoke | 5 endpoints hit | Soft gate |

---

## 11. Deployment & Environment Requirements

### 11.1 Environments

| Environment | Purpose | URL | Config |
|-------------|---------|-----|--------|
| **Local Dev** | Development and testing | localhost:3000 (FE) / localhost:8000 (BE) | .env file, SQLite local |
| **CI** | Automated testing | GitHub Actions runners | In-memory SQLite (unit tests), PostgreSQL 16 service container (schema drift checks), mocked AI |
| **Production** | Live deployment | https://cviper.uk | Azure Container Apps, PostgreSQL, Bicep IaC |

### 11.2 Docker Configuration

**Backend Dockerfile**:
- Base: `python:3.12-slim` (multi-stage: builder + runtime)
- Non-root user: `appuser`
- Data dir: `/app/data` (Docker volume)
- Health check: `curl /api/health` every 30s
- CMD: `uvicorn main:app --host 0.0.0.0 --port ${PORT:-8000} --workers ${WORKERS}`

**Frontend Dockerfile**:
- Build: `node:20-alpine` → `npm ci && npm run build`
- Serve: `nginx:alpine` with SPA routing config
- Build arg: `VITE_API_BASE=/api` (proxied via nginx)

**Grafana Container** (infrastructure):
- Image: `grafana/grafana-oss:latest`
- Port: 3001 (host) → 3000 (container)
- Environment: `GF_SECURITY_ADMIN_PASSWORD` from env var
- Plugins: `yesoreyeram-infinity-datasource` (auto-installed via `GF_INSTALL_PLUGINS` env var or entrypoint script)
- Provisioning: Datasource and dashboard created via Grafana API on first startup
- Health check: `curl /api/health` every 30s
- Volume: `grafana-data:/var/lib/grafana` for persistent dashboards and settings
- Depends on: Backend (datasource points to backend API)

**Ollama Container** (AI runtime):
- Image: `ollama/ollama:latest`
- Port: 11434
- Environment: `OLLAMA_MODEL` (default: `llama3.2`), `OLLAMA_HOST=0.0.0.0`
- Model Loading: Entrypoint script runs `ollama pull ${OLLAMA_MODEL}` on first start
- Health check: `curl /api/tags` every 30s
- Volume: `ollama-models:/root/.ollama` for persistent model storage (avoids re-download)
- Resources: CPU by default; GPU passthrough via `deploy.resources.reservations.devices` in Compose
- Depends on: Nothing (standalone inference server)

**Docker Compose Service Definitions**: See `docker-compose.yml` in the repository root. Key services: `backend` (port 8000), `frontend` (port 3000), `grafana` (port 3001), `ollama` (port 11434). Grafana and Ollama volumes persist dashboard config and model weights respectively.

### 11.3 Azure Container Apps Deployment (Bicep IaC)

**Resource Group**: `cviper-rg` (UK South)

| Service | Azure Resource | Tier/SKU | Purpose |
|---------|---------------|----------|---------|
| cviper-backend | Container App | 0.5 vCPU / 1 GiB, scale 1-3 | FastAPI application server |
| cviper-frontend | Container App | 0.25 vCPU / 0.5 GiB, scale 1-2 | React SPA (nginx) |
| cviper-ollama | Container App | 2 vCPU / 4 GiB, scale 0-1 (demo-only) | Local LLM inference engine |
| cviper-grafana | Container App | 0.25 vCPU / 0.5 GiB, scale 0-1 | Monitoring dashboards |
| cviper-prometheus | Container App | 0.25 vCPU / 0.5 GiB, scale 0-1 | Metrics collection |
| cviper-loki | Container App | 0.25 vCPU / 0.5 GiB, scale 0-1 | Log aggregation |
| cviper-pg-* | PostgreSQL Flexible Server | Burstable B1ms, 32 GB | Persistent relational database |
| cviperacr | Container Registry | Basic | Docker image storage |

**Infrastructure-as-Code**: `azure/container-apps.bicep` — single Bicep template defining all resources, deployed via `az deployment group create`.

**Custom Domain**: `cviper.uk` with Azure-managed SSL certificate (auto-renewing), bound to frontend Container App.

**Database Migrations**: Alembic (`backend/alembic/`) manages PostgreSQL schema. Migrations use `batch_alter_table()` context manager for SQLite compatibility (copy-and-move strategy for ALTER TABLE operations). Schema drift is validated in CI via a PostgreSQL 16 service container running `alembic upgrade head` followed by `alembic check`. Production migrations run from GitHub Actions with temporary firewall rules for runner IP access.

### 11.4 Required Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| AI_PROVIDER | Yes | pluribus | Default AI provider |
| AUTH_ENABLED | No | false | Enable multi-user auth |
| DATABASE_URL | Yes (prod) | sqlite:///data/job_match_pro.db (local), postgresql://... (prod) | Database connection string |
| CORS_ORIGINS | No | localhost origins | Allowed CORS origins |
| WORKERS | No | 1 | Uvicorn worker count |
| GRAFANA_URL | Yes | http://localhost:3001 | Grafana base URL (auto-set in Docker Compose) |
| GRAFANA_ADMIN_PASSWORD | Yes | changeme | Grafana admin login password |
| OPENAI_API_KEY | No | (empty) | OpenAI API key |
| ANTHROPIC_API_KEY | No | (empty) | Anthropic API key |
| GOOGLE_API_KEY | No | (empty) | Google Gemini API key |
| OLLAMA_ENABLED | Yes | true | Enable Ollama local inference |
| OLLAMA_BASE_URL | Yes | http://ollama:11434 | Ollama API URL (auto-set in Docker Compose) |
| OLLAMA_MODEL | No | llama3.2 | Default Ollama model to load |
| GF_INSTALL_PLUGINS | No | yesoreyeram-infinity-datasource | Grafana plugins to install on startup |
| MASTER_KEY | Yes (prod) | (none — startup fails without it in production) | Fernet master key for encrypting user API keys and prompt logs |
| SECURE_COOKIES | No | false | Enable `Secure` flag on session cookies (HTTPS) |
| ADZUNA_APP_ID | No | (empty) | Adzuna API application ID |
| ADZUNA_APP_KEY | No | (empty) | Adzuna API key |
| SANDBOX_FP_DAILY_LIMIT | No | 50 | Daily request limit per browser fingerprint (sandbox) |
| SANDBOX_IP_DAILY_LIMIT | No | 100 | Daily request limit per IP address (sandbox) |
| JWT_SECRET_KEY | Yes (prod) | dev-secret-change-in-production | Secret key for signing JWT access tokens |
| JWT_ALGORITHM | No | HS256 | JWT signing algorithm |
| JWT_ACCESS_TOKEN_EXPIRE_MINUTES | No | 15 | Access token expiry in minutes |
| JWT_REFRESH_TOKEN_EXPIRE_DAYS | No | 7 | Refresh token expiry in days |
| LINKEDIN_CLIENT_ID | No | (empty) | LinkedIn OAuth2 application client ID |
| LINKEDIN_CLIENT_SECRET | No | (empty) | LinkedIn OAuth2 application client secret |
| LINKEDIN_REDIRECT_URI | No | http://localhost:8000/api/auth/linkedin/callback | LinkedIn OAuth2 callback URL |
| GOOGLE_CLIENT_ID | No | (empty) | Google OAuth2 application client ID |
| GOOGLE_CLIENT_SECRET | No | (empty) | Google OAuth2 application client secret |
| GOOGLE_REDIRECT_URI | No | http://localhost:8000/api/auth/google/callback | Google OAuth2 callback URL |

### 11.5 Release Process

```
1. Developer pushes to feature branch
2. CI runs automatically: tests, lint, security scans (ci.yml)
3. PR reviewed and merged to main
4. CI runs on main: all gates must pass
5. Developer manually triggers deploy (GitHub Actions → Deploy to Azure → Run workflow):
   - Selects environment (production), services (all/backend/frontend), skip migrations (true/false)
   - Preflight check verifies CI passed on target commit
   - Docker images built, tagged with commit SHA, pushed to Azure Container Registry
   - Alembic migrations run against PostgreSQL (temporary firewall rule for runner IP)
   - Container Apps updated with new image revision
6. Health check verifies all services are Running
7. Deployment summary posted to GitHub Actions UI
```

**Manual Gate Rationale**: Production deployments require explicit human approval via `workflow_dispatch` rather than auto-deploying on merge. This provides a deliberate checkpoint between "code is ready" (CI passes) and "code is live" (deploy triggered), preventing accidental deployments and enabling batched releases.

### 9.7 Maintenance & Support

#### Support Model

| Tier | Scope | Response |
|------|-------|---------|
| Self-service | Documentation, CLAUDE.md, error messages | Immediate |
| Admin | Database cleanup, user management, backup | Via admin panel |
| Developer | Bug fixes, feature requests | GitHub Issues |

#### SLAs

| Metric | Target |
|--------|--------|
| API uptime | 99.5% |
| Incident response (critical) | 4 hours |
| Bug fix (high severity) | 48 hours |
| Feature request triage | 1 week |

#### Operational Monitoring

| Component | Tool | Alert |
|-----------|------|-------|
| Backend health | /api/health (Container App liveness probe) | Container restart on failure |
| AI providers | Grafana dashboard | Visual — error rate > 15% |
| Rate limits | Monitoring panel Usage tab | Visual — usage > 90% |
| Database size | Admin panel | Manual check |
| Frontend errors | /api/frontend-errors | Logged for review |

#### Scheduled Maintenance

| Task | Frequency | Mechanism |
|------|-----------|-----------|
| Database cleanup (old records) | Configurable (MAINTENANCE_INTERVAL_HOURS) | Background task, AUTO_CLEANUP_DAYS retention |
| Dead company pruning | As needed | DEAD_COMPANIES set in seed_company_boards.py |
| Dependency updates | Monthly | pip-audit + npm audit in CI |
| Seed data refresh | As needed | python -m backend.seed_company_boards --update |
| Database backup | On demand | GET /api/admin/backup |

---

*End of Functional Specification Document*
