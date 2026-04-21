# Deployment Guide

## Architecture Overview

```
User → Cloudflare (DNS/TLS) → Azure Container Apps → FastAPI Backend
                                                    → React Frontend (static)
                                                    → PostgreSQL (Azure Flexible Server)
```

- **Domain**: cviper.uk
- **Infrastructure**: Azure Container Apps with Bicep IaC
- **Database**: Azure Flexible Server for PostgreSQL
- **CDN/Security**: Cloudflare
- **CI/CD**: GitHub Actions (automatic CI, manual deploy gate)

## Prerequisites

- Azure CLI authenticated (`az login`)
- GitHub CLI authenticated (`gh auth login`)
- Access to the Azure subscription and resource group
- Docker installed locally (for smoke tests)

## Deployment Pipeline

### 1. CI Pipeline (automatic on push to main)

The CI pipeline runs automatically and must pass before deployment:

1. **Detect Changes** — test impact analysis (skip irrelevant suites)
2. **Backend Tests** — pytest with coverage summary
3. **Frontend Tests** — Vitest with coverage via `@vitest/coverage-v8`
4. **E2E Tests** — Playwright (Chromium) with route mocking, soft gate with HTML report artifact
5. **Lint & Dependency Checks** — flake8, ESLint, `npx depcheck` (advisory)
6. **Docs Drift & Code Hygiene** — stats freshness (hard gate), dead exports + unused deps (advisory)
7. **Security Scans (SCA + SAST)** — pip-audit, npm audit, secret scanning, SSRF scanning
8. **Security DAST** — OWASP ZAP frontend baseline + API scan (soft gate, runs in parallel)
9. **Docker Smoke Test** — build and boot the production container
10. **Build Verification** — frontend production build

### 2. Deploy (manual trigger)

Deployment requires manual approval via GitHub Actions:

```bash
# Via GitHub CLI
gh workflow run deploy.yml -f environment=production

# Or via GitHub UI: Actions → Deploy → Run workflow
```

The deploy workflow:
1. **Preflight check** — verifies CI passed on the target commit
2. **Build** — Docker image with production configuration
3. **Push** — to Azure Container Registry
4. **Deploy** — update Azure Container Apps revision
5. **Health check** — verify the new revision is healthy

### 3. Concurrency

- Deploy workflow uses `concurrency: deploy-${{ environment }}` — only one deploy per environment at a time
- CI uses `concurrency: ci-${{ github.ref }}` with `cancel-in-progress: true`

## Environment Variables

### Required (production)

All required env vars must be present in three places simultaneously (Fatal Startup Guard Rules):

| Variable | Where Set | Purpose |
|---|---|---|
| `DATABASE_URL` | Bicep + deploy.sh | PostgreSQL connection string |
| `AUTH_ENABLED` | Bicep | Enable session authentication |
| `JWT_SECRET_KEY` | deploy.sh (secret) | JWT access/refresh token signing key |
| `MASTER_KEY` | deploy.sh (secret) | Fernet master key for API key encryption |
| `PG_ADMIN_PASSWORD` | deploy.sh (secret) | PostgreSQL admin password |
| `GRAFANA_ADMIN_PASSWORD` | deploy.sh (secret) | Grafana admin password |
| `REGISTRATION_ENABLED` | Bicep | Enable/disable user registration |

### Sandbox Environment Variables

| Variable | Where Set | Purpose |
|---|---|---|
| `SANDBOX_GOOGLE_API_KEY` | Bicep (secret) | Dedicated Gemini key for sandbox users |
| `SANDBOX_OPENROUTER_API_KEY` | Bicep (secret) | Dedicated OpenRouter key for sandbox users |

### Cloud Mode & Blob Storage (issue #208)

| Variable | Where Set | Purpose |
|---|---|---|
| `CLOUD_MODE` | Bicep (`'true'`) | Switches frontend off local-FS affordances (Output Folder field, folder browser) and tells `helpers/blob_storage` to use the configured blob endpoint instead of `/tmp` |
| `CVIPER_BLOB_ACCOUNT_URL` | Bicep (sourced from `storageAccount.properties.primaryEndpoints.blob`) | Blob storage account URL — used by `DefaultAzureCredential` + `BlobServiceClient` |
| `CVIPER_BLOB_CONTAINER` | Bicep (`'cviper-documents'`) | Container name for generated CVs / cover letters |
| `CLOUD_MODE_OUTPUT_DIR` | optional (default `/tmp/cviper-output`) | Local fallback directory for the bridge period when blob upload fails |

Auth uses the backend Container App's **system-assigned managed identity** with the `Storage Blob Data Contributor` role on the storage account — no connection strings, no account keys, no secrets to rotate. The lifecycle policy on the storage account auto-deletes blobs at 30 days, so ongoing maintenance is zero.

### Job Source API Keys

| Variable | Where Set | Purpose |
|---|---|---|
| `REED_API_KEY` | Bicep (secret) | Reed.co.uk official API |
| `JOOBLE_API_KEY` | Bicep (secret) | Jooble global aggregator (LinkedIn alternative) |
| `FREELANCER_API_KEY` | Bicep (secret) | Freelancer.com REST API for freelance projects (always Contract type) |
| `ADZUNA_APP_ID` / `ADZUNA_APP_KEY` | Bicep (secret) | Adzuna affiliate API |

### OAuth Redirect URIs (production)

| Variable | Value | Purpose |
|---|---|---|
| `GOOGLE_REDIRECT_URI` | `https://cviper.uk/api/auth/google/callback` | Pinned to public host — startup guard refuses to boot if it contains `.internal.` or isn't `https://` |
| `MICROSOFT_REDIRECT_URI` | `https://cviper.uk/api/auth/microsoft/callback` | Same |
| `LINKEDIN_REDIRECT_URI` | `https://cviper.uk/api/auth/linkedin/callback` | Same |

### Adding a New Environment Variable

Follow the Deployment Checklist:

1. Add to `azure/container-apps.bicep` in the backend container's `env` array
2. Add to `azure/deploy.sh` (`require_secret` for secrets, default for non-secrets)
3. Update CI Docker smoke test in `.github/workflows/ci.yml`
4. Run `tests/test_deploy_config.sh` locally to verify parity
5. Commit all changes together (Fatal Startup Guard Rules — no exceptions)

## Infrastructure as Code

### Bicep Templates

```
azure/
  container-apps.bicep    # Container Apps, Container Registry, networking
  deploy.sh               # Deployment script with secret management
```

### Modifying Infrastructure

1. **Consult the dependency map** — read `docs/INFRASTRUCTURE_DEPENDENCY_MAP.md` and check every component your change connects to
2. Edit the Bicep template
3. Run `az deployment group what-if` to preview changes
4. Update `deploy.sh` if new parameters are needed
5. Update CI smoke test if new env vars are added
6. Run `tests/test_deploy_config.sh` to verify parity (33 checks including live Azure resource validation)
7. Fix **all instances**, not one — if a property applies to multiple container apps, update them all in the same commit

See also: CLAUDE.md "Infrastructure Change Discipline" section for file cluster rules.

### Pre-Deploy Validation

The deploy workflow (`deploy.yml`) runs `tests/test_deploy_config.sh` as a preflight gate before any container is updated. This includes:

- Static checks: nginx config, Dockerfile, Bicep parity, CI permissions, VNet/PostgreSQL consistency
- Live Azure checks (when authenticated): secret existence, VNet integration, activeRevisionsMode, custom domain bindings

If any check fails, the deploy is blocked.

## Rollback

### Quick Rollback (revision switch)

Azure Container Apps keeps previous revisions. To rollback:

```bash
# List revisions
az containerapp revision list -n cviper-backend -g cviper-rg -o table

# Activate previous revision
az containerapp revision activate -n cviper-backend -g cviper-rg --revision <previous-revision>
```

### Full Rollback (redeploy previous commit)

```bash
# Find the last known good commit
git log --oneline -10

# Deploy that specific commit
gh workflow run deploy.yml -f environment=production -f ref=<commit-hash>
```

## Staging Environment

A staging environment is available for pre-production testing:

```bash
gh workflow run deploy.yml -f environment=staging
# Or: ./azure/deploy.sh --env staging
```

Staging uses separate Azure resources but shares the same Bicep templates.

## Monitoring

- **SLOs**: Defined in `docs/SLO.md` — 99.5% availability, p95 latency targets
- **Dashboards**: Grafana at `cviper-grafana` container — latency, error rates, AI provider health
- **Logging**: Loki for structured log aggregation, queryable via Grafana
- **Metrics**: Prometheus custom metrics for AI pipeline and business events
- **Runbooks**: `docs/runbooks/` — 9 operational runbooks covering common failure modes
- **Incident Response**: `docs/INCIDENT_RESPONSE_PLAN.md`

## Troubleshooting

### Container won't start
1. Check Azure Container Apps logs: `az containerapp logs show -n cviper-backend -g cviper-rg`
2. Verify all required env vars are set (run `test_deploy_config.sh`)
3. Check the Docker smoke test in CI — if it passed, the issue is environment-specific

### Database connection fails
1. Check the `DATABASE_URL` env var is set correctly
2. Verify the PostgreSQL server is running and accessible
3. Check network rules (VNet configuration) — backend must be in the same VNet as PostgreSQL
4. Alembic migrations run inside the container entrypoint within VNet; if they fail, check migration idempotency
5. See `docs/runbooks/db-degradation.md`

### CI blocks deployment
1. All required checks must pass: Backend Tests, Frontend Tests, Schema Drift, Docker Smoke, Build
2. If a commit has `[skip ci]`, CI never ran — trigger manually: `gh workflow run ci.yml --ref main`
3. Check for flaky tests — `gh run view <id> --log-failed`

### Alembic migration fails in production
1. Migrations must be idempotent — use `IF NOT EXISTS` patterns
2. Use `batch_alter_table()` for constraint/index operations (SQLite compat in CI)
3. Check `docs/adr/001-dual-database-strategy.md` for dialect-aware migration patterns
4. RLS policies (migration 021) only apply on PostgreSQL — skipped on SQLite via dialect check
