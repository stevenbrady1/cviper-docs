# CViper

**Make Every Application Count — Strike with Precision.**

CViper is a professional career management tool that helps you discover job
opportunities, analyse your CV, and generate tailored application materials —
all powered by AI.

---

## What it does

- **Multi-Site Job Search** — searches 7 job boards (Reed, Adzuna, Jooble, Remotive, Findwork, Freelancer via official APIs; Jobserve via a plain self-identifying scraper) plus 200+ direct employer career pages, via SSE streaming. LinkedIn and eFinancialCareers block automated access and are surfaced as manual-search links instead.
- **AI CV Analysis & Tailored Documents** — extracts your profile from PDF/DOCX, scores every job against it (two-way, seniority-adaptive), and generates customised CVs and cover letters with ATS scoring and fabrication detection.
- **Multi-Provider AI** — supports 8 providers: OpenAI, Anthropic Claude, Google Gemini, Grok, Mistral, OpenRouter, Pluribus (local), and Ollama — with priority routing, automatic failover, circuit breakers, and keyword fallback.
- **Application Tracking & Insights** — Kanban status tracking, salary benchmarking (incl. contractor day rates/IR35), skills-gap analysis, interview prep, job alerts, career insights.
- **Accounts & Safety** — JWT auth with RBAC, Free/Pro tiers with usage limits, demo sandbox, GDPR data retention/export/erasure, cross-border AI consent, AI fairness guardrails, structured audit logging.

The full feature-by-feature specification lives in
[`docs/01-PRODUCT-SPEC.md`](docs/01-PRODUCT-SPEC.md).

---

## 5-minute setup (Docker)

1. **Create `.env`** in the project root — at least one AI provider key (the
   keyword matcher works with none):

   ```env
   OPENAI_API_KEY=sk-your-key-here        # or ANTHROPIC_API_KEY /
   GOOGLE_API_KEY=your-google-key         # GOOGLE_API_KEY / MISTRAL_API_KEY /
   AI_PROVIDER=openai                     # OPENROUTER_API_KEY / OLLAMA_ENABLED=true
   ```

2. **Start it:**

   ```bash
   docker compose up -d --build
   ```

3. **Open** http://localhost:3000 (backend API on port 8000).

Stop with `docker compose down`; rebuild after changes with
`docker compose up -d --build`.

**Local development instead of Docker** (Python 3.10+, Node 18+):

```bash
cd backend && python -m venv venv && . venv/bin/activate   # Windows: venv\Scripts\activate
pip install -r requirements.txt && python main.py          # API on :8000
cd ../frontend && npm install && npm run dev               # UI on :3000
```

---

## Documentation

| What you need | Where |
|---|---|
| Feature behaviour, user stories, workflows, UI | [`docs/01-PRODUCT-SPEC.md`](docs/01-PRODUCT-SPEC.md) |
| API inventory (generated — every route) | [`docs/02-API.md`](docs/02-API.md); live Swagger at http://localhost:8000/docs |
| Data model (generated — every table) | [`docs/03-DATA-MODEL.md`](docs/03-DATA-MODEL.md) |
| Testing strategy, test plan, traceability | [`docs/04-QUALITY.md`](docs/04-QUALITY.md) |
| Two-product architecture (hosted + Light) | [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) |
| Business requirements / functional spec | [`docs/BRD-Business-Requirements-Document.md`](docs/BRD-Business-Requirements-Document.md) · [`docs/FSD-Functional-Specification-Document.md`](docs/FSD-Functional-Specification-Document.md) |
| Deploying (Azure Container Apps, Bicep) | [`docs/DEPLOYMENT_GUIDE.md`](docs/DEPLOYMENT_GUIDE.md) |
| Contributing, conventions, working rules | [`CONTRIBUTING.md`](CONTRIBUTING.md) · [`CLAUDE.md`](CLAUDE.md) · [`docs/conventions/`](docs/conventions/) |
| Developer onboarding & tooling | [`DEVELOPER_GUIDE.md`](DEVELOPER_GUIDE.md) |
| Change history | [`CHANGELOG.md`](CHANGELOG.md) |

**Running tests:**

```bash
cd backend && pytest                 # backend (parallel)
cd frontend && npm test              # frontend (Vitest)
cd frontend && npm run test:e2e      # Playwright
```

More in [`docs/04-QUALITY.md`](docs/04-QUALITY.md) §13 Quick Reference.

---

## Tech stack

React 18 + Vite · FastAPI (Python 3.11+) · PostgreSQL (prod) / SQLite (local)
· Azure Container Apps + Bicep IaC · Capacitor (iOS/Android) · Playwright +
pytest + Vitest.

---

## Legal

[Terms of Service](docs/TERMS_OF_SERVICE.md) ·
[Privacy Policy](docs/PRIVACY_POLICY.md)

## License

Personal project — all rights reserved. Not licensed for redistribution.
