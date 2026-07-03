# Testing Strategy & Architecture

## CViper - AI-Powered Job Search & Application Platform

---

| Field | Value |
|-------|-------|
| **Document ID** | TSA-CVIPER-001 |
| **Version** | 0.9.0 |
| **Status** | Pre-Release |
| **Author** | CViper Project Team |
| **Date** | 2026-06-03 |
| **Classification** | Internal |
| **Related BRD** | BRD-CVIPER-001 v0.9.0 |
| **Related FSD** | FSD-CVIPER-001 v0.9.0 |

### Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.9.0 | 2026-06-03 | CViper Project Team | **New guard tests:** frontend↔backend status-enum parity (CV-370); personal-key provider-visibility + `ai-providers ⊇ ai-routing` parity (CV-253); forbid-list against unwrapped `repo.save_company` in routes (CV-288); `ConfigContext` + `UIPrefsContext` provider/consumer contracts (Pilots A + B, +18 tests). **CI reliability:** migration 042 SQLite FK-name fix (Docker smoke green); valid Fernet MASTER_KEY in production-guard smoke; Critical Regression E2E gate unblocked (consent-modal dismissal + `@mobile` specs excluded from the chromium gate via `--grep-invert`); schema-drift / event-type-registry / slow-test-gating fixes; CV-241 tier-expiry cron alert excludes the current run (LESSON-095). **Known flake (quarantined):** `cv-analysis-refresh-prompt` visual baseline `test.fixme` on mobile-safari — Playwright+WebKit drops in-flight intercepted `/api/*` requests under the boot fetch-burst (CV-372 / #640); root cause data-backed, app-side loader-retry fix tracked as follow-up. FE test files now 238. Commit: 82e58ce9. |
| 0.8.0 | 2026-05-21 | CViper Project Team | **New guard classes**: (1) **Prod-middleware-stack regression for shared-secret admin endpoints** (`backend/tests/security/test_cron_auth.py::TestCronAuthMiddlewareExemption`, 3 tests) — `monkeypatch.setattr("auth.AUTH_ENABLED", True)` to force prod-like AuthMiddleware behaviour and assert the response `detail` matches the route dependency's message (`missing or wrong-scheme bearer token` / `invalid token`), not AuthMiddleware's (`Authentication required` / `Token expired or invalid`). Closes the LESSON-091 test gap: the 7 original cron-auth tests passed in isolation because conftest sets `AUTH_ENABLED=false`, masking AuthMiddleware entirely. Adversarially verified RED→GREEN. (2) **Schema-drift recogniser broadening** (LESSON-090 follow-through) — `backend/tests/schema/test_alembic_alignment.py` now uses balanced-paren matching for column extraction (was greedy `(.*?)^\s*\)`) and accepts three idempotency-guard forms (`dialect.name`, `IF NOT EXISTS`, `inspector.has_table()`). Error messages enumerate all three accepted forms so the next contributor sees why their migration was flagged. (3) **ACR-login retry resilience** (`.github/workflows/ci.yml`) — 3-attempt retry wrapper with 5s/10s linear backoff around `az acr login`. Fail-fast property preserved (real auth errors still fail all 3 attempts in <15s). Triggered by one transient `ConnectionResetError(104, 'Connection reset by peer')` on run 26235282986; prevents the recurrence pattern from blocking deploys silently. (4) **Visual-regression mobile route-storm resilience** (`frontend/e2e/visual-baseline-mobile.spec.js`) — `cv-analysis-refresh-prompt` baseline now reload-and-retries once if the prompt is missing; rescues from WebKit's parallel-fetch burst on `initializeApp()` overflowing Playwright's interception queue (61 of 73 `/api/` requests bypassed mocks on 2026-05-20 + 2026-05-21 nightlies). (5) **Active-model chip group-resolution contract** (`frontend/src/components/ActiveModelIndicator.test.jsx`) — new test verifying that when `group` + `aiPriority.resolved_models[providerId][group]` are present, the chip renders the backend-resolved per-group model (mirrors `AIGateway._resolve_quality_override` at call-time). LESSON-038 family — single source of truth via `ai_meta`, not pre-call `provider` variable. (6) **Seed-merge schema-evolution contract auto-discovered the 3 new Japanese-bank entries** (`backend/tests/companies/test_company_boards_merge.py::TestSeedSchemaEvolutionContract`, 17 tests) — proves the rule #52 `{**seed, **user}` defaults-layer merge keeps working as the seed grows. All 351 tests in `tests/companies/` pass on the new total of 229 board entries. **Backend test count** — adjacent increment: +3 (LESSON-091 prod-middleware regression). **Frontend test count** — +1 (ActiveModelIndicator group resolution). **CI changes**: ACR login retry, no other workflow restructuring. **Tooling**: known pre-existing `version-docs.sh` Windows/Git-Bash quirk — `node -p require(...)` fallback chain can fail on Windows path quoting, leaving "App: unknown" in VERSION.md; manually overridden to 0.8.0 in this bump. Filed for follow-up. **Coverage closed for**: LESSON-091. Commit: 1821b5bd. |
| 0.7.0 | 2026-05-17 | CViper Project Team | **New guard classes**: (1) **CV-drafts 7-row coverage matrix** (`docs/cv-drafts-coverage-matrix.md`, CV-326) — formal QAE rubric output for the Phase 1 ADR-008 cards. Per CV item (CV-318 → CV-325), each row maps Happy-path / Negative / Boundary / Edge / E2E / Regression / Exploratory to ✓ Covered / ⚠ Missing / N/A with test file::class::method citations. Aggregate: 75 backend pass / 1 skip · 22 frontend component+contract · 1 lifecycle E2E. (2) **CV-drafts lifecycle E2E** (`frontend/e2e/cv-drafts-lifecycle.spec.js`) — three Playwright tests covering DraftTimeline render, Compare-picker open + diff + close, Promote POST + refetch. Stubs every API call (`/api/cv-drafts`, `/api/cv-drafts/{id}/promote`, `/api/documents/summary`, `/api/documents/{id}`) via `page.route` with state-mutating closure so a single test can simulate "second generation lands → v2 appears → Promote v1 → v1 becomes current". Each test carries a `test.skip()` fallback when DocumentCentre isn't surfaceable from the Applications tab — defers to the per-component Vitest suites in that case. (3) **DraftCompare contract** (`frontend/src/components/DraftCompare.test.jsx`) — 11 tests including the smart-default contract (Left=Base CV when available else v1; Right=current draft), the Base-CV-option presence/absence test, Swap-button direction-flip test, same-selection guard renders `<draft-compare-same>` rather than blank, overlay-click vs dialog-click close semantics, and a swap-direction added/removed-spans test that proves diff direction is correctly applied. (4) **DraftTimeline contract** (`frontend/src/components/DraftTimeline.test.jsx`) — 11 tests covering provider attribution (LESSON-038 family), is_current badge visibility, Promote-button conditional render, ATS chip render, fabrication-verdict surface, refreshKey-driven refetch. (5) **FAQ content forbid-list** (`frontend/src/components/LoginScreen.landingFaqContent.contract.test.jsx`) — 5 tests pinning topic + load-bearing trust signal per FAQ item (privacy: no-sell AND no-recruiter/employer; pricing: free AND no-card; AI fabrication: negation AND invention-term; demo: sample). Forbid-list pattern from LESSON-033 family — wording can evolve, signal coverage is pinned. (6) **Auth-event redaction tests** (CV-354 prep) — `username` field redacted from structured `auth_*` event_type log lines; only `user_id` retained. (7) **Native auth endpoint tests** — `POST /api/auth/apple` (CV-329) and `POST /api/auth/google` (CV-330) verify provider id_token, mint CViper JWT, reject invalid tokens with 401, expose JSON refresh token under `X-Client-Type: native` header. (8) **Device tokens endpoint tests** (CV-331a) — register/list/delete round-trip, idempotent re-register, cross-user isolation. (9) **Surfaces contract caught CV-322 oversight** — `DraftTimeline.jsx:338` shipped with bare `new Date(x).toLocaleString()` which the existing `frontend/src/utils/surfaces.contract.test.js` flagged on the next push (LESSON-033 forbid-list family applied to date-formatter anti-pattern). Fixed to `formatDateTime()` import — proves the architectural guard is doing its job. **Backend test count** — 365 files (was 364, +1 net for the CV-drafts adapter writes + native auth + device-tokens suite). **Frontend** — 215 files (was 212, +DraftCompare/DraftTimeline/landingFaqContent contracts + cv-drafts-lifecycle E2E). Components 84 → 86 (+DraftTimeline, +DraftCompare). Commit: bbd57487. |
| 0.6.4 | 2026-05-07 | CViper Project Team | **New guard classes**: (1) **PageHeader consistency contract** (`frontend/src/utils/page-header-consistency.contract.test.js`) — registry-driven source-scan that asserts every standardised top-level tab imports AND renders `<PageHeader>`. 22 tests: 11 standardised tabs × 2 assertions (import + usage) + 9 EXEMPT_TABS × 2 assertions (file-exists + does-NOT-render-PageHeader, the stale-exemption catcher). Plus a drift-coverage test that asserts the two registries together cover every top-level tab id mentioned in App.jsx — fails CI if a future tab is added without updating either list. Same architectural family as the LESSON-033 forbid-list pattern but applied to component-import contracts. (2) **PageHeader unit tests** (`frontend/src/components/PageHeader.test.jsx`) — 13 tests covering h2 rendering, subtitle as string vs ReactNode, AI-pill conditional render, `providerNameOverride` for CV Analysis's multi-select case, `rightSlot` positioning. Adversarially verified — removing each prop fails the matching test. (3) **Empty env-var anti-pattern forbid-list** (`backend/tests/infrastructure/test_env_var_default_pattern.py`) — source-scans `backend/` (excluding venv + tests) for `int(os.environ.get(NAME, DEFAULT))` / `int(os.getenv(NAME, DEFAULT))` / `float(...)` patterns and fails CI if any reappear. Three tests: main scan + adversarial positive (regex MUST match historical pre-fix forms) + adversarial negative (regex MUST NOT match the safe `or` form OR plain string `os.environ.get(NAME, default)` calls). The contract caught 2 sites in `monitoring.py` that the RCA's hand-listed audit missed. (4) **Updated chip-group contracts** (`SearchTab.modelChipGroup`, `ApplicationsTab.modelChipGroup`) — now component-name-agnostic; accept either `<PageHeader>` or `<ActiveModelIndicator>` since both render the same chip with the same group routing. The contract is on the props (group + aiPriority), not the element. **Backend test count** — 343 files (was 342, +1 env-var contract). **Frontend** 200 files (was 199, +1 PageHeader test, +1 page-header-consistency contract). **Tooling test surface**: `docs/version-docs.sh` fixed — preserved commit-to-version map across runs (was being wiped); three-tier App-Version resolver (node → python → grep) so the manifest stamps a real version regardless of which tools are on PATH. Commit: 6921a085. |
| 0.6.3 | 2026-05-07 | CViper Project Team | **New guard classes**: (1) **Direct Employers favourites-first sort contract** (`frontend/src/utils/companyBoardSort.test.js`) — 6 unit tests including the exact bug-scenario fixture from the user's screenshot (7 favourites scattered alphabetically), null/undefined favourite handling, missing-company-name boundary. (2) **Industry-dropdown / rendered-grid parity contract** (`frontend/src/utils/companyBoardVisibility.test.js`) — 32 tests, with the load-bearing prevention being a 15-permutation parity matrix (5 filter combos × 3 industries) that asserts `countVisibleInIndustry(boards, industry, flags) === applyVisibilityFilters(boards, {...flags, industryFilter: industry}).length` for every combination. Eliminates the dropdown/grid drift class regardless of how the predicate is reorganised in future. Same architectural family as LESSONs 035/043/048/051/052 (single source of truth for state shared across surfaces) but at the in-component filter-pipeline surface. (3) **Mobile cleanup** — no new contracts, but `theme.css` mobile padding sweep is asserted to satisfy `mobile-safety.contract.test.js` (no inline-style violations introduced); `useSearch.test.js` 2 positive assertions converted to forbid-list assertions ("no success-tone toast fires") — LESSON-033 family. (4) **Diagram pipeline contracts** (Rule #57) — 30 unit tests in `backend/tests/infrastructure/test_diagram_gen.py` covering layout, intersection-detection, stats interpolation; 19 contract tests in `frontend/src/utils/svg-quality.contract.test.js`; 18 Playwright tests in `frontend/e2e/diagram-visual-regression.spec.js`; new workflow `.github/workflows/diagram-visual-regression.yml`. Cache-buster chain contract test (LESSON-082 / Rule #58). (5) **Cross-user data layer**: `backend/tests/companies/test_company_boards_persistence.py` (6 tests including the multi-replica drift simulation + source-scan contract that the POST handler calls `get_config(..., for_update=True)`) — LESSON-080 / Rule #55. `backend/tests/infrastructure/test_serialiser_completeness.py` AST-walks every `_*_to_dict` helper in `repositories.py` and asserts each helper exposes every column on its model minus an explicit per-helper exemption set — LESSON-CV-287 / Rule #56. (6) **WebKit + Mobile Safari** — Playwright projects added (tagged subset only); cross-browser regression coverage extended without fanning out the full E2E run. **Stable-selector contracts** — new `data-testid="readiness-badge"` and `readiness-badge-lg` on `<FitScoreBadge>` plus `data-testid="search-source-results-toggle"` on the now-collapsible Source Results card; the LESSON-059 brittle-selector registry remains the canonical pattern for switching tests away from text-regex selectors when labels evolve. **Backend test count** — 342 files (was 333); **frontend** 199 files (was 191). Total +17 test files this release. Commit: e79a2baf. |
| 0.6.2 | 2026-04-30 | CViper Project Team | **New guard classes**: (1) **SearchProfile direct-query forbid-list** (`backend/tests/infrastructure/test_searchprofile_direct_query.py`) — AST scan of every function under `backend/routes/`; any function querying `(database\.)?SearchProfile.user_id ==` MUST also call a fallback helper (`get_user_profile` / `get_latest_cv_analysis` / `get_latest_search`) in the SAME function body, OR carry the `# ALLOWED_DIRECT_SEARCHPROFILE_QUERY:` opt-out annotation. Function-scope check (not file-scope) catches the LESSON-072 bug class regardless of unrelated helpers in sibling functions. 6 tests including positive control, adversarial control, and a function-vs-file scope regression guard. (2) **Login footer-link parity** (`frontend/src/components/LoginScreen.publicLinkParity.test.jsx`) — source-scans `LoginScreen.jsx` for every `<a href="/?tab=X">` link and asserts X is in the auth-gate `PUBLIC_TABS` allowlist or the `score` special-case. RED→GREEN proven: synthetic source with unknown tab is flagged; allowlist sanity check fails LOUDLY if any of privacy/terms/status/score is removed (LESSON-076). (3) **Async health-check tests** (`backend/tests/companies/test_board_health.py`) — 8 new unit tests for `health_check_boards_async`: concurrency cap (Semaphore 4 vs 20 boards), per-board timeout doesn't kill batch, per-board exception isolation, on_result callback fires once per board, callback-exception doesn't abort batch, disabled-skip, empty-input boundary. Plus 1 endpoint integration test simulating `CancelledError` mid-flight to assert the partial-results-persist try/finally contract (LESSON-073). (4) **Public-tab bypass contract** (`frontend/src/App.publicTabs.test.jsx`) — 7 contract tests covering positive (status/privacy/terms render with back-link header) and negative (search/no-tab/admin fall through to login) branches of the auth-gate bypass (LESSON-076). (5) **StatusPage unit tests** (`frontend/src/components/StatusPage.test.jsx`) — 5 tests covering live status, degraded headline, `/api/health` fallback path, error banner, no-authFetch invariant. (6) **Auto-trigger health useEffect tests** (`frontend/src/tabs/search/SearchForm.test.jsx`) — 8 tests covering on-mount fire/skip gating (sandbox, empty, no-enabled, fresh-vs-stale, success state-update). **Multi-agent infra**: staged-set snapshot guard in pre-commit (`frontend/.husky/pre-commit`) catches commit-absorption races (LESSON-074); LESSON-number collision guard in commit-msg hook (LESSON-077). **Backend test count** — 333 files; **frontend** 191 files. Commit: 80fae424. |
| 0.6.1 | 2026-04-26 | CViper Project Team | **New guard classes**: (1) **Admin-registry leak forbid-list** (`backend/tests/infrastructure/test_admin_registry_leaks.py`) — AST/source scan of `gateway.py`; every `for X in self.registry.clients` loop in user-facing code (identified by `provider_errors.append` / `fb_display` in the body) must show a filter-call indicator in the 30 lines preceding it, or the test fails (LESSON-051 / Rule #44). (2) **Per-user provider model source forbid-list** (`backend/tests/infrastructure/test_provider_model_source.py`) — AST scan of every branch in `ProviderRegistry.build_client_config`; reading `os.getenv("..._MODEL", ...)` directly is forbidden, builders must source the `model` field via `self._resolve_model_for(provider_id)` (LESSON-055 / Rule #46). (3) **Personal-key live-model E2E** (`backend/tests/ai/test_personal_key_uses_live_model.py`) — 28 tests including an autoheal-then-rebuild simulation that does NOT mock `build_client_config` (proves the next personal-key build returns the new model). (4) **Contradiction-class UI contract** (`SimpleAISetupCard.test.jsx::describe('contract: Active + Not Connected cannot coexist on the same card')`) — fuzz across realistic prop combinations including the asymmetric scope case (admin-only key + user has only a different one), proves the LESSON-052 contradiction cannot recur (Rule #45). (5) **Truncation-detect-and-retry tests** (LESSON-053) — class-level closure shared across all 5 provider paths; retry at 2× budget is opt-in. **Backend test count** — 6,000+ tests across 302 files; **frontend** 2,300+ tests across 171 files; **E2E** 144 specs (up from 50 at v0.5.5). **Auto-correction rules added**: #44 (admin-registry leak guard), #45 (contradiction-class contract test), #46 (per-user model source forbid-list). Commit: a32272f6. |
| 0.6.0 | 2026-04-23 | CViper Project Team | **New guard classes**: (1) **state-drift prevention registry** (`backend/tests/infrastructure/test_state_drift_prevention.py`) — contract test enumerating every (DB Config row ↔ in-memory state) pair; every declared read endpoint must call its hydrate method or the test fails, catching Azure Container Apps replica drift before CI (LESSON-048). (2) **Read-read parity registry** (`backend/tests/infrastructure/test_read_read_parity.py`) — every pair of GETs exposing the same field must add a `ReadPair` entry; mirrors `PARITY_PAIRS` (LESSON-035) for read-side consistency (LESSON-043). (3) **Live OpenRouter model tests** (`backend/tests/ai/test_openrouter_live_models.py`) — cached live fetch validation, 404 fallback to safety-net, PUT-side validation against live list (LESSON-049 / CV-232). (4) **Provider-model drift canary** (`scripts/check_provider_model_drift.py` + `.github/workflows/provider-drift-check.yml`) — nightly CI check + runtime self-heal (LESSON-050 / CV-233). (5) **Render-and-walk `\uXXXX` literal leak guard** (`frontend/src/components/noUnicodeEscapeLeak.test.jsx`) — catches JSX text-node escape mistakes that source-scanning can't (LESSON-046). (6) **Typography drift detector** — Tier 2 + pre-commit hook (Layer 9) catches inline style objects re-inventing existing surface/typography patterns. **New tests for the CV Optimisation Pipeline**: 21 unit + 11 endpoint + 10 component tests for the Bullet Quality Scorer (`test_bullet_scorer.py`, `test_score_bullets_endpoint.py`, `BulletScorer.test.jsx`) — full 7-row matrix (happy / negative / boundary / edge / unicode / metric variants / regression). **Pre-push Layer 8** (CV-225) — soft nudge when substantive fix commits lack a tracking reference. **Auto-correction rules added**: #43 (read-read parity), #44 (AI string sanitisation), #45 (cloud-user CV resolver). Commit: 9534f137. |
| 0.5.5 | 2026-04-21 | CViper Project Team | Header version re-stamped from 0.2.4 to 0.5.5 — the 0.5.2 row below was added to the history table but the header metadata was never updated, producing doc-version drift that VERSION.md reported incorrectly for ~4 days. New regression-prevention tests documented: `test_regular_user_put_response_has_same_keys_as_get` + 8 aiPriorityReorder helper tests (Rule #36 / LESSON-035), `TestSearchAutoResolveFallback` with explicit `CLOUD_MODE=1` scenario (Rule #37 / LESSON-037), user-scope parity framework (CV-197, Rule #24 guard). Coverage closed for CV-192 through CV-201. Commit: 11778d0b. |
| 0.5.2 | 2026-04-17 | CViper Project Team | History row added when BRD/FSD bumped to 0.5.2, but TSA header was not updated — drift introduced here, corrected at 0.5.5. No TSA content changes in this slot. |
| 0.2.4 | 2026-04-12 | CViper Project Team | Added 7-row Test Design Checklist. Updated E2E to 50 specs (100% journey coverage). Added `@critical-regression` deploy gate (CV-180). Test Design Matrix CI enforcement (CV-172). Updated test counts to 4,400+ backend / 1,500+ frontend. Added `e2eCoverageParity.test.js` enforcement. |
| 0.2.3 | 2026-03-27 | CViper Project Team | Updated gateway retry/circuit breaker test counts (+20 tests). CI schema drift check now uses PostgreSQL 16 service container. Alembic migration 013 batch fix for SQLite compat. |
| 0.2.2 | 2026-03-27 | CViper Project Team | Version reset to align with application semver (pre-release). Consolidates v1.0 content with updated test counts, folder structure, and CI pipeline layout. Prior version archived in `docs/Archive/`. |

> **Note:** Prior to v0.2.2 (v1.0, dated 24 March 2026) an independent numbering scheme was used. That document is preserved in `docs/Archive/` for reference. From v0.2.2 onward, document versions track the application version in `package.json`.

**Audience:** Developers, QA, DevOps, Product
**Scope:** Backend (Python/FastAPI), Frontend (React/Vite), E2E (Playwright), CI/CD (GitHub Actions)

---

## Table of Contents

1. [Testing Philosophy](#1-testing-philosophy)
2. [Test Pyramid & Layer Definitions](#2-test-pyramid--layer-definitions)
3. [Repository Folder Structure](#3-repository-folder-structure)
4. [Naming Conventions](#4-naming-conventions)
5. [Testing Strategy by Layer](#5-testing-strategy-by-layer)
6. [Regression Prevention Plan](#6-regression-prevention-plan)
7. [CI Pipeline Design](#7-ci-pipeline-design)
8. [Cross-Platform Guidance (Windows & macOS)](#8-cross-platform-guidance-windows--macos)
9. [Mocking & Determinism](#9-mocking--determinism)
10. [Performance & Flaky Test Elimination](#10-performance--flaky-test-elimination)
11. [Coverage Strategy](#11-coverage-strategy)
12. [Maintenance & Hygiene](#12-maintenance--hygiene)
13. [Quick Reference](#13-quick-reference)

---

## 1. Testing Philosophy

### Principles

| Principle | What it means in practice |
|-----------|--------------------------|
| Tests are a safety net, not a checkbox | Every test must protect against a real failure mode. Delete tests that verify nothing useful. |
| Fast by default, thorough on demand | Local runs complete in < 90s. Full suite (with integration/quality) runs in CI only. |
| Deterministic always | No test may depend on wall-clock time, network state, filesystem ordering, or global mutable state. |
| Single source of truth | Business rules live in shared modules. Tests verify the module, and contract tests verify all consumers agree. |
| Fail loud, fail early | A broken test blocks the PR. A flaky test is treated as a P1 bug until stabilised or quarantined. |
| Developer experience matters | Tests must be easy to run, easy to read, and easy to debug. Relevant tests runnable in < 10s. |

### Test Budget

| Layer | Target % of suite | Target runtime (local) | Target runtime (CI) |
|-------|-------------------|----------------------|-------------------|
| Unit | 60–70% | < 20s | < 30s |
| Component / Integration | 20–25% | < 40s | < 60s |
| Contract | 3–5% | < 5s | < 10s |
| E2E | 5–10% | N/A (CI only) | < 120s |
| Quality gate (AI) | < 1% | N/A (manual) | N/A (separate workflow) |

---

## 2. Test Pyramid & Layer Definitions

The test suite follows a classic test pyramid, with the majority of tests at the unit level, fewer at the component/integration level, and the fewest at E2E.

### Layer Definitions

| Layer | Scope | Database | Network | Browser | Marker |
|-------|-------|----------|---------|---------|--------|
| Unit | Single function/class | None | None | None | `@pytest.mark.unit` |
| Component | One module in isolation | In-memory SQLite | Mocked | None | `@pytest.mark.component` |
| API / Functional | Single endpoint via TestClient | In-memory SQLite | Mocked | None | `@pytest.mark.api` / `functional` |
| Contract | Multiple endpoints, same rule | In-memory SQLite | Mocked | None | `@pytest.mark.contract` |
| Security | Auth, RBAC, SSRF, rate limiting | In-memory SQLite | Mocked | None | `@pytest.mark.security` |
| Integration | Real network I/O (job boards) | In-memory SQLite | Real | None | `@pytest.mark.integration` |
| E2E | Full stack (browser → API → DB) | SQLite file | Real (localhost) | Chromium | N/A (Playwright) |
| Quality gate | AI output scoring | In-memory SQLite | Real AI | None | `@pytest.mark.quality` |

---

## 3. Repository Folder Structure

### Backend Tests (447 files)

```
backend/tests/
├── conftest.py                     # Session fixtures, DB isolation, mocks
├── ai/                             # AI service, scoring, routing (33 files)
│   ├── fake_ai_gateway.py          # Reusable test double
│   └── test_keyword_matching.py ...
├── api/                            # HTTP endpoint tests (16 files)
├── companies/                      # Company boards, salary (7 files)
├── cv/                             # CV analysis, generation (10 files)
├── data/                           # Repository CRUD, migrations (9 files)
├── features/                       # Analytics, deep analysis, reminders (8 files)
├── infrastructure/                 # Admin, encryption, monitoring (12 files)
├── scoring/                        # Fit score, history (4 files)
├── search/                         # Job search, career pages (22 files)
└── security/                       # Auth, RBAC, GDPR, sandboxing (23 files)
```

### Frontend Tests (301 files)

```
frontend/src/
├── App.test.jsx                    # Main integration tests
├── components/*.test.jsx           # Co-located with source (24 files)
├── hooks/*.test.js                 # Hook tests (4 files)
├── tabs/*.test.jsx                 # Tab component tests (5 files)
├── utils/*.test.js                 # Utility tests (3 files)
└── test/
    ├── setup.js                    # @testing-library/jest-dom
    ├── appTestHelpers.js           # Global fetch mock, default API responses
    └── timerConstants.js           # Test timer config
```

### E2E Tests (Playwright)

144 specs providing 100% user journey coverage. Key specs include:

```
frontend/e2e/
├── smoke.spec.js                   # Main paths after deployment
├── deploy-smoke.spec.js            # Critical regression checks for deploy gate
├── cv-upload.spec.js               # CV upload workflows
├── cv-upload-boundaries.spec.js    # Upload boundary/negative tests
├── cv-analysis-ai.spec.js          # AI-powered CV analysis
├── cv-editing-export.spec.js       # Edit + export generated documents
├── search-streaming.spec.js        # Real-time search updates
├── job-search-flow.spec.js         # Full search user journey
├── job-scoring.spec.js             # Fit scoring workflows
├── document-editing.spec.js        # Document editing workflows
├── applications-lifecycle.spec.js  # Full applications CRUD lifecycle
├── auth-register.spec.js           # Registration flow
├── auth-negative.spec.js           # Auth negative/boundary tests
├── email-verification.spec.js      # Email verification deep-link
├── cross-user-isolation.spec.js    # Multi-user data isolation
├── try-demo-fetch-storm.spec.js    # Demo login fetch-storm regression
├── about-showcase.spec.js          # About page and showcase gallery
├── company-salary.spec.js          # Company salary estimation flows
├── essential-controls.spec.js      # Close/Download always visible
├── floating-layout.spec.js         # Floating element overlap checks
├── gdpr-account.spec.js            # GDPR data export/erasure
├── ... (50 specs total — see frontend/e2e/ for full list)
└── visual-regression.spec.js       # Visual regression snapshots
```

**Enforcement:** `e2eCoverageParity.test.js` fails CI if any tab or modal lacks a matching E2E spec.

### Structure Rationale

| Decision | Reason |
|----------|--------|
| Backend tests in `tests/` (separate) | FastAPI convention; avoids shipping test code in Docker; simpler coverage exclusion. |
| Frontend tests co-located with source | React convention; easier to find and maintain; Vitest auto-discovers. |
| Subdirectories by domain, not layer | Developers think in features ("I changed search"), not layers. |
| Contract tests inside domain folder | Primary audience is the domain owner. |
| E2E tests in separate `e2e/` folder | Different runner (Playwright), different lifecycle, different CI stage. |

---

## 4. Naming Conventions

### Backend (Python / pytest)

```
File:     test_{domain}_{aspect}.py
          test_keyword_matching.py
          test_provider_visibility_contracts.py

Function: test_{action}_{scenario}[_{expected_outcome}]
          test_fallback_analysis()
          test_regular_user_never_sees_ollama_on_any_endpoint()

Class:    Test{Feature}{Aspect}
          TestLocalOnlyHiddenContract
          TestGoogleGeminiProvider
```

### Frontend (JavaScript / Vitest)

```
File:     {ComponentName}.test.jsx
          {ComponentName}.{feature}.test.jsx

Describe: describe('{Component or AC-N: Acceptance Criteria}')
Test:     it('{does something specific}')
```

### E2E (Playwright)

```
File:     {feature-slug}.spec.js
Test:     test('{user action} → {expected outcome}')
```

### Markers / Tags

Every backend test file must have a `pytestmark` at module level:

| Marker | When to use |
|--------|-------------|
| `unit` | Pure logic, no I/O, no DB, no HTTP |
| `component` | Tests one module with real DB (in-memory) |
| `api` | Tests an HTTP endpoint via TestClient |
| `contract` | Same rule holds across multiple endpoints |
| `security` | Auth, RBAC, SSRF, rate limiting, encryption |
| `regression` | Guards against a previously-fixed bug |
| `integration` | Makes real network calls (excluded from default CI) |
| `quality` | AI output quality gates (manual only) |
| `acceptance` | Given/When/Then business requirement tests |
| `sanity` | Validates test infrastructure itself |
| `smoke` | Post-deployment verification |
| `gdpr` | Data export, deletion, consent |
| `allow_ai_gateway` | Opt out of AI mock |

---

## 5. Testing Strategy by Layer

### 5.1 Unit Tests

- **What to test:** Pure business logic (score calculation, keyword matching, date parsing), data transformations, validation rules, utility functions.
- **What NOT to test:** Database queries, HTTP response codes, multi-step workflows.
- **Fixture strategy:** Use `shared_ai_service` for read-only AI methods (saves ~6s/test). No database or HTTP mocks needed.

### 5.2 Component / Integration Tests

- **What to test:** Repository CRUD against in-memory SQLite, single API endpoint request/response, AI service with mocked gateway, background task execution.
- **Fixture strategy:** `client` (TestClient), `db_session` (direct DB), `_block_ai_gateway` and `_mock_outbound_http` autouse mocks.

### 5.3 Contract Tests

**Purpose:** Verify that the SAME business rule is enforced consistently across ALL endpoints. If a rule is applied in one endpoint but missed in another, these tests fail.

- `test_provider_visibility_contracts.py` — Ollama/Pluribus hidden from non-admin on 3 endpoints
- `test_company_boards_contracts.py` — Seed data merged into 4 endpoints

**When to add:** A business rule must hold across 2+ endpoints; a regression was caused by fixing one endpoint but missing another.

### 5.4 Security Tests

| Category | What's tested | Example file |
|----------|--------------|--------------|
| Authentication | Login, logout, session management | `test_auth.py` |
| Authorisation (RBAC) | Role-based access to admin endpoints | `test_rbac_enforcement.py` |
| Data isolation | User A cannot see User B's data | `test_user_data_isolation_ac.py` |
| Input validation | URL validation, path traversal | `test_url_validation.py` |
| Rate limiting | Endpoint-level throttling | `test_rate_limiting.py` |
| Encryption | API keys encrypted at rest | `test_key_encryption_at_rest.py` |
| GDPR | Data export, deletion, consent | `test_gdpr.py` |
| Sandbox | Abuse prevention, session isolation | `test_sandbox_abuse_prevention.py` |
| OAuth | External auth provider flows | `test_oauth.py` |

### 5.5 E2E Tests (Playwright)

144 specs providing 100% user journey coverage. Representative specs:

| Spec | User journey |
|------|-------------|
| `smoke.spec.js` | Main paths after deployment |
| `deploy-smoke.spec.js` | `@critical-regression` tagged — blocks deploys |
| `cv-upload.spec.js` | Upload CV → see analysis results |
| `cv-analysis-ai.spec.js` | AI-powered CV analysis workflow |
| `search-streaming.spec.js` | Start search → see streaming results |
| `applications-lifecycle.spec.js` | Full applications CRUD lifecycle |
| `email-verification.spec.js` | Email verification deep-link flow |
| `cross-user-isolation.spec.js` | Multi-user data isolation (P0 regression) |
| `try-demo-fetch-storm.spec.js` | Demo login fetch-storm regression guard |
| `essential-controls.spec.js` | Close/Download buttons always visible |
| `gdpr-account.spec.js` | GDPR data export and erasure |
| `job-scoring.spec.js` | Fit scoring workflows |
| `about-showcase.spec.js` | About page renders, showcase gallery loads |
| `company-salary.spec.js` | Company salary estimation flows |
| `settings-ai-config.spec.js` | AI provider settings configuration |

See `frontend/e2e/` for all 50 specs and `frontend/e2e/test-plan-coverage.md` for the full coverage matrix.

**Shared Helpers** (`e2e/helpers/`): `mocks.js` (route interception), `test-data.js` (users, jobs, documents), `actions.js` (navigation, login). All tests use Playwright route mocking — no real backend needed.

**CI Integration:** E2E runs after frontend tests pass. Tests tagged `@critical-regression` act as a **hard gate on deploys** (CV-180) — deploy workflow fails if any critical regression test fails. Remaining E2E tests run as a soft gate (`continue-on-error`). Single Chromium worker, 2 retries, Playwright HTML report + JUnit XML uploaded as artifacts. Browser binaries cached across runs.

**Enforcement:** `e2eCoverageParity.test.js` (CV-182) runs in CI and fails if any tab or modal lacks a matching E2E spec. Every new tab/modal must ship with E2E coverage.

**Guidelines:** Test user journeys not implementation details. Use `data-testid` attributes. Each spec independent. Use Playwright auto-waiting — never `sleep()`.

### 5.6 Quality Gate Tests (AI Output)

**Purpose:** TDD-style feedback loop for prompt engineering. Uses real AI APIs. NOT run in CI by default. Run manually after changing AI prompts.

---

## 6. Regression Prevention Plan

### 6.1 Root Cause Analysis of Recurring Issues

| Recurring bug | Root cause | Prevention |
|--------------|------------|------------|
| Careers page missing data | Board seed data not merged in new endpoint | `company_boards_merge.py` + contract tests |
| Provider leaking to wrong role | Visibility rules duplicated across 3 endpoints | `provider_visibility.py` + contract tests |
| Services not starting | Missing env vars / config not validated | `test_config_smoke.py` + startup diagnostics |
| Logging/monitoring disappearing | Config changes removing monitoring | `test_monitoring.py` + `test_monitoring_endpoints.py` |
| Rate limiter breaking tests | State leaking between tests | `reset_rate_limiter` autouse fixture |
| AI calls in tests (slow/flaky) | Test making real AI calls | `_block_ai_gateway` autouse + `shared_ai_service` |
| New user sees admin features | Role check missing on new endpoint | `test_rbac_enforcement.py` + contract tests |
| Database schema drift | Model change without migration | Alembic migration + `test_conftest_guards.py` + CI PostgreSQL schema drift check |

### 6.2 Structural Safeguards

**Single source of truth modules:**
- `provider_visibility.py` — all provider filtering rules
- `company_boards_merge.py` — all board data merging

**Autouse fixtures in conftest.py:**
- `_block_ai_gateway` — blocks ALL AI calls unless opted out
- `_mock_outbound_http` — blocks ALL external HTTP
- `_mock_cv_analysis` — prevents real CV analysis
- `_mock_adaptive_ai` — prevents adaptive parser AI calls
- `_mock_background_salary_check` — prevents background AI calls
- `reset_rate_limiter` — resets rate limits between tests
- `clean_db_between_tests` — resets DB state

### 6.3 Regression Prevention Checklist (Code Reviews)

- **New endpoint?** → Needs provider visibility filtering? Board seed merging? RBAC? Add to contract tests.
- **Business rule changed?** → In single-source module? All consumers updated? Contract tests updated?
- **New DB model/column?** → Added to `clean_db_between_tests`? Migration created?
- **New external API call?** → Mocked in conftest.py? Blocked by autouse fixtures?
- **Changed AI prompt?** → Run quality gate tests manually.
- **Changed auth/role logic?** → RBAC and data isolation tests updated?

---

## 7. CI Pipeline Design

### 7.1 Pipeline Stages

| Stage | Trigger | What runs | Gate |
|-------|---------|-----------|------|
| Detect Changes | All | `paths-filter`: backend/frontend changed? | N/A |
| Backend Tests | Backend changed or push | 9 test areas in parallel (`-n auto`), coverage summary | Hard gate |
| Frontend Tests | Frontend changed or push | 5 test areas + coverage via `@vitest/coverage-v8` + build | Hard gate |
| Lint | All | flake8 + ESLint + `npx depcheck` (unused frontend deps) | Soft gate |
| Docs Drift & Code Hygiene | All | `docs_drift_check.py` (stats.json freshness, stale patterns, sandbox refs), `dead_exports_check.py` (unused components/hooks), `unused_deps_check.py` (unused backend packages) | Hard (drift) / Soft (exports, deps) |
| Security SCA | All | pip-audit, npm audit, Snyk | Soft gate |
| Security SAST | All | Snyk Code, Aikido, secrets, SSRF | Soft gate |
| Schema Drift Check | Backend changed | Alembic autogenerate against PostgreSQL 16 service container | Hard gate |
| Test Design Matrix | PRs | Checks PRs for 7-row coverage matrix (CV-172) | Hard gate |
| E2E (Playwright) | After frontend tests | 144 specs, `@critical-regression` subset blocks deploys (CV-180) | Soft (general) / Hard (deploy) |
| Integration Smoke | After tests pass | Start backend, hit 5 endpoints | Hard gate |
| CI Summary | Always | Aggregate results, JUnit XML report | Final gate |
| Build & Push | Main push only | Docker images to ACR | N/A |

### 7.2 Backend Test Areas in CI

| CI Step | Test directory | Approx. tests | Approx. time |
|---------|---------------|---------------|--------------|
| AI & Scoring | `tests/ai/` `tests/scoring/` | ~370 | ~15s |
| API Endpoints | `tests/api/` | ~250 | ~10s |
| Search | `tests/search/` | ~400 | ~12s |
| Companies & Salary | `tests/companies/` | ~120 | ~5s |
| CV & Documents | `tests/cv/` | ~150 | ~8s |
| Data & Repository | `tests/data/` | ~200 | ~6s |
| Security | `tests/security/` | ~350 | ~10s |
| Features | `tests/features/` | ~100 | ~5s |
| Infrastructure | `tests/infrastructure/` | ~80 | ~4s |

### 7.3 What Runs Where

| Test type | Local | CI (PR) | CI (main) | Manual |
|-----------|-------|---------|-----------|--------|
| Unit | Yes | Yes | Yes | — |
| Component | Yes | Yes | Yes | — |
| Contract | Yes | Yes | Yes | — |
| Security | Yes | Yes | Yes | — |
| Integration | No (opt-in) | No | No | Yes |
| E2E (Playwright) | No (opt-in) | Yes (soft gate; `@critical-regression` = hard on deploy) | Yes (soft gate; `@critical-regression` = hard on deploy) | Yes |
| Quality gate | No | No | No | Yes |
| Coverage | No | Yes (3.12) | Yes (3.12) | — |
| Lint | No (opt-in) | Yes | Yes | — |
| Security scans | No | Yes | Yes | — |

---

## 8. Cross-Platform Guidance (Windows & macOS)

### 8.1 Environment Setup

| Requirement | Windows | macOS |
|-------------|---------|-------|
| Python | 3.11 or 3.12 via python.org | `brew install python@3.12` |
| Node.js | 20.x or 22.x via nodejs.org | `brew install node@22` |
| Shell | Git Bash (ships with Git) | Terminal (zsh) |
| Playwright | `npx playwright install chromium` | Same |

### 8.2 Running Tests

```bash
# Backend
cd backend
pytest                              # Default: parallel
pytest -n0                          # Sequential (debugging)
pytest -n0 -x -vvs tests/ai/       # Debug one area
pytest -m "not integration"         # Skip network tests

# Frontend
cd frontend
npm test                            # Vitest (CI mode)
npm run test:watch                  # Watch mode
npm run test:e2e                    # Playwright headless
```

### 8.3 Platform-Specific Gotchas

| Issue | Windows | macOS | Fix |
|-------|---------|-------|-----|
| Path separators | Backslash | Forward slash | Use `pathlib.Path` — never hardcode |
| Line endings | CRLF | LF | `.gitattributes` normalises |
| Temp directory | `AppData\Local\Temp` | `/tmp` | Use pytest `tmp_path` or `tempfile` |
| SQLite locking | Less aggressive | WAL mode | `StaticPool` in conftest |
| Case sensitivity | Case-insensitive | Case-sensitive | Use exact filenames. CI runs on Linux. |
| Port conflicts | Antivirus may block | Rarely | Smoke test retries. E2E `reuseExistingServer`. |

---

## 9. Mocking & Determinism

### 9.1 Mock Architecture

| Real World | Test Mock | Scope |
|-----------|-----------|-------|
| AI Providers (OpenAI, Google, etc.) | `_block_ai_gateway` → stub JSON | Autouse |
| External HTTP (job boards) | `_mock_outbound_http` → stub HTML | Autouse |
| Database (PostgreSQL / SQLite) | In-memory SQLite + `StaticPool` (tests); PostgreSQL 16 service container (CI schema drift) | Session |
| CV Analysis pipeline | `_mock_cv_analysis` → fallback profile | Autouse |
| Rate Limiter (slowapi) | `reset_rate_limiter` → `.reset()` | Autouse |

### 9.2 Mock Rules

| Rule | Why |
|------|-----|
| Mock at the boundary, not inside | Mock `AIGateway.call()` (chokepoint). One mock covers all 9 providers. |
| Autouse for safety, opt-out for intent | All outbound blocked by default. Opt out with `@pytest.mark.allow_ai_gateway`. |
| Never mock the thing you're testing | Testing keyword matching? Don't mock keyword service. |
| Session-scoped for expensive resources | `shared_ai_service` (6s init), `isolate_database`. |
| Function-scoped for state | `clean_db_between_tests`, `reset_rate_limiter` run per-test. |

### 9.3 Determinism Checklist

| Non-determinism source | Mitigation |
|-----------------------|------------|
| Wall-clock time | Use `freezegun` or mock `datetime.now()` |
| AI response variance | AI gateway blocked; quality tests separate |
| DB auto-increment IDs | Never assert on exact IDs; use `filter_by()` |
| Rate limiter state | Reset between tests |
| Global singletons | `deps._invalidate_cache()` in cleanup |

---

## 10. Performance & Flaky Test Elimination

### 10.1 Current Performance

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| Backend (sequential) | ~380s | ~234s | 38% faster |
| Backend (parallel) | ~380s | ~60s | 84% faster |
| Frontend (Vitest) | ~47s | ~47s | (already fast) |
| Total local run | ~430s | ~107s | 75% faster |

### 10.2 Optimisations Applied

| Optimisation | Savings | How |
|-------------|---------|-----|
| `shared_ai_service` fixture | ~220s | Session-scoped init (was 6.3s x 35) |
| pytest-xdist parallel | ~60% wall time | `-n auto --dist worksteal` |
| Selective DB cleanup | ~30s | Skip cleanup for pure unit tests |
| Autouse mock layering | ~200s prevented | Blocks AI/HTTP calls |
| `worksteal` distribution | ~10% vs loadgroup | Dynamic rebalancing |

### 10.3 Flaky Test Prevention

| Pattern | Detection | Prevention |
|---------|-----------|------------|
| Timer-dependent | Passes alone, fails in suite | `await waitFor()` with assertions, never `sleep()` |
| Order-dependent | Passes in isolation | `clean_db_between_tests` + cache invalidation |
| Rate-limit bleed | Sporadic 429 errors | `reset_rate_limiter` autouse |
| Port conflict | E2E can't start server | `reuseExistingServer` in Playwright |
| Network-dependent | Real API that's down | All HTTP mocked by default |

### 10.4 Flaky Test Quarantine Process

1. **Detect:** CI reports flaky test (passes on retry)
2. **Quarantine:** `@pytest.mark.skip(reason="FLAKY: #123")`
3. **Investigate:** Within 48 hours, diagnose root cause
4. **Fix:** Address non-determinism, remove skip marker
5. **Monitor:** Watch 1 week before considering stable

---

## 11. Coverage Strategy

### 11.1 Targets

| Component | Statements | Branches | Functions | Rationale |
|-----------|-----------|----------|-----------|-----------|
| Backend | 60% (current) | — | — | Growing towards 75% |
| Frontend | 80% | 75% | 80% | Configured in `vite.config.js` |
| E2E | Not measured | — | — | About user journeys, not lines |

### 11.2 Priority Code Paths

| Priority | Code path | Target |
|----------|----------|--------|
| P0 | Auth (login, session, RBAC) | 90%+ |
| P0 | Data isolation (user A ≠ user B) | 90%+ |
| P1 | AI service fallback chain | 80% |
| P1 | API endpoints | 85% |
| P2 | Search parsers | 75% |
| P2 | Document generation | 70% |
| P3 | Admin/config endpoints | 60% |

---

## 12. Maintenance & Hygiene

### 12.1 Test Debt Signals

| Signal | Action |
|--------|--------|
| Test takes > 5s without `@pytest.mark.slow` | Profile it. Usually an unmocked external call. |
| Test name doesn't describe what it verifies | Rename to `test_{action}_{scenario}_{expected}`. |
| Asserts on exact error message strings | Assert on status codes and structure instead. |
| Creates DB records AND uses `client` | Pick one: direct DB or via API. |
| `# TODO: fix this test` older than 30 days | Fix or delete. |
| Test file has > 500 lines | Split by feature area. |

### 12.2 Quarterly Review

- **Slowest 20 tests** (`pytest --durations=20`) — optimise or move to separate stage
- **Skipped tests** — still relevant? Unskip or delete.
- **Coverage gaps** — new critical paths uncovered?
- **Contract test completeness** — new endpoints missing?
- **Marker accuracy** — still correct?

### 12.3 New Test Decision Tree

1. Pure function? → **Unit test** (`@pytest.mark.unit`)
2. Single API endpoint? → **API test** (`@pytest.mark.api`)
3. Rule across multiple endpoints? → **Contract test** (`@pytest.mark.contract`)
4. Requires browser? → **E2E** (Playwright)
5. Real network calls? → **Integration** (`@pytest.mark.integration`)
6. Otherwise? → **Component** (`@pytest.mark.component`)

---

## 13. Quick Reference

### Run Commands

```bash
# Backend
pytest                              # Full suite (parallel)
pytest -n0 -x -vvs tests/ai/       # Debug one area
pytest -m unit                      # Unit tests only
pytest -m contract                  # Contract tests only
pytest -m security                  # Security tests only
pytest --durations=20               # Show 20 slowest
pytest --cov=. --cov-report=term    # With coverage

# Frontend
npm test                            # Vitest (CI mode)
npm run test:watch                  # Watch mode
npx vitest run src/components/      # One directory
npm run test:e2e                    # Playwright headless
npm run test:e2e:headed             # With browser
```

### Fixture Quick Reference

| Fixture | Scope | Purpose |
|---------|-------|---------|
| `shared_ai_service` | Session | Pre-initialised AIService (saves 6s/test) |
| `isolate_database` | Session | In-memory SQLite |
| `isolate_data_dirs` | Session | Temp config/db directories |
| `default_user_id` | Session | Default user's UUID |
| `client` | Function | FastAPI TestClient |
| `db_session` | Function | SQLAlchemy session |
| `sample_job_data` | Function | Sample job dict |
| `sample_search_file` | Function | Search record in DB |
| `sample_saved_job` | Function | Saved job record in DB |
| `temp_cv_dir` | Function | Temp dir with sample CVs |
| `clean_db_between_tests` | Function (autouse) | Wipes DB between tests |
| `reset_rate_limiter` | Function (autouse) | Resets slowapi |
| `_block_ai_gateway` | Function (autouse) | Blocks AI calls |
| `_mock_outbound_http` | Function (autouse) | Blocks external HTTP |

### Current Test Counts (as of v0.6.1)

| Suite | Files | Tests |
|-------|-------|-------|
| Backend (pytest) | 302 | 6,000+ |
| Frontend (Vitest) | 171 | 2,300+ |
| E2E (Playwright) | 144 | 400+ |
| **Total** | **617** | **8,700+** |

> Source: `python scripts/generate_stats.py` (refreshes `frontend/src/data/stats.json` and `docs/STATS.md`).
> About-page stats hydrate from `stats.json` so on-page counters stay aligned with this table.

### 7-Row Test Design Checklist (Mandatory)

Every new feature or bug fix must consider all 7 test design approaches. The Test Design Matrix CI gate (CV-172) enforces this on PRs.

| # | Type | When Required |
|---|---|---|
| 1 | **Happy-path** | Always |
| 2 | **Negative** | Any user input or external boundary |
| 3 | **Boundary** | Any numeric, length, size, count, or date limit |
| 4 | **Edge cases** | User-supplied data, time, encoding, dual-environment |
| 5 | **E2E** | Any workflow crossing ≥3 layers |
| 6 | **Regression** | Any bug fix (mandatory) |
| 7 | **Exploratory** | Manual, before release of significant features |

QAE must flag PRs shipping only happy-path tests. CR must reject PRs missing boundary/negative coverage on user input.

---

*This document should be reviewed and updated quarterly, or whenever the test architecture changes significantly.*
