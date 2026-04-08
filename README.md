# CViper — Public Documentation

> **This is a curated public documentation snapshot of [CViper](https://cviper.uk)**, an AI-powered CV analysis and job-matching application designed, built, and shipped end-to-end by **Steven Brady** as a full-stack portfolio case study.
>
> **Live demo:** [cviper.uk](https://cviper.uk) — click "Try Demo" on the landing page for a zero-signup walkthrough with seeded sample data.

---

## What this repository is

A curated subset of CViper's engineering documentation, published publicly so recruiters, engineers, and technical reviewers can read the key decisions and architecture without needing access to the private source repository.

### What's in here

| Document | What it covers |
|---|---|
| [`docs/BRD-Business-Requirements-Document.md`](docs/BRD-Business-Requirements-Document.md) | Business Requirements Document — user personas, functional requirements, business rules, data subject rights, risk register, and glossary. |
| [`docs/FSD-Functional-Specification-Document.md`](docs/FSD-Functional-Specification-Document.md) | Functional Specification — data model, API surface (~240 endpoints), authentication flow, AI provider routing, streaming job search, security controls. |
| [`docs/Testing-Strategy-and-Architecture.md`](docs/Testing-Strategy-and-Architecture.md) | Testing philosophy, CI pipeline design, test counts, coverage targets, prevention-test patterns. |
| [`docs/SLO.md`](docs/SLO.md) | Service-level objectives for the production deployment. |
| [`docs/STATS.md`](docs/STATS.md) | Auto-generated project statistics (routes, components, migrations, test counts). |
| [`docs/adr/`](docs/adr/) | Architecture Decision Records — the thinking behind significant technical choices. |
| [`showcase/`](showcase/) | Architecture diagrams (SVG) — system overview, AI routing, CI/CD pipeline. |
| [`CHANGELOG.md`](CHANGELOG.md) | Version history following [Keep a Changelog](https://keepachangelog.com). |

### Architecture Decision Records

- **[ADR-001: Dual-Database Strategy](docs/adr/001-dual-database-strategy.md)** — why CViper runs on SQLite locally and PostgreSQL in production, and how the schema stays compatible.
- **[ADR-002: AI Tier Routing](docs/adr/002-ai-tier-routing.md)** — the three-tier routing system (premium / standard / light) that assigns different tasks to different model classes.
- **[ADR-003: Sandbox Abuse Prevention](docs/adr/003-sandbox-abuse-prevention.md)** — the five-layer strategy that lets CViper run an open demo mode without getting hammered by abuse.
- **[ADR-004: State-Based Navigation](docs/adr/004-state-based-navigation.md)** — why CViper uses `useState('tab')` navigation instead of React Router.
- **[ADR-005: Fatal Startup Guard Rules](docs/adr/005-fatal-startup-guard-rules.md)** — the pattern that prevented a 4-day production outage from recurring.

---

## What's NOT in here (and why)

The full CViper source code lives in a **private GitHub repository**. This public docs repo excludes everything that isn't directly useful for external technical review:

- **Source code** — excluded because it contains hardcoded demo data, seed fixtures, and commit history that mentions internal planning.
- **Operational runbooks** (key rotation, incident response, database degradation, deployment failure) — excluded because these are operational playbooks useful to attackers, not interesting to reviewers.
- **Infrastructure dependency map** — excluded because it reveals Azure resource topology.
- **Internal audit reports and post-incident reviews** — excluded because they reference specific commit hashes and live infrastructure.
- **The `.env.example`, CI workflow files, and deploy scripts** — excluded as operational.
- **Draft legal documents** (Privacy Policy v2.0 draft, Terms of Service v2.0 draft, DPIA draft) — excluded until solicitor review is complete. The **live** Privacy Policy and Terms of Service are published at [cviper.uk/privacy](https://cviper.uk/privacy) and [cviper.uk/terms](https://cviper.uk/terms).
- **Strategic planning documents and roadmaps** — excluded as internal business planning.

This is a deliberate curation, not an oversight. The public surface area is kept tight so that the things that ARE here can be high-signal.

---

## How to request full codebase access

If you're a recruiter, hiring engineer, or technical reviewer who wants to see the actual code (commit history, PRs, CI pipelines, implementation details):

**Email [cviperuk@gmail.com](mailto:cviperuk@gmail.com)** with a short note about who you are and what you're looking for. Read-only access to the private repository is typically granted within 24 hours for time-limited review windows.

This is a cheaper filter than making everything public: people who genuinely want to review the code ask, and I can grant access on a per-request basis.

---

## How it was built

CViper is a full-stack single-person portfolio project. The engineering highlights:

- **Backend:** Python 3.12 + FastAPI, ~240 API endpoints across 21 route modules, 26 Alembic migrations, 193 backend test files
- **Frontend:** React 18 + Vite, 35 components, 15 custom hooks, 59 frontend test files
- **AI integration:** 9 providers (OpenAI, Anthropic, Google, Mistral, xAI, OpenRouter, GitHub Models, Pluribus, self-hosted Ollama) with priority-based routing and keyword-based fallback
- **Infrastructure:** Azure Container Apps + PostgreSQL Flexible Server + Cloudflare (DNS/DDoS/TLS) + Azure Key Vault, all deployed via Bicep IaC and GitHub Actions
- **Observability:** Prometheus metrics, Loki log aggregation, Grafana dashboards, structured JSON logging, OpenTelemetry-ready
- **Security:** bcrypt password hashing, Fernet encryption at rest, CSP + HSTS headers, CSRF middleware, rate limiting, OWASP ZAP in CI, pip-audit + npm audit + SSRF scanning
- **CI/CD:** GitHub Actions multi-job pipeline, matrix backend tests with parallelisation, frontend tests + lint + security scans, Docker smoke tests, preflight Azure resource checks

See the [FSD](docs/FSD-Functional-Specification-Document.md) and [Testing Strategy](docs/Testing-Strategy-and-Architecture.md) for the full picture.

---

## Versioning

Documents in this repo are versioned with the app. The current snapshot reflects **CViper v0.4.1** (2026-04-08). Older versions live in the private repo's `docs/Archive/` folder.

For the live app version, see [cviper.uk/?tab=about](https://cviper.uk/?tab=about).

---

## Contact

- **Live application:** [cviper.uk](https://cviper.uk)
- **Email:** [cviperuk@gmail.com](mailto:cviperuk@gmail.com)
- **About:** The live app has a "Behind CViper" tab with interactive architecture diagrams, the same ADR decisions in clickable cards, and screenshots of every major feature.

---

*This documentation snapshot is maintained by Steven Brady. Published under the same terms as the CViper service itself — see [cviper.uk/terms](https://cviper.uk/terms). Not a license to reuse CViper's code or branding.*
