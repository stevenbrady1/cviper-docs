# CViper

**Make Every Application Count — Strike with Precision.**

CViper is a professional career management tool that helps you discover job opportunities, analyse your CV, and generate tailored application materials — all powered by AI.

---

## What It Does

- **Multi-Site Job Search** — searches LinkedIn, CWJobs, Reed, Indeed UK, eFinancialCareers, and Totaljobs via SSE streaming
- **AI CV Analysis** — extracts skills, experience, and profile from PDF/DOCX files with multi-provider comparison
- **Multi-Provider AI** — supports 9 providers: OpenAI, Anthropic Claude, Google Gemini, Grok, GitHub Models, Mistral, OpenRouter, Pluribus (local), and Ollama. Run providers in parallel and compare results side-by-side with Prompt Lab.
- **Priority-Based AI Routing** — configurable provider priority chain with automatic failover, circuit breakers, and keyword fallback
- **Async Task Queue** — background AI task processing with real-time progress tracking and status polling
- **Tailored Documents** — generates customised CVs and cover letters for each role (DOCX & PDF) with ATS scoring and fabrication detection
- **Two-Way Scoring** — evaluates both candidate-to-job fit and job-to-candidate fit with sub-scores, using seniority-adaptive weighting that adjusts scoring dimensions by role level
- **Application Tracking** — track status from Saved → Applied → Interviewing → Offer, with Kanban board
- **Salary Benchmarking** — AI-generated salary estimates grounded against curated London IT/Financial benchmarks, with contractor day-rate and IR35 support
- **Skills Gap Analysis** — dashboard with weekly stats, CSV/XLSX export, and learning resource recommendations
- **Interview Prep** — AI-generated interview questions, preparation notes, and follow-up email drafting
- **Company Career Pages** — discover and manage company career pages (Greenhouse, Lever, iCIMS, Taleo) with concurrent discovery pipeline and health monitoring
- **Guided Onboarding** — step-by-step tour for new users (including demo mode) with nudge banners and progress tracking
- **Demo Mode** — restricted sandbox with sample data, guided tour, and a step-by-step API key guide for non-technical users
- **RBAC & Auth** — JWT-based authentication with admin, standard, and sandbox user roles, with email verification via deep-link after registration
- **Banking-Grade Security** — security observability dashboards, structured audit logging, and anomaly detection for sensitive operations
- **Data Retention & GDPR** — automated data retention schedules with configurable policies and email token cleanup for GDPR compliance
- **Feedback System** — in-app feedback collection with image attachments and admin triage panel
- **FAQ & Help** — searchable FAQ across 10 categories including AI explainers (tokens, accuracy, API keys)
- **Career Insights** — rejection pattern analysis, application strategy, and skills gap roadmap
- **Job Alerts** — automatic new-job notifications from saved search profiles with deduplication, daily/weekly frequency, and in-app notification bell
- **Usage Tracking & Tiers** — Free/Pro tier system with per-operation daily limits (AI calls, CV scores, salary estimates, document generation), upgrade prompts, and usage dashboard
- **AI Fairness** — bias audit framework with 31 synthetic CVs across 5 demographic dimensions, FAIRNESS_GUARDRAIL in all scoring prompts, automated regression tests
- **CV Optimisation** — guided bullet improvement with AI suggestions
- **AI Disclaimer** — transparent attribution: all AI results come from the user's chosen model, with accuracy warnings in-app and in the privacy policy
- **PWA Support** — service worker with stale-while-revalidate caching, installable on mobile
- **Infrastructure Resilience** — pre-deploy config validation, server path redaction, 17 lessons-learned with automated prevention guards

---

## Getting Started

### Prerequisites

Choose **one** of the two setup paths below:

| Path | You need |
|------|----------|
| **Docker** (recommended) | [Docker Desktop](https://www.docker.com/products/docker-desktop/) |
| **Local development** | Python 3.10+, Node.js 18+ |

Both paths require at least one AI provider API key (see [AI Provider Setup](#ai-provider-setup) below). The keyword-based matcher works without any key.

---

### Option A: Docker (Recommended)

This is the fastest way to get running. Docker handles all dependencies for you.

**Step 1 — Create your `.env` file**

Copy the example below into a file called `.env` in the project root:

```env
# At least one AI provider key is needed (all are optional)
OPENAI_API_KEY=sk-your-key-here
ANTHROPIC_API_KEY=sk-ant-your-key-here
GOOGLE_API_KEY=your-google-ai-studio-key
GITHUB_TOKEN=ghp-your-token-here
OPENROUTER_API_KEY=your-openrouter-key
MISTRAL_API_KEY=your-mistral-key
OLLAMA_ENABLED=true

# Default provider for single-provider operations
AI_PROVIDER=openai
```

**Step 2 — Start the containers**

```bash
docker compose up -d --build
```

**Step 3 — Open the app**

Go to **http://localhost:3000** in your browser. The backend API runs on port 8000.

**Stopping the app:**

```bash
docker compose down
```

**Rebuilding after code changes:**

```bash
docker compose up -d --build
```

---

### Option B: Local Development

#### Step 1 — Clone and create `.env`

```bash
git clone <repo-url> cviper
cd cviper
```

Create a `.env` file in the project root with your API keys (same format as Option A above).

#### Step 2 — Set up the backend

```bash
cd backend
python -m venv venv
```

Activate the virtual environment:

```bash
# Windows
venv\Scripts\activate

# macOS / Linux
source venv/bin/activate
```

Install dependencies:

```bash
pip install -r requirements.txt
```

#### Step 3 — Set up the frontend

```bash
cd ../frontend
npm install
```

#### Step 4 — Start the backend

Open a terminal in the `backend/` folder:

```bash
venv\Scripts\activate          # Windows
source venv/bin/activate       # macOS / Linux
python main.py
```

The backend starts on **http://localhost:8000**. Wait until you see `Uvicorn running on http://0.0.0.0:8000` before starting the frontend.

#### Step 5 — Start the frontend

Open a **second** terminal in the `frontend/` folder:

```bash
npm run dev
```

The frontend starts on **http://localhost:3000**. Open this URL in your browser.

#### One-Command Start (Windows)

If you've already run setup once, you can use:

```
start-app.bat
```

This launches both servers, waits for health checks, and opens the app automatically.

#### Automated Setup (Windows)

If you haven't installed dependencies yet:

```
setup.bat
```

This creates the Python virtual environment and installs all backend and frontend dependencies in one go.

---

## AI Provider Setup

CViper supports multiple AI providers. You only need **one** to get started.

| Provider | Env Variable | Get a Key |
|----------|-------------|-----------|
| OpenAI | `OPENAI_API_KEY` | platform.openai.com |
| Anthropic Claude | `ANTHROPIC_API_KEY` | console.anthropic.com |
| Google Gemini | `GOOGLE_API_KEY` | aistudio.google.dev |
| GitHub Models | `GITHUB_TOKEN` | github.com/settings/tokens |
| Mistral | `MISTRAL_API_KEY` | console.mistral.ai |
| OpenRouter | `OPENROUTER_API_KEY` | openrouter.ai |
| Ollama (local) | `OLLAMA_ENABLED=true` | ollama.com (free, runs locally) |

Add keys to your `.env` file and restart the backend. The Settings tab in the UI shows which providers are active.

---

## Using CViper

### 1. Upload Your CV

Go to the **CV Analysis** tab and drag-and-drop your CV (PDF or DOCX). CViper extracts your skills, experience, and profile automatically.

### 2. Search for Jobs

Go to the **Job Search** tab. Select your AI providers, enter a location and optional minimum salary, then click **Search for Jobs**. Results are scored against your CV.

### 3. Apply for Jobs

Select matching jobs from the results and click **Apply for Selected Jobs** (up to 5 concurrent). A tailored CV and cover letter are generated for each role.

### 4. Track Applications

Go to **Applications** to update statuses, add notes, and manage your pipeline in table or Kanban view.

### 5. Compare AI Models (Prompt Lab)

Go to **Prompt Lab → Compare** to send a single prompt to multiple AI providers simultaneously and compare their responses side-by-side. Adjust temperature and max tokens per run.

---

## Auto-Start on Windows Boot (Docker)

To have CViper launch automatically when you log in:

```
install-startup.bat
```

This places a shortcut in your Windows Startup folder. The script waits for Docker Desktop to be ready, then starts the containers.

To remove auto-start:
```
install-startup.bat --uninstall
```

Logs are written to `docker-start.log` in the project root.

---

## Project Structure

```
cviper/
├── backend/                    # FastAPI backend (Python)
│   ├── main.py                 # App setup, middleware, startup
│   ├── database.py             # SQLAlchemy models & engine
│   ├── repositories.py         # CRUD operations
│   ├── ai_service.py           # AI facade → ai/ package
│   ├── ai/                     # AI subpackage
│   │   ├── providers.py        #   Provider registry (8 cloud + 2 local providers + 2 sandbox)
│   │   ├── router.py           #   Priority-based provider routing
│   │   ├── gateway.py          #   Universal call dispatcher
│   │   └── services/           #   Domain services (CV, matching, docs, scoring)
│   ├── routes/                 # 20 route modules (search, jobs, auth, admin, etc.)
│   ├── migrations/             # Alembic migrations (PostgreSQL)
│   ├── requirements.txt
│   └── Dockerfile
├── frontend/                   # React 18 + Vite frontend
│   ├── src/
│   │   ├── App.jsx             # Main application component
│   │   ├── theme.css           # Design system stylesheet
│   │   ├── components/         # Extracted UI components (see docs/STATS.md)
│   │   ├── tabs/               # Tab components (CV, Search, Apps, Companies, etc.)
│   │   ├── hooks/              # Custom hooks (useApi, useAIProviders, useToast)
│   │   └── context/            # React context providers (AppContext)
│   ├── nginx.conf              # Production nginx config
│   └── Dockerfile
├── azure/                      # Infrastructure as Code
│   ├── container-apps.bicep    # Azure Container Apps + networking
│   └── deploy.sh               # Deployment script with secret management
├── docs/                       # Project documentation
│   ├── runbooks/               # 9 operational runbooks
│   └── adr/                    # 5 architecture decision records
├── docker-compose.yml          # Local container orchestration
├── .github/workflows/          # CI (automatic) + Deploy (manual gate)
├── .env                        # API keys (not committed)
├── start-app.bat               # Local dev launcher (Windows)
├── setup.bat                   # Dependency installer (Windows)
├── setup.sh                    # Dependency installer (Linux/Mac)
└── Makefile                    # Cross-platform dev commands
```

---

## Running Tests

**Backend:**
```bash
cd backend
venv\Scripts\activate
pytest
```

**Frontend (unit/component):**
```bash
cd frontend
npm test
```

**Frontend (E2E — Playwright):**
```bash
cd frontend
npm run test:e2e           # headless Chromium
npm run test:e2e:headed    # with visible browser
```

E2E tests use route mocking — no backend needed. 50 specs providing 100% E2E journey coverage across all tabs, modals, and user workflows.

---

## API Documentation

When the backend is running, interactive docs are available at:

- **Swagger UI:** http://localhost:8000/docs
- **ReDoc:** http://localhost:8000/redoc

---

## Troubleshooting

### Docker containers won't start

```bash
docker compose logs backend     # Check backend errors
docker compose logs frontend    # Check frontend errors
docker compose down && docker compose up -d --build   # Clean rebuild
```

### Backend won't start (local)

```bash
python --version                             # Needs 3.10+
pip install --upgrade -r requirements.txt    # Reinstall deps
python -m py_compile main.py                 # Check syntax errors
```

### Frontend shows blank page

```bash
node --version                               # Needs 18+
rm -rf node_modules && npm install           # Clean reinstall
```

### "Unable to connect to backend server" in the UI

- Make sure the backend is running on port 8000
- Check the backend terminal for errors
- Click the **Retry** button in the UI after starting the backend

### AI providers not showing in Settings

- Check `.env` has at least one valid API key
- Restart the backend after editing `.env`
- Look for `[OK] ... client initialized` in the backend startup log

### Port already in use

```bash
# Windows
netstat -ano | findstr :8000
taskkill /PID <PID> /F

# macOS / Linux
lsof -i :8000
kill -9 <PID>
```

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React 18, Vite, CSS (custom design system) |
| Backend | Python, FastAPI, Uvicorn |
| AI | OpenAI, Anthropic, Google Gemini, GitHub Models, Mistral, OpenRouter, Ollama |
| CV Parsing | PyPDF2, python-docx |
| Database | PostgreSQL (production via Azure Flexible Server), SQLite (local dev) |
| ORM | SQLAlchemy 2.0 + Alembic migrations |
| Documents | python-docx (DOCX), custom PDF writer |
| Auth | JWT (access + refresh tokens), bcrypt, RBAC (admin/standard/sandbox) |
| CI | GitHub Actions (tests, lint, security scans, Docker smoke, schema drift, docs drift, dead exports, unused deps) |
| Deploy | Azure Container Apps, Bicep IaC, Cloudflare DNS/TLS, manual deploy gate |
| Monitoring | Prometheus, Grafana, Loki (structured logging) |

---

## Legal

- **[Privacy Policy](docs/PRIVACY_POLICY.md)** — data collection, lawful bases, GDPR rights, AI disclaimer
- **[Terms of Service](docs/TERMS_OF_SERVICE.md)** — acceptable use, third-party services, limitation of liability

Both documents are also reachable in-app via **Settings → Privacy & Data** or directly at `https://cviper.uk/?tab=privacy` and `https://cviper.uk/?tab=terms`.

## License

Proprietary — CViper
