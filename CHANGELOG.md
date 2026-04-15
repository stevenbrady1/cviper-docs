# Changelog

All notable changes to CViper are documented in this file.

Format based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
This project uses [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.5.2] - 2026-04-15

### Added
- **Usage tracking and Free/Pro tiers (CV-093).** Full-stack metering system: `User.tier` column, `UsageDailySummary` table, `UsageLimitMiddleware` enforcing daily limits on 15 AI endpoints, `GET /api/usage` with per-operation breakdown, `GET /api/usage/limits`, `UsageBadge` with per-operation tooltip, `UpgradeModal` on 429. Free tier: 10 AI calls, 3 CV scores, 5 salary estimates, 2 doc generations per day. Pro/Admin: unlimited. 75 tests.
- **Job alerts — live search integration (CV-082).** `AlertService._run_profile_search()` now calls the real job search engine, filters via `seen_jobs` dedup, and creates in-app notifications via `NotificationBell`. Frontend alert toggle on active search profiles. Background loop runs every 900s. 18 tests.
- **AI bias audit Phase 1 (CV-187).** 31 synthetic CV profiles across 5 bias dimensions (name origin, university prestige, career gaps, graduation year, gendered language). 17 prompt-level tests verify FAIRNESS_GUARDRAIL presence, prompt invariance across demographics, and job-relevant-only scoring dimensions. Audit report in `ClaudeReports/audits/`.
- **API contract tests (LESSON-035).** `test_api_response_contracts.py` validates real endpoint response keys match frontend expectations — prevents mock-hiding-mismatch bugs.
- **Benchmark separation guard (LESSON-036).** `test_benchmark_separation_guard.py` seeds both permanent + contract benchmarks and asserts filtered queries never leak the other type.
- **14 E2E tests for UK Regional Salary Comparison.** Negative (empty salary, API error), boundary (zero, negative, large salary), edge (same location, API failure), regression (dropdown population).
- **4 E2E tests for contract rate rendering.** Verifies day rate vs annual display, mixed contract+permanent, null rate_unit fallback.

### Fixed
- **Salary comparison dropdowns empty.** Backend `list_cost_of_living_locations()` returned `{"location": ...}` but frontend read `l.name`. Fixed key to `"name"`.
- **Contractor rates returning blended data.** `get_benchmarks_for_role()` had no `role_type` filter, mixing permanent and contract benchmarks. Added SQL-level filtering, updated all 5 callers.
- **CI stale branch noise.** Session-start hook reported failures from closed PR branches as actionable. Fixed: filter against open PRs, added `types: [opened, synchronize, reopened]` to CI workflow, enabled `delete_branch_on_merge`.

### Changed
- **DPIA updated** with bias audit Phase 1 results, usage tracking limits, solicitor action summary table (7 items), and Appendix A.

## [0.5.1] - 2026-04-13

### Added
- **Career Intelligence UI (CV-137, CV-138, CV-139).** Role Discovery, Career Progression Map (IC + management tracks), and AI Training Plan sections wired into Career Insights and Skills tabs — all backed by existing API endpoints.
- **UK Regional Salary Comparison (CV-072).** Interactive salary comparison card in Companies tab using cost-of-living index across 13 UK locations. Shows purchasing-power equivalent with percentage difference.
- **Legal markdown rendering.** Privacy Policy and Terms of Service now rendered from canonical `docs/*.md` via `react-markdown`, eliminating manual sync between markdown and JSX.
- **AI Provider Settings redesign.** Visual card grid replacing flat key list — brand color accents, provider descriptions, status badges, free tier hints, model selectors, and Get Key links.
- **Progressive tab disclosure (CV-200).** New users see 4 core tabs; secondary tabs unlock after CV upload + search; full tabs after applying or via Settings toggle. 3-tier model (focused/standard/full).
- **Wizard Mode guided onboarding (CV-200).** Auto-starting wizard constrains UI to one tab at a time. Separate step sets for registered and demo users.
- **Desktop tab grouping (CV-200).** 13 tabs reduced to 7 visible via Insights and More dropdowns.
- **Demo value-first experience (CV-200).** First 2 scoring results untruncated for sandbox users. SandboxWelcomeModal redesigned.
- **Data retention schedule + email token cleanup (CV-102).** All 5 retention policies now scheduled in `_scheduled_maintenance()`.
- **Dynamic sub-score weights by role seniority (CV-085).** Fit scoring adjusts weights by seniority level.
- **Security observability for banking enhancements (CV-190).** Monitoring dashboards, anomaly alerts, fingerprint tracking.
- **Few-shot score calibration examples (CV-086).** Three calibration anchors for consistent scoring.
- **Email verification deep-link handler (CV-142).** Secure single-use token verification flow with E2E tests.
- **100% E2E journey coverage (CV-182).** All 48 user journey scenarios covered by Playwright specs.
- **Pre-commit untracked import guard (Layer 8).** Blocks commits when staged files import untracked local modules.
- **E2E tab navigation guard.** Fast-fail spec verifying all 11 E2E-referenced tabs are reachable after mock setup.
- **UI consistency audit.** 54 issues found and fixed, surface style registry + ESLint rule + contract test added.

## [0.4.3] - 2026-04-10

### Fixed
- **P0 cross-user data leak in multi-provider CV generation.** Six `multiGen*` state variables (CV text, cover letter, ATS scores, target job ID) were not cleared on logout or user switch, allowing the previous user's multi-provider CV comparison results to remain visible. Added resets to both `handleLogout()` and the user-switch `useEffect`.
- **Job Search blocked in cloud mode.** The `cvFolder` guard rejected all searches for cloud-deployed users with "Please specify CV folder location or load a profile". Now bypassed when `cloudMode` is true, matching the existing sandbox user exemption.

### Added
- **State cleanup regression guard** (`App.userScopedState.test.js`). Static-analysis test that parses App.jsx and verifies both cleanup paths (handleLogout + user-switch useEffect) reset the same setters, all registered user-scoped setters appear in both, and no unregistered setters slip through. Makes the entire class of missing-state-reset bugs structurally impossible.
- **Pre-launch banner** on the login page — amber "Coming Soon" notice informing visitors that CViper is in development.

## [0.4.2] - 2026-04-10

### Added
- **Persistent generated documents (closes #208).** Generated CVs and cover letters are uploaded to Azure Blob Storage immediately after generation. Lifecycle policy auto-tiers to Cool after 7 days and hard-deletes after 30. Survives container restarts and revisions, no operational burden. Bicep adds the `cviper-documents` blob container, lifecycle policy, and grants the backend's managed identity Storage Blob Data Contributor — auth via `DefaultAzureCredential`, no connection strings or account keys. Backend `helpers/blob_storage.py` wraps the SDK with `upload_file` / `download_document` / `delete_user_blobs` helpers. New `jobs.blob_keys` JSON column via Alembic migration 028. Download route checks blob storage first then falls back to local FS for legacy rows pre-#208. Edited documents are mirrored back via `PUT /api/documents/{id}/text`. Falls through silently when blob storage isn't configured (local dev / CI / on-prem).
- **Blob storage audit logging.** New `diagnosticSettings` on the storage account's blob service streams `StorageRead` / `StorageWrite` / `StorageDelete` events to the existing Log Analytics workspace — same pattern as the Key Vault diagnostic logs. Forensic trail for "who accessed which generated document and when" (GDPR Art. 32 auditability).
- **GDPR right-to-erasure for blob storage.** `/api/gdpr/delete-account` enumerates `users/{uid}/` and deletes every blob before the DB cascade so users get a clean "all data gone" guarantee. Idempotent and non-fatal — failures fall through to the 30-day lifecycle as a backstop. Logs `gdpr_blob_sweep` and `gdpr_blob_cleanup_summary` events for audit correlation.
- **Freelancer.com job source.** New `FreelancerAPI` scraper class wrapping the public REST endpoint `/api/projects/0.1/projects/active/`. Returns active freelance projects (always Contract type) with budget rendered as a project range. Wired into Bicep, `deploy.sh`, and `routes/config.py`. About page Job Sources stat now reads 14 (was 13).
- **176 roles in registration step 4** (was 43). New `_ADDITIONAL_ROLES_BY_SECTOR` dict in `routes/auth.py` merges ~130 commonly-searched UK IT roles into the salary-seed baseline without polluting the curated benchmark data.
- **Industry filter on registration step 6 (Career Pages).** Dropdown above the list filters by industry with per-industry counts. Select All / Clear All scoped to the current filter so users can mass-select within an industry without disturbing prior selections.
- **`Back to sign in` button** at the top of the registration form so users can return to login from any step.
- **`parseServerTimestamp` / `parseServerExpiry`** shared frontend utility for parsing backend ISO timestamps. Treats naive strings as UTC and refuses to return values implausibly far in the past — defence-in-depth for the LESSON-027 sandbox-bounce class of bug.
- **Ruff `DTZ` rules** enabled in `backend/pyproject.toml` to ban naive `datetime.now()` in new code. CLAUDE.md auto-correction rule 21 + LESSON-027 entry document the bug class.
- **Employment Type filter** in Search Criteria — Permanent / Contractor dropdown, filters results client-side on `contract_type`. Search results column relabelled "Salary" → "Salary / Rate".
- **Sticky search status banner** pinned near the top while scrolling — pulsing blue during search, solid green when complete. Shows source count, jobs found, and failure count.
- **Auto-favourite company** when saving a job from search results — career-page favourites reflect the user's interests from day one.
- **Quick-jump to Applications** — "View →" button next to Saved badge, plus "View My Applications →" in bulk save bar.
- **Email pre-check at registration step 1.** Debounced availability check with inline ✓/✗ marker and fail-fast gate.
- **"Don't show again" checkbox** on the Welcome modal so users control whether it reappears on next login.
- **Per-user namespaced localStorage** (`userStorage.js`). Keys stored under `cviper:u:<userId>:<rawKey>` — physically prevents cross-user reads.
- **Friendly error messages** (`errorMessages.js`). Translates HTTP codes to actionable text with troubleshooting hints. 20 worst-offender sites migrated.
- **Skills tab** promoted to first-class top-nav feature with training providers, skill gap analysis, and market demand.
- **AI model badge** in CV Analysis and Document Centre showing actual model used (not just provider).

### Security
- **P0: Cross-user data leak — three layers fixed (LESSON-029 + LESSON-030).** (1) Backend `repo.get_*` scoped with `user_id`. (2) Frontend user-switch effect resets 37+ state slots (was 17). (3) `localStorage` keys migrated to `userStorage` + `purgeAllUserScopedStorage()` on switch/logout. Backend audit test catches unscoped calls at PR time.
- **`completeAuthentication` helper** — single post-login state machine replaces three duplicated auth handlers. `handleLogin` was missing JWT bearer token storage (LESSON-028).

### Fixed
- **AI model switcher crash (React error #31).** `showMessage` calls passed `{type, text}` objects instead of `(text, type)` args.
- **"Please wait for username check"** blocked fast typists at registration step 1. Gate now only blocks on definitive "taken".
- **useSearch state survived user switch.** `siteSummaries`, `lastSearchParams`, `keywordsUsed` leaked between sessions. Exposed `resetSearchState()` in the hook.
- **Work Mode defaulted to stale saved preference.** Now resets to "Any" each session.
- **Document Centre close button** replaced bare "X" with clear "✕ Close" pill. Job rail shows AI model alongside provider.
- **Toast notifications** moved to bottom-centre above the FAB. Floating-element z-index centralised in `utils/layers.js`.
- **Try Demo bounced users back to login (LESSON-027).** Sandbox `expires_at` was serialised as a naive ISO string from `datetime.now()` (Azure container = UTC). JS `new Date()` parses naive strings as local time, so a BST browser saw a fresh 30-minute session as 30 minutes expired and `SandboxBanner`'s countdown fired `onLogout()` on mount. Three-layer fix: backend uses `datetime.now(timezone.utc)`; frontend `parseServerExpiry` appends `Z` and refuses values >30s in the past; `SandboxBanner` rewritten to use the shared util.
- **Folder browser exposed Unix container paths** (third recurrence). `/api/browse-folders` returns 410 in cloud mode. PUT settings scrubs incoming `cv_folder` / `output_folder` starting with Unix server prefixes. New `CLOUD_MODE=true` env var on the backend.
- **Output Folder field in General Settings was unusable** for cloud-hosted users. Frontend now reads `cloud_mode` from `/api/config/settings` and hides the input + Browse button entirely in cloud mode, replaced with an info card. `helpers/documents.py` defaults `output_folder` to `$CLOUD_MODE_OUTPUT_DIR` when empty.
- **OAuth `redirect_uri_mismatch` errors** in production. Bicep now sets explicit `*_REDIRECT_URI` env vars; new startup guard refuses to boot if any contain `.internal.` or aren't `https://`.
- **Try Demo broke 3× in 24h** before LESSON-027 was identified. Restored the 5s `suspendAuthInterceptorFor` window alongside the header gate, exempted `/api/auth/refresh`, added Playwright `try-demo-fetch-storm.spec.js`, CLAUDE.md auto-correction rule 19, and a microtask flush in `handleSandboxLogin`.
- **Demo profile auto-load surfaced a scary "Error loading profile" toast** on the demo landing screen. Added a `silent` flag to `loadProfileIntoForm`; auto-restore on app mount uses `{ silent: true }`.
- **Contract salary estimates showed annual figures** instead of day rates. New `_apply_employment_type_units()` helper converts via `salary_utils.perm_to_day_rate` when `employment_type == "Contract"` and tags with `rate_unit: "day"`. Migration 027 backfills existing rows.
- **Generated documents 410'd silently** when ephemeral `/tmp` files had been wiped. Backend returns 410 with a friendly "regenerate" message; frontend `DocumentCentre` converts the four `<a href>` download tags to `<button>` handlers that fetch via `authFetch` and surface the toast.
- **Showcase SVG diagrams** had overlapping rects, mis-routed arrows, and inconsistent diamond sizes across all three diagrams. All fixed with explicit coordinate adjustments. Stale "23" fallback method count refreshed to "26".
- **Search action bar layout** — LinkedIn caption span pushed History to the right edge. Restructured to two rows: three buttons close together, then the caption full-width below.
- **Salary estimate `rate_unit` persistence** — was tagged on the API response but never stored. Migration 027 adds the column; model + repo + serialiser all carry it through.
- **Login page chip sizes** standardised to 12px / 4px 10px / fontWeight 500 across CV Analysis and Job Search.
- **"New to AI?" toggle** was a near-invisible ghost link. Converted to a tinted pill button.
- **`Search on LinkedIn` button height** misaligned with siblings — pinned `lineHeight: 1.5` + `boxSizing` and shrank the SVG to 12px.
- **Action bar floated over the page** — removed `position: sticky` so it sits inline.

## [0.4.1] - 2026-04-08

### Fixed
- **Try Demo flashes then disappears (root-cause fix).** The global 401 interceptor in `authFetch.js` used to fire the logout handler on any 401 response, which kicked demo users back to the login screen when any of the ~20 parallel post-login reload requests raced the token-propagation window. Replaced the earlier grace-window workaround with a header-gated interceptor: the backend's `AuthMiddleware` now sets `X-Auth-Status: session-expired` on its own 401 response, and the frontend only fires the global logout when that exact header is present. Per-endpoint 401s from individual routes are returned to the caller locally without triggering a logout. Mid-session real expired-session logouts still work correctly. Added two regression tests in `authFetch.test.js` and CLAUDE.md auto-correction rule #18.
- Task Group Preferences dropdowns (CV & Document Generation vs the other three) misaligned in the grid because the longer description wrapped to a second line. Each card is now a flex column with `margin-top: auto` on the select wrapper so all dropdowns sit at the same vertical position regardless of description length.
- Top nav tab bar developed a horizontal scrollbar after the tab count grew from 9 to 14. Compacted the tab buttons (gap 2→1, padding 8×12→6×9, font 12→11.5) and quick-stats card, added `flex: 0 1 auto` + `min-width: 0` so tabs shrink gracefully, and replaced `overflow: auto` with `overflow: hidden` + `text-overflow: ellipsis`. ~130px of horizontal headroom reclaimed; no scrollbar at any typical desktop width.
- Sandbox banner countdown timer wasn't displaying. The `sandbox_expires_at` field was set correctly on the DB record but three separate user-dict serialisers in `routes/auth.py` were not returning it in the API response. Added the field to `auth_status`, `auth_me`, and `_user_response`.
- OAuth 500 errors on `/api/auth/{linkedin,google,microsoft}` caused by authlib 1.6.9 rejecting `state=""`. Only pass the `state` kwarg when there's an actual sandbox session to carry through; omit entirely otherwise so authlib generates its own CSRF nonce.
- "New to AI?" API key guide was unreadable past the top 2 lines on the login page because the form panel had overflow hidden and the expanded guide pushed content past the viewport. Capped the expanded guide at `maxHeight: 50vh` with its own scroll region and inset shadows as a scrollability hint.
- LinkedIn and JP Morgan scraper error classification. LinkedIn 403/connection-reset from cloud egress IPs now classified as `blocked` (not generic `error`) with an honest message pointing users to Reed/Adzuna/career pages. JP Morgan's Oracle HCM API was already working correctly — the bug was that `success_filtered` / `success_empty` statuses had no render branch in the frontend, so they displayed as blank. Added rendering for both statuses plus an "Experimental" badge on LinkedIn.
- Landing page About link was a tiny 13px ghost link almost invisible to recruiters. Replaced with a full-width navy portfolio strip above the Try Demo CTA — labelled "For recruiters & engineers" with real stats from `stats.json` and a "See how →" CTA.
- Sandbox banner copy said "CVs processed in memory and never stored on disk" but the code stores them in PostgreSQL during the session. ToS v2.0 draft acknowledges the inaccuracy; banner correction deferred to a separate UI commit.

### Added
- **GDPR consent work programme** — structured legal review following solicitor preliminary review on 2026-04-07:
  - Commit 1: Full GDPR compliance gap audit saved to `ClaudeReports/audits/2026-04-07-audit-gdpr-compliance-gaps.md`. 480 lines covering sub-processor inventory, per-table data inventory, data flow map, Article 13 gap matrix, and risk-ranked gap summary for solicitor consultation.
  - Commit 2: Drafted Privacy Policy v2.0 (`docs/PRIVACY_POLICY-v2.0-draft.md`) and Terms of Service v2.0 (`docs/TERMS_OF_SERVICE-v2.0-draft.md`) plugging every Article 13 gap. Drafts include 33 `[SOLICITOR CONFIRM]` flags marking legal judgement calls. Live v1.1/v1.0 documents remain in force until solicitor sign-off.
  - Commit 3: Drafted Data Protection Impact Assessment (`docs/DPIA-AI-Profiling-DRAFT.md`) following the ICO template. In-app affiliate disclosure strip on search results, CV upload privacy notice warning about special-category data before AI analysis.
- **Explicit GDPR consent checkboxes** at registration step 7 (required ToS acceptance + required Privacy acknowledgement + optional AI processing consent, unticked by default). Consent records persisted per row to `user_consents` with policy version `2.0`. Accept & Register button disabled until both mandatory boxes are ticked. Server-side validation returns 400 if mandatory boxes are missing.
- **PDF footer** on every generated CV and cover letter PDF: "Generated by CViper (cviper.uk) — AI-assisted draft. Review for accuracy before sharing with employers or recruiters." Drawn on every page via reportlab `onFirstPage` / `onLaterPages` callbacks.
- **Share/Download disclaimer** in Document Centre: persistent strip under the action bar plus a `window.confirm()` pop-up at the moment of clicking Share. Second checkpoint creates the legal record of "we warned them at click-time".
- **Jooble job provider** — free global job aggregator that crawls LinkedIn, Indeed, and company career pages. Works from Azure egress unlike LinkedIn's own guest endpoint. New `JoobleAPI` class in `job_sites_api.py` wired into all three aggregator `site_map` dicts. `JOOBLE_API_KEY` env var. Bare city names auto-enriched with UK country qualifier because Jooble's location parser treats ambiguous cities as no-match.
- **Sandbox session countdown** now actually shows in the SandboxBanner — previously the banner rendered text but the timer line was silently skipped because `sandbox_expires_at` wasn't in the API response.
- **Post-login 401 prevention layers**: standalone silent exception ratchet script (`backend/scripts/check_silent_exceptions.py`) that runs without pytest or a database, wired into the pre-commit hook. Ruff lint config (BLE001, S110) for IDE-level feedback. OAuth endpoint integration tests (`test_oauth_redirect.py`) that hit the real FastAPI endpoints with no sandbox session and assert 302 redirect. Generic "no route returns 5xx" sweep test (`test_no_5xx_sweep.py`) walking every GET `/api/*` route. Status page worker OAuth probes for production synthetic monitoring.
- **Navigation parity test** (`TopNav.navParity.test.jsx`) that parses `App.jsx` for every `activeTab === 'X'` literal and asserts each is registered in the new `config/tabs.js` TABS array. New `config/tabs.js` is the single source of truth for TopNav tabs. Prevents the class of bug where a tab branch is added to App.jsx but never surfaced in the nav.
- **In-app "Behind CViper" discoverability**: renamed the About tab to "Behind CViper" with a cpu icon. New dismissible `AboutPill` component renders a fixed-position pill 30 seconds after login ("How CViper was built") with a 7-day localStorage dismissal.
- **New logo/icon** applied across favicon, PWA manifest, TopNav, and LoginScreen. Source PNG (`cviper-icon-source.png`) committed at repo root; 6 raster sizes + 2 SVG wrappers generated from it via Pillow LANCZOS downsample.

### Changed
- **Tab registry architecture**: the previous hand-drawn `Sidebar.jsx` component (160 lines of inline SVG) was deleted. Navigation is now driven entirely from TopNav + the `config/tabs.js` registry. `CViperLogo.jsx` replaced with a 25-line `<img>` wrapper rendering the new PNG instead of inline SVG.
- **Quick stats moved from TopNav to My Applications page header.** Three stat cards (Results/Saved/Selected) were crowding the desktop top nav and only made sense on Search and Applications tabs. Mobile hamburger menu still shows them.
- **Sandbox demo default job sites disabled**: LinkedIn was enabled by default on sandbox seed but fails from cloud egress with "Failed to connect". First career page (JP Morgan) was auto-enabled but the Oracle HCM scraper returned only US-based roles which got filtered out by the London filter. Both disabled so demo users see a clean empty-state instead of two red ✗ errors.
- Tab row now shows `Career Insights`, `Skills`, `News Feed`, and `My Requests` which had been wired into App.jsx but never added to the TopNav tabs array — effectively unreachable since the Sidebar was removed.

### Security / Compliance
- Explicit `X-Auth-Status: session-expired` header contract between backend `AuthMiddleware` and frontend `authFetch` interceptor. Auto-correction rule #18 added to CLAUDE.md.
- GDPR consent records written to `user_consents` table on registration with policy version tracking.
- Microsoft OAuth userinfo fallback no longer swallows exceptions silently (`logger.warning` added with `event_type: oauth_userinfo_fallback`).

## [0.4.0] - 2026-04-07

### Added
- **Phase 4: UK regional salary expansion + cost-of-living adjustment** (CV-140, CV-141). 8 new UK locations (Bristol, Glasgow, Cambridge, Oxford, Cardiff, Belfast, Reading, Remote UK) in salary benchmark seed data, going from 374 → 648 rows across 13 unique locations. New `cost_of_living_index.json` covering all 13 locations with London=100 baseline. New `/api/salary-comparison` and `/api/cost-of-living-index` endpoints. New `adjust_salary()` helper in `location_service.py` with confidence-level output.
- **Phase 5: Email verification + persistent reminder dismissals**. Verification email sent at signup, verification token table, verify-email endpoint. Reminder dismissal state persisted to DB so dismissals survive reloads.
- **Phase 0: Security hardening** — encrypted secrets at rest via `key_encryption.py`, standard security headers (HSTS, CSP, X-Frame-Options, Referrer-Policy, Permissions-Policy), and fatal startup guards (refuses to start in production without MASTER_KEY, JWT_SECRET_KEY, SECURE_COOKIES, non-wildcard CORS, HTTPS-only CORS).
- **Microsoft OAuth (Entra ID)** social login alongside LinkedIn and Google. Tenant `common` so both personal Microsoft accounts and work/school accounts can sign in.
- **Hybrid JWT/OAuth auth with per-session sandbox users**. Each Try Demo click creates a unique `sandbox_<uuid8>` user with 30-minute TTL and per-session seeded data, replacing the shared sandbox account.
- **Public routes single-source-of-truth registry** (`backend/public_routes.py`) via `register_public()` — eliminates drift between route declarations and the AuthMiddleware's public-path allowlist.
- **Prevention tests for OAuth/auth regressions**: `tests/security/test_oauth_redirect.py` (endpoint-level integration tests), `tests/infrastructure/test_no_5xx_sweep.py` (walks every GET `/api/*` route), status page worker OAuth probes.
- **In-app affiliate disclosure strip** in search results (UK CMA compliant position — near the content, not buried in ToS).
- **CV upload privacy notice** warning about special category data (health, ethnicity, religion, trade union membership) before AI analysis.
- **Dismissible "How CViper was built" pill** in the bottom-left corner of the app. 30-second delay before appearing, 7-day dismissal via localStorage.
- **Landing page portfolio strip** — navy-gradient CTA above the Try Demo button labelled "For recruiters & engineers" with real stats from `stats.json`.
- **Behind CViper tab** (renamed from About, new cpu icon) with the existing showcase content (architecture diagrams, tech stack table, ADR decision cards, screenshot gallery).
- **Navigation parity test** (`TopNav.navParity.test.jsx`) parses App.jsx for every `activeTab === 'X'` literal and asserts each is registered in `config/tabs.js`.
- **Single-source-of-truth tab registry** at `frontend/src/config/tabs.js`. TopNav imports from it.

### Fixed
- **Session startup crash (SECURE_COOKIES fatal guard)** in Docker smoke test — CI was not setting `SECURE_COOKIES=true` in the production smoke environment, causing the backend to refuse startup. Added the env var to the smoke job.
- **Backlog sync workflow failures** on main. The old direct-push workflow was rejected by branch protection. Rewrote to use PR-based sync (`fix: use PR-based sync for backlog workflow to respect branch protection`).
- **CViperLogo.jsx** was a 160-line hand-drawn inline SVG. Replaced with a 25-line `<img>` wrapper rendering the new 256×256 brand PNG. All 6 raster sizes regenerated from `cviper-icon-source.png` via Pillow.
- **Landing page login overflow**: the form panel pushed past the viewport when the API key guide was expanded. Capped at `maxHeight: 50vh` with its own scroll region.
- **"New to AI?" API key explainer wording** rewritten for clarity and accuracy (removed the vague "free pass" metaphor, corrected the "we'll walk you through every step right here on this page" claim).

### Changed
- **Sidebar component deleted entirely**. Was leftover dead code that had been replaced by TopNav but was still imported and referenced. Removed from index.js and from App.jsx.
- **Folder browser refuses to operate in cloud mode**. `/api/browse-folders` was exposing the container filesystem tree (`/app`, `/tmp`, `/home`) to users clicking "Browse" in Settings. Added cloud-mode check and server-path scrubbing on PUT to prevent `/app/...` paths from being stored as user settings.
- **Tab count expanded from 9 to 14**: added Career Insights, Skills, News Feed, My Requests, Feedback to TopNav (they had been wired into App.jsx's render branches but never surfaced in the nav).

### Security / Compliance
- All production startup guards now active: MASTER_KEY, JWT_SECRET_KEY, SECURE_COOKIES, CORS origins, HTTPS-only origins. Backend refuses to start if any are missing or use insecure defaults.
- Silent exception ratchet test (`tests/infrastructure/test_conftest_guards.py`) added to prevent new `except Exception: pass` handlers slipping into production code. Current baseline: 52.
- Docker smoke test promoted to a required CI gate before deploy.

## [0.3.1] - 2026-04-04

### Added
- DAST in CI: OWASP ZAP frontend baseline + API scan integrated into pipeline (soft gate, runs in parallel with integration tests)
- E2E test suite: 10 Playwright specs (42 tests) with shared helpers, route mocking, and CI soft gate integration
- Portfolio case study: redesigned About page with architecture diagrams, ADR decision cards, screenshot gallery, and visual narrative
- Staleness detection CI gates: dead export detection, unused dependency check, frontend `depcheck`
- Multi-agent safe editing: pre-commit/pre-push hooks block commits when behind remote, warn on hot-file conflicts
- CI concurrency guard: serializes runs per branch, cancels stale runs
- Observability foundation: Prometheus metrics, SLOs, recording rules, operational runbooks
- Custom Prometheus metrics for AI pipeline and business events
- iCIMS and Taleo ATS handlers for career page discovery (CV-126)
- Concurrent discovery pipeline for career pages (CV-127)
- Board health monitoring with status tracking (CV-125)
- Responsive foundation with bottom navigation and mobile breakpoints (CV-078)
- useSearch hook extraction from App.jsx (CV-001)
- Step-by-step API key guide for novice users (FAQ + landing page)
- AI-generated content disclaimer: FAQ entry, Settings note, Privacy Policy Section 5 (v1.1)
- Token explainer FAQ entry: what tokens are and how AI models use them
- Prompt Lab read-only mode for demo/sandbox users
- Infrastructure dependency map documenting all Azure component couplings
- Pre-deploy config validation gate in deploy.yml (runs test_deploy_config.sh before deploying)
- Live Azure resource checks in deploy config tests (secrets, VNet, activeRevisionsMode, custom domains)
- Infrastructure change discipline rules in CLAUDE.md (consult dependency map, fix all instances, batch before redeploying)
- Icon completeness test: scans all source files and verifies every referenced icon has a definition
- Custom domain binding verification for cviper.uk and www.cviper.uk
- Server-side path redaction: API never exposes Unix filesystem paths to any user
- SVG hand-crafted sentinel check: drift check verifies diagrams haven't been overwritten by Mermaid
- Azure Key Vault for secrets management (CV-118)
- Security boundary tests (RBAC, sandbox, data isolation)
- Email injection defence and username disclosure fix

### Fixed
- Demo mode tour: "Take a Tour" button added to sandbox welcome modal
- Missing icon definitions (help-circle, minus, x, menu, info, chevron-up, close, cpu, edit, layers, refresh) rendering as text
- SVG diagram arrows crossing over boxes in system architecture, CI/CD, and AI routing diagrams
- Recurring SVG diagram regression (4th occurrence): service worker cached stale Mermaid-rendered diagrams
- E2E sandbox login test: updated "Try Sandbox" to "Try Demo" after rename
- E2E AI settings test: asserted on admin-only element, changed to all-user element
- Server filesystem paths leaking to demo users in Output Folder field
- Top nav, tab bar, and main content horizontal padding misalignment at narrow widths
- CI permissions: `contents: read` missing from Detect Changes job (LESSON-013)
- activeRevisionsMode fix applied to all 4 container app resources, not just 1 (LESSON-014)
- Azure secret wipe during Bicep redeploy made visible in logs (LESSON-015)
- VNet/PostgreSQL private endpoint parity check added (LESSON-016)
- Nginx DNS race condition: deferred resolution prevents startup crash (LESSON-017)
- Docs drift: frontend test file count synced (55 to 56)

### Changed
- AI routing: replaced 3-tier model (premium/standard/light) with priority-based routing and automatic fallback chain
- Single-source stats: `generate_stats.py` writes `stats.json` + `STATS.md`; CI docs-drift gate enforces freshness
- Landing page reordered: demo button first, collapsible API key guide, login below
- Service worker: switched from cache-first with manual version to stale-while-revalidate (auto-invalidation)
- Privacy Policy bumped to v1.1: added AI-Generated Content Disclaimer section
- Deleted `regenerate_diagrams.sh`: hand-crafted SVGs are source of truth, not Mermaid output
- Renamed "Free Trial" to "Demo Mode" across all user-facing text
- CLAUDE.md rule 12 rewritten: SVGs are hand-crafted, never regenerated from .mmd files

## [0.3.0] - 2026-03-27

### Added
- Cloud deployment to Azure Container Apps (cviper.uk)
- PostgreSQL production database (Azure Flexible Server)
- Bicep IaC for all Azure resources
- Manual deploy gate via GitHub Actions
- Cloudflare DNS and security
- AI model routing: 3-tier task routing (premium/standard/light)
- Parallel job processing: up to 5 concurrent Apply jobs with queue UI
- CV analysis persistence to database
- Score detail panel with sub-scores, pros/cons, missing skills
- Company salary estimates with AI-generated ranges
- Document editor: inline CV and cover letter editing
- Salary benchmarks with curated London IT/Financial entries
- Sandbox mode with abuse prevention (fingerprint + IP rate limit + dedicated sandbox providers + truncated outputs + session expiry)
- Session-based authentication (optional, `AUTH_ENABLED` env var)
- Rate limiting on search, AI, and login endpoints
- URL validation and SSRF prevention
- Saved searches with deduplication and work mode filter
- Job comparison side-by-side and Kanban drag-drop board
- Interview prep, follow-up reminders, rejection analysis
- Skills gap analytics dashboard with weekly stats and CSV/XLSX export
- DOCX/PDF document generation with ATS scoring
- Job spec analysis (seniority, essential/desirable skills, contract details)
- Foundation: salary normalization, synonyms, error handling, env config

### Architecture
- Backend: FastAPI with modular AI service package (`backend/ai/`)
- Frontend: React 18 + Vite SPA with state-based tab navigation
- AI: 8 provider clients, constructor-injected services, keyword/fallback systems
- CI: GitHub Actions with test impact analysis, security scans, Docker smoke tests
