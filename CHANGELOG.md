# Changelog

All notable changes to CViper are documented in this file.

Format based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
This project uses [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- **Release tooling (CV-390)** — releasing is now a single command. `scripts/bump_version.py major|minor|patch | --set X.Y.Z` updates `frontend/package.json`, the new `backend/version.py` (`__version__`, what production `/api/version` now actually serves instead of `"unknown"`), and this changelog in one shot. A forbid-drift CI guard (`scripts/check_version_consistency.py` + pytest twin) fails the build if the three locations ever disagree. Pushing a `vX.Y.Z` tag auto-deploys that release to **staging**; production deploys remain manual (`workflow_dispatch`, new `version_tag` promotion input). `/api/health` now also reports the running version.

## [0.9.0] - 2026-06-03

### Added

- **Cross-border consent system (CV-353)** — GDPR Art. 44-49 region map plus two consent modals (special-category data + cross-border processor). A new `ConsentGate` gates every AI call and CV upload, prompting the user before their data leaves for a non-EU provider. The cross-border modal shows the provider's display name rather than its internal id (CV-353 follow-up).
- **Push notifications (CV-348)** — unified `push_service` dispatch layer, FCM HTTP v1 send module (Android), a daily-digest method + cron endpoint, and a saved-search trigger. Deploy parity added for `FCM_SERVICE_ACCOUNT_JSON`.
- **Status enums API (CV-370)** — `/api/enums` exposes status values as a single source of truth, with a frontend↔backend membership-aware parity guard so the two can never silently drift.
- **Per-user circuit breaker (CV-250)** — the AI circuit breaker is now scoped per-user for personal-key calls, so one user's provider outage can't trip the breaker for everyone.

### Changed

- **App.jsx god-component de-drilling (Pilots A + B)** — `configMode` extracted into `ConfigContext`; `advancedMode` and `showAllTabs` extracted into `UIPrefsContext`. Context-per-domain strategy with no behaviour change; reduces prop-drilling out of the ~5,200-line `App.jsx`.

### Security

- **CV-202** — admin-gate global AI-routing writes, closing a non-admin RBAC hole.
- **CV-354** — redact username/email PII from auth and GDPR structured logs.
- **CV-257** — surface a `decrypt_failed` signal when a user's personal key is corrupt (instead of a silent admin-key fallthrough).
- **CV-253** — acceptance guards for personal-key provider visibility and `ai-providers ⊇ ai-routing` parity.
- **CV-288** — companies write paths (bulk-estimate, salary-check) wrapped in a collision-safe helper; forbid-list guard against unwrapped `repo.save_company` in routes.

### Fixed

- **Migration 042** — SQLite branch now drops the real FK name (`fk_prompt_log_user`), unblocking the Docker production-smoke test on fresh SQLite.
- **CI Docker smoke** — valid Fernet `MASTER_KEY` supplied to the production-guard check.
- **Critical Regression E2E gate** — unblocked: `cloud-mode-cv-search` now dismisses the cross-border consent modal; `@mobile` specs excluded from the chromium gate (they belong to the mobile-safari project).
- **CV-241** — tier-expiry cron failure alert excludes the current run from its prev-run lookup (LESSON-095).
- Schema-drift, event-type-registry, and slow-test-gating CI fixes.

### Known Issues

- **CV-372 / #640** — the `cv-analysis-refresh-prompt` mobile visual baseline is quarantined (`test.fixme`) on mobile-safari. Root cause (data-backed): Playwright + WebKit drops a fraction of in-flight intercepted `/api/*` requests under the boot fetch-burst. The real fix (app-side retry on boot data-loaders) is tracked as a follow-up; the test still runs on chromium.

## [0.8.0] - 2026-05-21

> Backfilled 2026-06-24 — this release shipped at commit `1821b5bd` and was recorded in the BRD/FSD/VERSION manifests, but its changelog entry was omitted at the time.

### Added

- **Public-rollout Phase D — legal docs** — `docs/SUB_PROCESSORS.md` (transitive processor index for GDPR Art. 28 DPAs), `docs/DPIA-AI-Processing.md` (AI Data Protection Impact Assessment across the model providers), and `docs/SOLICITOR_BRIEF_TOS_REVIEW.md` (one-shot brief for external Terms-of-Service review before public launch). All three registered in the Documentation Registry under the DPO persona.
- **Public-rollout Phase E — private-beta invite gate** — feature flag + invite-code table check on `/api/auth/register`, with an admin-only bypass; the gate releases on the go-live signal.
- **Admin System Health tab (`SystemHealthTab`)** — three panels: Live Health (consumes `/api/health/detailed`), Secrets Inventory (read-only Azure Key Vault secret names + last-modified, never values), and Logs (recent structured log events). New E2E spec `frontend/e2e/admin-system-health.spec.js`.
- **Tier-expiry cron endpoint (CV-241)** — `POST /api/admin/cron/run-tier-expiry`, shared-secret Bearer auth via `require_cron_secret` (`hmac.compare_digest` against `CRON_SECRET`). Wrapped in `register_public()` so `AuthMiddleware` doesn't pre-empt the dependency in prod (LESSON-091). Returns `{demoted: N}`; 503 when `CRON_SECRET` is unset, 401 on a missing/wrong-scheme/invalid token. Replaces the direct-DB script that broke when Postgres moved behind the VNet private endpoint. Daily cron at 03:00 UTC (`.github/workflows/tier-expiry.yml`).
- **Mobile — iOS TestFlight Build 16** — full Apple credential chain: Team `TT8K4YT4TQ`, manual signing with Distribution profile `UMAHD524CU` + certificate `8W7UH8F2LS`, `ExportOptions.plist` locked to `signingStyle: manual`, `ITSAppUsesNonExemptEncryption=NO` for auto-pass export compliance, Xcode 26+ runner. `APPLE_*` + APNs env vars wired into `container-apps.bicep` + the staging Bicep. `CapacitorHttp` plugin enabled to bypass WKWebView fetch quirks.
- **Push notifications (CV-343-2)** — foreground toast component + tap deep-link routing in the iOS and Android wrappers (the `device_tokens` table already existed from CV-331a — no new migration).
- **Three Japanese-bank career boards** added to `COMPANY_BOARDS_SEED` (226 → 229): Mizuho EMEA (SuccessFactors), MUFG (Workday tenant `mufgub`), SMBC EMEA (SuccessFactors EU) — each verified via a live registry probe.

### Changed

- **Cross-app header standardisation** — all top-level tabs render their title/subtitle/AI-pill through the canonical `<PageHeader>` component, pinned by a registry-driven contract test (22 tests across 11 standardised + 9 exempt tabs).
- **AI routing chip fix** — `ActiveModelIndicator` now resolves the per-group model from `aiPriority.resolved_models[providerId][group]` (mirrors `AIGateway._resolve_quality_override` at call-time) instead of showing the provider's configured default while a different quality-preset model actually ran. LESSON-038 family — single source of truth via `ai_meta`. Google AI timeout bumped for `gemini-2.5-flash-lite` 95th-percentile latency.
- **Backend version surface** — `_APP_VERSION` reads from `frontend/package.json` (now 0.8.0); exposed at `GET /` and `GET /api/health`.
- **Empty env-var crash class** — `int(os.environ.get(NAME, "DEFAULT"))` migrated to the `or` form across 22 sites, with a CI contract scanning for the vulnerable form.

### Fixed

- **Schema-drift recogniser hardening (LESSON-090)** — multi-line `Column(...)` declarations no longer break column extraction (balanced-paren matching); the idempotency-guard recogniser now accepts `dialect.name`, `IF NOT EXISTS`, and `inspector.has_table()`.
- **CI ACR login resilience** — `az acr login` wrapped in a 3-attempt retry with linear backoff (5s, 10s) to survive transient TLS resets, with fail-fast preserved for real auth errors.
- **Visual-regression mobile route-storm resilience** — the `cv-analysis-refresh-prompt` baseline reload-and-retries once when the prompt is missing, rescuing from WebKit's parallel-fetch burst on `initializeApp()` overflowing Playwright's interception queue.

### Internal

- 57 commits since 0.7.0. Coverage closed for LESSON-091 (the `AUTH_ENABLED=false` test env had masked `AuthMiddleware` gating of shared-secret admin endpoints in prod) — 3 new prod-middleware regression tests. Operator action pending at release time: set `CRON_SECRET` in GitHub Actions + Azure Key Vault.

## [0.7.0] - 2026-05-17

### Added

- **Phase 1 of iterative CV authoring (ADR-008)** — every legacy generation now lands in a persistent draft DAG with version history, fabrication metadata, drift-score, and provider attribution. Nine cards from CV-318 through CV-326:
  - **CV-318** — `cv_drafts` table (migration 040), repository layer, serialiser-completeness contract, and ADR-008 written.
  - **CV-319** — REST surface: `GET /api/cv-drafts?job_id=…`, `GET /api/cv-drafts/{id}`, `POST /api/cv-drafts`, `POST /api/cv-drafts/{id}/promote`. Pydantic v2 with `extra="forbid"` on inputs, `extra="allow"` on responses, `func.max() + 1` auto-versioning.
  - **CV-320** — fabrication + drift check on the new POST path. AI verdict persisted on the draft row; provider attribution from `ai_meta` (LESSON-038 pattern).
  - **CV-321** — non-blocking adapter-write on `/api/apply-single` + `/api/documents/{id}/update-cv`. Linear lineage, auto-promote. Failures swallowed with `event_type=draft_dual_write_failed` log line; user response never breaks.
  - **CV-322** — `<DraftTimeline>` component in DocumentCentre. Vertical card list with version, provider, ATS score, drift, fabrication verdict, Promote button. Lazy-loads `<DraftCompare>` only on first compare click.
  - **CV-323** — adapter-write extended to `/api/alternative` + `/api/multi-generate`. Sibling style: `parent_draft_id` set, `auto_promote=False` — preserves the user's original "current" draft when they generate alternatives.
  - **CV-324** — `<DraftCompare>` picker + extracted DiffView. Word-level diff between any two drafts or any draft vs base CV. Smart defaults: Left=Base CV (if available), Right=current. Swap button, same-selection guard, overlay-click-to-close.
  - **CV-325** — fabrication retrofit on the legacy adapter-write path. `CV_FABRICATION_STRICT` env var (default off): in observation mode, verdict is persisted but never blocks; in strict mode, high-risk verdicts skip persistence and emit `event_type=draft_blocked_high_fabrication`. Filesystem write of the CV/cover-letter is unaffected either way. Sentinels in the verdict (`no_base_cv`, `check_raised`, `check_malformed`) let the UI label outcomes explicitly.
  - **CV-326** — Phase 1 closeout: 7-row test coverage matrix (`docs/cv-drafts-coverage-matrix.md`) per QAE rubric, plus Playwright lifecycle E2E spec (`frontend/e2e/cv-drafts-lifecycle.spec.js`).
  Aggregate: 75 backend pass / 1 skip · 22 frontend component+contract · 1 lifecycle E2E.

- **Native authentication** — App Store and Play Store prerequisites:
  - **CV-329 — Sign in with Apple** (`/api/auth/apple`). Verifies Apple `id_token` and mints a CViper JWT.
  - **CV-330 — Native Google sign-in** (`/api/auth/google`). Verifies Google `id_token` and mints a CViper JWT. `oauth_native` module shipped with deploy chain + tests.

- **Mobile native rollout — Capacitor scaffolding** (CV-327, CV-332). iOS + Android wrappers, gitignore hygiene, native build CI workflows, JDK 21 / Node 22 bump for Capacitor 8. Universal links wired for iOS Universal Links + Android App Links. iOS Xcode scaffold restored after CI workflow consolidation.

- **Push notifications backend foundation** (CV-331a). `device_tokens` table + `POST /api/devices/register`, `GET /api/devices`, `DELETE /api/devices/{id}` endpoints. APNs + FCM sender service (CV-348) tracked separately.

- **Landing FAQ refresh** (commit `d39fe7dd`). Replaced the two engineering-flavoured items ("Which job boards are supported", "Do you fetch from company career pages") with the two trust signals a CV-tool visitor actually scans for: pricing (free + no card) and AI fabrication (no invention). Privacy answer expanded to cover both no-sell AND no-recruiter/employer-visibility reassurances. New forbid-list content contract (`LoginScreen.landingFaqContent.contract.test.jsx`) pins topic + load-bearing signals without freezing wording.

### Changed

- **GDPR — `cv_text` TTL + Pattern A/B/C wiring across 5 features** (CV-328). 7-day idle expiry on `cv_text` with profile preserved; Refresh CV prompt drives re-upload; Pattern A (full content), Pattern B (degraded with placeholder), Pattern C (block + prompt) wired across the 5 features that depend on `cv_text`. Privacy policy refreshed to v1.2 to reflect the TTL + CV-329 hygiene sweep.

- **Native JSON refresh tokens** (CV-328 follow-up). Refresh-token issuance gated by `X-Client-Type` header: cookie for web, JSON body for native clients. Capacitor cannot read `HttpOnly` cookies cross-origin, so the JSON path is required for Sign-in-with-Apple/Google flows on iOS + Android.

- **Login landing — helper button → FAQ spacing** (commit `d39fe7dd`). The action panel's flex container uses `gap: 32px` between every zone, which produced ~65px of dead vertical space between the "Not sure where to start?" helper CTA and the FAQ block — enough to force a scrollbar at first paint on common 768–820px laptop viewports. HR `margin: 0 → -16px 0` eats 32px of that gap. Visible span drops to ~33px; HR remains visible to mark the purpose change.

### Fixed

- **`DraftTimeline.jsx:338` used bare `toLocaleString()`** (commit `3897037d`). Anti-pattern flagged by the surfaces contract test on push; swapped to `formatDateTime()` from `utils/formatters.js` for consistent locale-aware formatting across pages.

- **Auth event log redactions** (CV-354 prep work). Username field redacted from structured `auth_*` event_type log lines — only `user_id` retained. Stops PII bleeding into observability stacks.

- **Drift banner for stale profile `cv_snapshot`** (commit `f9867ca1`, ADR-007). Saved-search profile snapshot was being auto-loaded over the latest CV analysis on tab switch — Search results scored against a stale CV. Now: latest analysis wins; stale snapshot surfaces a drift banner instead.

### Internal

- **Stats refresh** — Backend tests 364→365 files, Frontend tests 212→215 files, Frontend components 84→86 (DraftTimeline, DraftCompare). Architecture SVG, Testing-Strategy-and-Architecture.md, stats.json, STATS.md all in sync.

- **CHANGELOG / VERSION / BRD / FSD / TSA** archived at v0.6.6 and stamped at v0.7.0.

## [0.6.6] - 2026-05-13

### Fixed

- **P1 regression — AI model attribution mismatch** (commits `b608d09e`, `ba863ed4`, `f18b4b03`). Personal-key users changing a provider's model in Settings kept seeing the OLD model attributed to freshly-generated CVs/cover letters for up to 5 minutes — chip on My Applications said `Google Gemini · gemini-3.1-pro-preview` while Document Centre showed `Gemini · gemini-2.5-flash-lite`. Root cause: `UserKeyResolver._cache` snapshotted the model field at build time alongside the SDK client object, with no eviction on model change. Sister to CV-252. Three-layer fix: (1) `_on_model_change` callback fires from `ProviderRegistry.set_provider_model`, AIService wires it to invalidate the resolver cache per provider — covers PUT, autoheal, and multi-replica hydrate paths uniformly (LESSON-087, CLAUDE.md rule #60); (2) structural — `resolve_client` overlays the live model from `_resolve_model_for` on every call, making the bug class impossible even if layer 1 is unwired or bypassed; (3) cleanup — folded autoheal's duplicated inline invalidation into the hook (single writer, single invalidator). 8 new regression tests across `test_provider_model_resolver_cache_invalidation.py` (behavioural + idempotent + source-scan + structural overlay).

### Changed

- **Observability stack disabled** (commit `158760c6`, CV-272). Grafana / Loki / Prometheus container apps scaled to zero replicas. Azure Monitor + Log Analytics workbook scaffolded as the replacement path (commits `3be49b1e`, `4c03dae9`). Bicep `maxReplicas` constraint enforced at >=1 (`f78b392a`) since ARM API rejects 0.

- **Mobile fixes — 4 waves**. Wave 1.4–1.7 user-reported screenshot bugs (commits `960e05d9`, `0f5f050e`, `1fcc5a39`, `f4098df8`, `0d57c773`): stacked Search row panels on mobile (#5), 4 pre-existing 375px bugs found in proactive sweep, 5 + 6 + 7 user-reported regressions across density/chips/responsive layout. iOS Safari quirks documentation registered (`ae030a95`).

### Added

- **Resend SMTP outbound email wired into backend Container App** (commit `9298dd86`).
- **Visual regression baselines at 390×844** for mobile flows (commit `0012b50b`).
- **PR template — 375px screenshot requirement for frontend changes** (commit `5c733c1f`).
- **E2E — flow-aware post-action CTA assertions + bounding-box overlap detector at 375px** (commits `ff04fb5d`, `f004e34b`).
- **Pre-commit hook — block non-main commits by default** (commit `8ad6a64b`).

### Internal

- Self-hosted GitHub Actions runner runbook (`52c0e190`, `f592b30f`).
- Login CTA copy refinement (`97cb463d`).

## [0.6.5] - 2026-05-07

### Changed

- **Market Briefing rework** (commits `85c26c33`, `977b652c`, `37cc39ae`). Page renamed `Industry News Feed` → `Market Briefing` (no news, no dates, no sources — it's a personalised AI briefing, the new title matches what the page actually delivers). Tab `id` stays `newsfeed` so deep links and TopNav routing keep working. Three transparency improvements after the user2 review on 2026-05-07:
  1. **Disclose the location driving the analysis.** Backend was already passing `profile.location` into the AI prompt but it was invisible in the UI. Response payload now carries `location` + `weeks_analysed` (cached + fresh paths); the page renders `Based on jobs in <location> · change` directly under the header. Empty location renders nothing — no silent default, no "Based on jobs in undefined".
  2. **Lead with grounded data, not AI inference.** The "Trending Skills (from your searches)" block — the only section backed by real signal (8 weeks of scraped postings via `repo.get_trending_skills`) — was the smallest and last item. Promoted to directly under Market Overview, renamed `What your searches show` with a "last 8 weeks of searches" subtitle.
  3. **Stop fabricating precision.** Skill Demand cards rendered a 6px bar at hardcoded width 85% / 55% / 25% based on the qualitative bucket — looked like a measurement, wasn't one. Bar removed; replaced with a clean colour-coded chip. Section renamed `Your Skill Demand` with a `?` methodology tooltip and a one-line subtitle that honestly describes the AI inference.
  
  **Industry Trends + Recommendations merged into one `What to do next` section.** Pre-fix the two duplicated each other (a trend's `action` field IS a recommendation), shipped as flat numbered lists with no priority, and gave the user no way to act. The merged section keeps both data sources and adds keyword-routed action buttons that navigate to the relevant tab: `skill`/`learn`/`training`/`course`/`upskill`/`study` → Skills & Training; `cv`/`resume`/`portfolio`/`tailor` → CV Analysis; `apply`/`job`/`search`/`role`/`position`/`posting` → Job Search.
  
  **AI prompt strengthened.** Two new clauses in `build_industry_briefing_prompt`: (a) "anchor in supplied data" — for any skill that ALSO appears in the trending data, demand/trend MUST align with the user's quantitative signal; the user's own job-search data is ground truth. (b) "name the location" — AI must mention the candidate's location string by name in `market_overview` at least once. Both clauses are conditional (anchor only when trends_block non-empty, location only when location non-empty).

- **Domain rebrand to cviper.ai** (canonical) — code and docs migrated from `cviper.uk` → `cviper.ai`. `cviper.uk` retained in CORS allowlists and Azure custom-domain bindings for the transition window so existing sessions continue to work; Cloudflare 301 will redirect `.uk` → `.ai` once Azure cert is healthy. OAuth callback URIs (Google + Microsoft + LinkedIn) switched to `.ai`; old `.uk` callbacks must remain registered in the OAuth consoles for the transition. CSP `connect-src` allows both `api.cviper.ai` and `api.cviper.uk`. Status page (Cloudflare Worker), Grafana iframe-embed comments, sales/marketing copy, support emails (`support@`, `security@`, `privacy@`), legal docs (Privacy Policy + Terms of Service), and Plausible analytics domain all updated. `test_deploy_config.sh` now asserts the `.ai` hostnames are bound (legacy `.uk` becomes a soft warn so we know if it's accidentally unbound). PDF footer on every generated CV/cover letter now reads "Generated by CViper (cviper.ai)". See `ClaudeReports/changes/2026-05-07-change-domain-rebrand-cviper-ai.md` for the cutover runbook (registrar / DNS / Azure / OAuth consoles / email DKIM / 301 redirect).

### Added

- **Skills & Training: self-rated proficiency dots** + UCA/UXR polish (commit `528c1415`).

- **TopNav username pill** (commit `eec2206b`) — username + PRO badge now wrapped in a single rounded pill container. Internal flex + `line-height: 1` puts both children on the same axis by construction; eliminates the cross-browser font-rendering off-centre that 3 prior pixel-tuning iterations couldn't fix.

### Fixed

- **Backend test response-shape contract** — 5 new tests in `test_news_feed.py` covering the `location` + `weeks_analysed` keys (cached + fresh paths, empty-string boundary case) and the two new prompt clauses (anchoring + location naming). 14 → 16 prompt-class tests.

- **Frontend rendering contract** — 3 new tests in `NewsFeedTab.test.jsx` for the merged `What to do next` section, action-button presence, and keyword→tab navigation. Old "renders industry trends" / "renders recommendations" tests replaced by the merge contract test that explicitly forbids the old headings (forbid-list pattern, LESSON-033 family).

### Stats

- Backend tests: +5 (response-shape and prompt assertions)
- Frontend tests: +3 (merged-section contract, action-button keyword routing)

## [0.6.4] - 2026-05-07

### Changed

- **Cross-app header consistency — single source of truth** (commit `f1882618`). Every standardised top-level tab (CV Analysis, Job Search, Applications, Settings, Admin, Companies, Career Insights, Skills, News Feed, FAQ, My Requests — 11 in total) now renders its title / subtitle / AI-pill via a single `<PageHeader>` component at [`frontend/src/components/PageHeader.jsx`](frontend/src/components/PageHeader.jsx). Replaces ~10-25 lines of hand-rolled JSX per tab with one element. Drift becomes structurally impossible because the rendering is owned by one component. Pattern parallels Rule #57 (showcase SVGs are JSON-generated) and Rule #56 (`_*_to_dict` serialiser contracts) — structural fix + forbid-list contract.

- **Insights tabs cleanup completed** (commits `edd2cfd2`, `3bfc6040`, `f7a3963b`, `0dccfdb3`, `a6c92f17`).
  - Companies & Salary Estimates: AI provider attribution standardised on the slim `Powered by …` pill (replacing the legacy fat radio-tile picker — last surviving caller of that legacy component, now removed); UK Regional Salary Comparison normalised to standard `.card` markup (NOT AI-powered, no AI star added); filter selects + bulk actions merged into the Estimates Table card-header (was a floating sibling card); Market Benchmarks moved to the bottom of the page (reference data, hoisted via `marketBenchmarksBlock` const).
  - Career Insights / Skills / News Feed: standard `<PageHeader>` pattern. News Feed compact gradient strip removed; Refresh button absolute-positioned top-right (`rightSlot` prop); generated-at footer caption restored.
  - Skills hero is dismissible (localStorage `cviper_skills_hero_dismissed`) with a "What's this?" inline link in the subtitle to restore. Reclaims ~250px above the fold for returning users.
  - FAQ + My Requests headers standardised.

- **Mobile experience cleanup** (commits `a5e1effc`, `9e12aa81`, `e79a2baf`). StatusBar pinned to bottom on mobile (above mobile-bottom-nav with `safe-area-inset-bottom`); Search action buttons inline side-by-side; Search Source Results detail card default-collapsed; Help & Feedback FAB visually recedes (0.7 opacity, full on hover/focus/tap); tighter mobile card padding; redundant "search complete" success toast removed (sticky in-page banner already conveys this); duplicate "Search LinkedIn" header button removed.

- **Job Search structural overhaul** (commits `4f530bf3`, `6558fdff`). Search Criteria card split into three structurally honest cards: Search Criteria (location, salary, titles, filters that drive the search), Match Scoring (CV Skills used to rank returned jobs, with explicit "does not affect what's searched" sub-line), Job Sources (boards + direct employers). Asymmetric default behaviour between Job Titles (opt-in) and CV Skills (opt-out) is now self-documenting via purpose-lines. Health-dot legend hoisted to Job Sources level so a single key serves both grids.

- **Direct Employers parity fixes** (commits `80c65d6d`, `2cf0f1ab`). Favourites now sort to the top of the grid (mirrors Job Boards behaviour); industry-dropdown count matches the rendered grid for any combination of filter flags (single-source-of-truth predicate in `companyBoardVisibility.js` + 15-permutation parity contract).

- **Readiness badge labels** (commit `2e3684d8`). Apply Now → Strong Match, Tailor CV → Needs Tailoring, Upskill → Skills Gap. Purely descriptive adjective phrases; no longer reads as a clickable button.

### Fixed

- **Empty env-var crash class — 22 sites + CI contract** (commit `6921a085`). The CV-241 nightly tier-expiry cron failed for 5 nights because `int(os.environ.get("SMTP_PORT", "587"))` returned `int("")` → ValueError. The second-arg default fires only when the key is ABSENT from the environment, not when it's present-but-empty — exactly what GitHub Actions / Azure / docker-compose do for unset secrets. email_service.py was patched at the time of the RCA, but the RCA's "Open items" listed 15+ sibling sites with the same anti-pattern. All 22 vulnerable sites now use the safe `or` form (`int(os.environ.get(NAME) or "DEFAULT")`). New `backend/tests/infrastructure/test_env_var_default_pattern.py` is a forbid-list contract scanning the production source tree; the contract caught 2 sites in `monitoring.py` (`float(os.environ.get(...))` calls) that the RCA's hand-listed audit missed.

  **Out of scope**: the cron is still failing in production due to Issue #479 (Azure PG firewall blocks GitHub Actions runner IPs). That fix requires Azure CLI / Bicep / self-hosted runner — outside code-only PR scope.

- **`docs/version-docs.sh` bug-fixes** (commit `a1dd2d8c`). Two bugs preventing the script from producing a correct VERSION.md:
  1. Version-to-Commit map was wiped on every run because `cat > "$VERSION_FILE" <<EOF` truncated the file before the inline `$(grep ... "$VERSION_FILE")` command-substitution ran. Fix: read the existing map into a shell variable BEFORE the heredoc.
  2. `App Version: unknown` because `node -p require('$ROOT_DIR/package.json').version` looked at the repo root, but `package.json` lives at `frontend/package.json`. Fix: corrected path + three-tier fallback (`node` → `python` → `grep`) so the manifest stamps a real version regardless of which tools are on PATH.

### Added

- **`<PageHeader>` component + 13 unit tests** + the registry-driven consistency contract (22 tests). Documented exemption set covers the 9 legitimate special-cases (About landing page, Privacy/Terms via MarkdownLegal, public Status page, Monitoring multi-panel, Prompts Lab admin tool, AdminDatabase + FeedbackAdmin nested sub-components).

- **Companies tab utilities + tests** (commits `80c65d6d`, `2cf0f1ab`). `companyBoardSort.js` (`favouritesFirstByCompany` comparator + 6 unit tests) and `companyBoardVisibility.js` (visibility predicate + 32 tests including a 15-permutation parity matrix). Single source of truth shared between `App.jsx` and `SearchForm.jsx` industry-dropdown count.

- **GitHub Issue #501** opened to track the favourites-sort and dropdown-count fixes (post-hoc — Issue created after the commits landed because branch protection blocked the amend). Linkage lives issue-side; commit messages don't carry `Closes #501` for this pair.

### Removed

- **Legacy `<AIProviderCard>` and `<AIMultiProviderCard>` components** (commit `8835f0b4`). Last call site (CompaniesTab) migrated to the slim `<ActiveModelIndicator>` pill. Removed 4 files (component + test for each), 379 lines deleted.

### Stats

- Components: 75 → 76 (+1 `PageHeader`)
- Backend test files: 342 → 343 (+1 env-var contract)
- Frontend test files: 199 → 200 (+1 PageHeader test, +1 page-header-consistency contract; some prior commits also rolled in)

## [0.6.3] - 2026-05-07

### Changed

- **Job Search Search Criteria card split into three structurally honest cards** (commits `4f530bf3` + `6558fdff`). The single "Search Criteria" card was housing two structurally different inputs: ones that drive the actual search (Location, Salary, Job Titles, filters) and ones that score returned jobs (CV Skills) — same card, different jobs, with opposite default behaviour (titles opt-in, skills opt-out) and no on-page explanation of why. Now three sibling cards: **Search Criteria** (location / salary / titles / filters), **Match Scoring** (CV Skills only, with explicit "Tunes how returned jobs are ranked — does not affect what's searched" sub-line in the card header), and **Job Sources** (boards + direct employers, unchanged). Purpose-lines under each input label spell out the asymmetric defaults — Job Titles "Sent as search queries — pick the titles you want, or click Use all to add every CV suggestion"; CV Skills "Used to score how well returned jobs match your CV — all skills count automatically. Remove any that aren't relevant to this search."

- **Health-dot legend hoisted to Job Sources level** (commit `30cc2542`, contract follow-up `c8746d09`). The working/fragile/broken/unchecked legend lived inside the Direct Employers card body — invisible to anyone scanning Job Boards above, even though both grids use the same `<HealthDot>` vocabulary. Hoisted to a single thin strip at the top of the expanded `{showJobSources && <>}` block; one key now serves both grids. The strip uses `var(--neutral-50)` background (no hex fallback per `surfaces.contract.test.js`).

- **Industry News Feed compact strip header** (commit `2960ffb6`). Replaced the ~170px centred hero with a 60px single-row strip (icon + title + subtitle on the left, stat chips in the middle, Refresh on the right). Chips are sourced from existing briefing payload — `briefing.industry_trends.length` for trends, `briefing.skill_demand.length` for skills, `briefing.generated_at` for last-refresh time, plus a "Cached" italic chip when `data.cached` is set. Bottom "Generated …" footer removed (now redundant — same data in the strip).

- **Readiness badge labels rewritten as descriptive adjectives** (commit `2e3684d8`, file `frontend/src/components/FitScoreBadge.jsx:36-45`). The green pill on the FIT-score column read as a clickable button because all three labels were imperative verbs: `Apply Now` / `Tailor CV` / `Upskill`. Renamed to `Strong Match` / `Needs Tailoring` / `Skills Gap` — purely descriptive, parallel adjective phrases that mirror the existing **Strong / Fair / Weak** score-tier vocabulary already in the cell. New `data-testid="readiness-badge"` and `readiness-badge-lg` attributes so the viewport-clipping E2E spec selects via stable testid (LESSON-059) instead of the visible label regex.

- **Direct Employers favourites-first sort + 6 unit tests** (commit `80c65d6d`, files `frontend/src/utils/companyBoardSort.js` + `companyBoardSort.test.js`). The grid sorted purely alphabetically, so favourited employers (Abound, Aldermore, Bank of America, Barclays, BlackRock, Chase UK, Citi …) were scattered through the list — Job Boards above already showed favourites in their own row, and the on-page tip ("favourites appear first on every visit") was only true for one of the two grids. Extracted `favouritesFirstByCompany` into a pure function so its contract is unit-testable; App.jsx's `displayedBoards` useMemo calls it after applying the visibility filters.

- **Industry-dropdown count parity contract** (commit `2cf0f1ab`, files `frontend/src/utils/companyBoardVisibility.js` + `companyBoardVisibility.test.js`). The dropdown showed e.g. "Gaming (4)" while only 3 cards rendered when Gaming was selected. Two parallel filter pipelines had drifted: the dropdown counted against unfiltered `companyBoards` while the grid applied additional filters (experimental boards hidden unless enabled, broken boards hidden unless enabled, favourites-only mode). Single source of truth + 15-permutation parity contract: `isVisibleIgnoringIndustry` / `isVisible` / `applyVisibilityFilters` / `countVisibleInIndustry` exported from one module; 32 unit tests including a parity matrix that asserts `countVisibleInIndustry(boards, industry, flags) === applyVisibilityFilters(boards, {...flags, industryFilter: industry}).length` for every combination of 5 filter combos × 3 industries. Same architectural family as LESSONs 035/052 (single source of truth for state shared across surfaces) at the in-component filter-pipeline surface. CI fails if a future commit hand-rolls a different counter at either call site.

- **Mobile experience cleanup sweep** (commits `a5e1effc` + `9e12aa81`). Six coordinated changes targeting "feels cluttered and difficult to navigate" feedback at viewport ≤768px:
  - **StatusBar pinned to bottom on mobile** (above `.mobile-bottom-nav` with `env(safe-area-inset-bottom)`) — was top-pinned, competed with content for scroll space. Border-and-shadow direction inverted; `--status-bar-height` switches from `padding-top` to `padding-bottom` on `.main-container` so the last result card never hides under the bar. Z-index 1999 sits below the nav (2000) so the nav still wins on overlap.
  - **Search + LinkedIn buttons inline on mobile** (was stacked) — `flex: 2 1 0` (Search) vs `1 1 0` (LinkedIn) preserves primary/secondary hierarchy. The long LinkedIn caption ("Opens LinkedIn in a new tab (sign-in required)…") is hidden on mobile (the same info lives in the button's `title` tooltip). ~80px reclaimed above the fold.
  - **Search Source Results card default-collapsed on mobile** — header + completion badge stay visible; tap to expand. `useEffect` syncs collapsed state to viewport so rotating to landscape opens it automatically.
  - **Help & Feedback FAB visually recedes** — opacity 0.7 default, 1.0 on hover/focus/tap. Tap-target stays 44×44 (iOS guideline minimum).
  - **Tighter mobile card padding** — new `@media (max-width: 768px)` block in `theme.css` reduces `.card` margin-bottom 16→12px, `.card-header`/`.card-body` padding 12×16→10×14, `.card-header h3` font-size 14→13px. Applies broadly with no restructuring; reclaims ~5-10% vertical on every card.
  - **Redundant "search complete" success toast removed** — the sticky in-page banner already shows "Search complete · N/M sources · K jobs found" with the same data; the toast was duplicate signal floating over the banner. Warning toasts (zero-results / errors) are preserved because they need eye-catching prominence. 2 useSearch.test.js positive assertions converted to forbid-list ("no success-tone toast fires") — LESSON-033 family.

- **Duplicate "Search LinkedIn" button removed from Search Criteria header** (commit `e79a2baf`). The header carried a green-bordered LinkedIn quick-action that duplicated the bottom action-bar "Search on LinkedIn" CTA — same target, same behaviour, different visual weight. The 2026-04-21 reason for adding it ("bottom button got lost beneath the 210-item Direct Employers grid") is now mitigated by the Direct Employers card collapsing by default and the mobile cleanup compressing the form ~150px.

### Fixed

- **User-scoped Config reads silently used per-replica cache** (LESSON-080 / Rule #55, commit `ccc1abaf`). Azure Container Apps runs the backend with `minReplicas: 1, maxReplicas: 3`. The `_TTLCache(ttl=30)` in `backend/deps.py:load_config()` was per-process, so a user saving a Direct Employer selection on replica A would see the change reverted after the post-login GET load-balanced to replica B whose cache held the pre-save empty state. Plus the POST handler did read-modify-write across two sessions with no row lock, racing against concurrent toggles. Architectural fix: `load_config()` caches ONLY when `user_id is None` (global state); user-scoped reads bypass the cache and hit the DB directly. The POST handler opens a single managed session, calls `repo.get_config(session, name, user_id, for_update=True)`, mutates, and writes via `repo.save_config_db(session, name, data, user_id)` in the same session. 6 regression tests including a multi-replica drift simulation (prime cache → write via repo bypassing invalidation → assert GET reads fresh) + source-scan contract that the POST handler calls `get_config(..., for_update=True)`.

- **`_user_to_dict` dropped fields exposed by `/api/auth/me`** (CV-287 / Rule #56, commits `621eecae` + `22b0d55c`). The Pro/Free Plan column on the Admin Users tab read from `/api/admin/users` which returned a hand-built dict missing `tier`, `is_sandbox`, and 5 other columns from the model. Pro users showed as Free; sandbox-row Set Plan picker was confused. Fix all four bug instances in one session, then the structural fix: registry-driven contract test in `backend/tests/infrastructure/test_serialiser_completeness.py` AST-walks every `_*_to_dict` helper in `repositories.py` and asserts each helper exposes every column on its model minus an explicit per-helper exemption set. Generic helpers that iterate `Model.__table__.columns` directly are immune by construction.

- **Salary-estimate route 500 on cross-user slug collision** (CV-289, commit `a14314a7`). When user B saved a salary estimate for a company slug that user A had already used, a UNIQUE constraint violation surfaced as a 500. Fix: route auto-renames the slug on cross-user collision; `save_company` filters its existing-row check by `user_id` so it can't shadow another user's row.

- **Job Search column-width drift clipped expansion right-side badges** (Rule #54, commit `ecd449e7`). The Search results expansion's "Strong Match" / "Career Change" / "Well Matched" badges were clipped off the right edge of the viewport because [`SearchResultsList.jsx`](frontend/src/tabs/search/SearchResultsList.jsx) declared column widths via `maxWidth` on `<td>` while [`ApplicationsTab.jsx`](frontend/src/tabs/ApplicationsTab.jsx) (where the same expansion works correctly) declared `width:` on `<th>`. Two tables that originally shipped parallel had quietly drifted under feature pressure. Fix: align the canonical width-on-`<th>` pattern; new Vitest parity contract + Playwright viewport-fit spec.

- **News Feed Industry chip silently dropped for legacy users** (LESSON-063, prior to v0.6.3 — included in v0.6.2 baseline but called out here as recurring in cycle audits). The seed-merge helper `merge_with_seed()` let user rows entirely replace seed rows on dedup, so the seed's `industry: "Banking"` was silently dropped for every legacy user. Auto-discovery contract test picks up new seed fields automatically.

### Architecture

- **Diagram pipeline overhaul** (LESSON-081 / Rule #57, commit `84101952` + follow-ups). The 6 showcase SVGs (`frontend/public/showcase/*.svg`) are now generated from JSON specs in `docs/diagrams/*.json` by `scripts/diagram_gen.py`. Direct SVG edits fail CI. The defense layers — generator (Layer 1), unit tests (Layer 2), output contract for orthogonality + text-bounds + intersection-free (Layer 3), stats interpolation (Layer 4), Playwright visual regression (Layer 5), nightly + PR-gated CI workflow (Layers 6+7) — only hold if humans don't bypass the JSON layer. Two production bugs in the live SVGs were caught by the generator's intersection-detection during the migration and refused to ship: the `is_sandbox` arrow in `auth-rbac-flow.svg` going through the 401 Interceptor box, and the AI Service → AI Providers arrow in `system-architecture.svg` going through the Blob Storage and Managed Identity boxes. Bug class "lines through boxes / missing data / lines not aligned" had recurred so often that rules #12, #16, and #29 each tightened the SVG-edit path without ever closing it.

- **Cache-buster chain contract** (LESSON-082 / Rule #58, commit `ee0e3657`). Showcase SVGs are versioned via a `?v=<git-sha>` query string baked at build time. New contract test source-scans every `<img src=` referencing `/showcase/*.svg` to assert the cache-buster suffix is present and references `import.meta.env.VITE_GIT_COMMIT`. Companion fix in `frontend/Dockerfile.azure` to receive the `GIT_COMMIT` build-arg and propagate it to Vite's environment so the cache-buster is real, not the literal string `undefined`.

### Added

- **Manual Search Boards panel** (commit `c777b82e`). New UI for boards that don't support automated scraping — admins/users can paste a search URL and CViper opens it in a new tab with the user's keywords pre-filled. Paired with a backend `manual_search_boards` module and an `infra(hooks)` untracked-import guard.

- **WebKit + Mobile Safari cross-browser E2E** (commit `8728d5e3`). New Playwright projects added to the cross-browser matrix on a tagged-subset only (the full suite stays on chromium for speed). Several existing specs were either skipped on webkit (auth/sandbox-mocking) or refactored to scope `/skills/i` locators to the content area (avoiding hidden-nav matches).

- **About page Financial Services Context tag-cloud collapsed by default** (commit `126f40b4`). The dense tag cloud was burying the page's value props for first-time visitors; collapsed behind a "Show details" toggle.

### Stats

- Backend test files: 333 → 342 (+9)
- Frontend test files: 191 → 199 (+8)
- New auto-correction rules: #54 (table-expansion column-width parity), #55 (user-scoped Config cache bypass + RMW lock), #56 (serialiser completeness), #57 (showcase SVGs are generated), #58 (cache-buster chain)
- New utility modules: `companyBoardSort.js`, `companyBoardVisibility.js`

## [0.6.2] - 2026-04-30

### Fixed

- **Insights News Feed ignored the loaded CV** (LESSON-072, commits `7667fe32` + `f8dfac45`). `news_feed_briefing()` queried `database.SearchProfile` directly and silently degraded when a user had a CV uploaded but no saved-search profile — producing a generic "0 years experience, general tech roles" briefing for every freshly-onboarded user. Fix: fall back to `repo.get_latest_cv_analysis()` when no SearchProfile exists; lookup moved before the cache check so cache-hit responses also label `profile_used` correctly; skip persisting briefing when both `target_titles` and `target_skills` are empty (no more 4h-stale generic cache shadowing a later CV upload). Frontend nudge copy corrected: "Upload your CV in the **CV Analysis** tab" replaces the misleading "Create a search profile in the CV Analysis tab" (the action it directed users to didn't exist there). 4th-surface fix in the architectural family — LESSON-037 (search), LESSON-067 (3 document endpoints), LESSON-072 (news feed). Forbid-list contract added at [`backend/tests/infrastructure/test_searchprofile_direct_query.py`](backend/tests/infrastructure/test_searchprofile_direct_query.py) — AST function-scope scan that fails CI when a route queries `SearchProfile.user_id` without consulting a fallback helper in the same function body. 2 new endpoint tests at [`backend/tests/features/test_news_feed.py`](backend/tests/features/test_news_feed.py) (CvAnalysis-only fallback + empty-state boundary).

- **Direct Employers health check truncated at HTTP timeout** (LESSON-073, commit `e4332418`). For users with all 212 boards enabled, `health_check_boards()` ran a sequential for-loop over real HTTP requests (~5s × 212 = ~17 min, far past any reasonable HTTP timeout). The Job Boards section worked because it had only 11 boards. Tier 1: new `health_check_boards_async()` with `asyncio.gather` + `asyncio.Semaphore(10)` + `asyncio.wait_for(timeout=8s)` per board; 212 boards now complete in ~170s worst case. Per-board timeouts and exceptions surface as `health_status="broken"` with the error text rather than aborting the batch. Pure helper `_build_health_result()` shared by sync + async paths so status mapping can't drift. The HTTP route at [`backend/routes/companies.py`](backend/routes/companies.py) wraps the await in `try/finally` with an `on_result` callback so partial results persist even if the request is cancelled mid-flight. Tier 2: new `useEffect` in [`frontend/src/tabs/search/SearchForm.jsx`](frontend/src/tabs/search/SearchForm.jsx) auto-fires the health check on Search-page mount when at least one enabled jobSite or companyBoard has unset OR stale (>24h) `health_status` — single fire per mount via `useRef` guard, skipped for sandbox / empty / no-enabled. 8 new backend unit tests + 1 endpoint integration test (simulates `CancelledError` mid-flight to assert the partial-results-persist contract) + 8 new frontend unit tests. Drive-by: narrowed bare `except` in `career_search/job_site_health.py:90` to `(OSError, IOError)` to clear an unrelated silent-exception ratchet baseline.

- **Login footer Privacy/Terms/Status links bounced back to login** (LESSON-076, commits `166dedb4` + `74345ae5`). The auth gate at [`App.jsx`](frontend/src/App.jsx) had a single hardcoded bypass for `?tab=score`; every other footer link (`/?tab=privacy`, `/?tab=terms`, `/?tab=status`) caused a full-page reload that re-rendered the login screen. Status was broken at a second layer too — not in `EXEMPT_TABS`, no `StatusPage` component, no `activeTab` branch. Fix: new `PUBLIC_TABS` allowlist in the auth-gate code generalises the score bypass; new [`StatusPage.jsx`](frontend/src/components/StatusPage.jsx) component pulls live data from `/api/health/detailed` with a fallback to `/api/health` and uses raw `fetch()` not `authFetch()` so unauthenticated visitors don't trigger the global 401 interceptor; "Back to CViper" header on the unauthenticated path so visitors aren't trapped on a chrome-less page. `'status'` added to `EXEMPT_TABS`. Source-scan parity contract at [`frontend/src/components/LoginScreen.publicLinkParity.test.jsx`](frontend/src/components/LoginScreen.publicLinkParity.test.jsx) catches the bug class at PR time: any `<a href="/?tab=X">` in `LoginScreen.jsx` must point to a tab in the `PUBLIC_TABS` allowlist or fail CI. 5 StatusPage unit tests + 7 public-tab bypass contract tests + 5 footer-link parity tests = 17 new tests.

### Added

- **Multi-agent commit-safety infrastructure** (LESSON-074 + LESSON-077). Pre-commit `staged-set snapshot guard` in [`frontend/.husky/pre-commit`](frontend/.husky/pre-commit) detects when a parallel agent's `git commit` modifies the index mid-hook and blocks the commit (catches the wrong-content-under-wrong-message pattern hit 4× on 2026-04-28). Commit-msg `LESSON-number collision guard` flags a new commit referencing a LESSON ID already used by a different lesson in `memory/lessons_learned.md`.

**2026-04-28 AI key/model/tier security sweep** — 14 fixes shipped across two batches in one day. Batch 1 (8 fixes, CV-245..CV-255) closed P1 issues from a deep audit ([`ClaudeReports/audits/2026-04-28-audit-ai-key-model-tier-deep-dive.md`](ClaudeReports/audits/2026-04-28-audit-ai-key-model-tier-deep-dive.md)) that found ~22 issues at the edges of existing LESSON contracts (auth boundaries, transition cleanup, multi-replica state, encryption rotation). Batch 2 (6 fixes, CV-241 follow-up + CV-258..CV-263) closed P2/P3 follow-ups from the same audit plus the long-running CV-241 cron failure surfaced from session-start signals. 23 backlog cards opened (CV-245 → CV-267); 14 closed below. The remaining 9 (CV-250/251/253 P1 + CV-256/257/260 P2 + CV-264..CV-267 P3) stay open as future cycles.

- **CV-245 + CV-246 — rate limit on `POST /api/ai-keys/{id}/validate`; auth gate verified on `POST /api/ai-providers/{id}/test`** (commit `df7ebe8c`). Audit flagged both endpoints as "unauthenticated brute-force / quota-exhaustion oracles" (NEW-1, NEW-2). Re-verification under `AUTH_ENABLED=true` showed the AuthMiddleware already gates both paths — the agent's source-scan inspected route handlers in isolation and missed the middleware layer, inflating severity. Real residual gap: validate endpoint had no rate limit. Fix: `@limiter.limit("3/minute")` on `validate_ai_key`. With auth enforced, brute-force is now bounded to ~3 attempts/min per authenticated user, making a key-space sweep impractical even from a compromised low-tier account. 5-test regression guard at [`backend/tests/security/test_unauth_ai_endpoints.py`](backend/tests/security/test_unauth_ai_endpoints.py) — auth gate (forbid-list, LESSON-033) + rate-limit decorator presence (forbid-list).

- **CV-248 — `SELECT FOR UPDATE` on admin-key + personal-key save** (commit `d9aa0bcb`). `PUT /api/ai-keys/{provider}` read-modify-wrote the keys dict from Postgres without a row lock. Under default `READ COMMITTED`, two admins saving DIFFERENT providers in the same window raced and lost one update (audit NEW-3 — `T0`/A reads, `T1`/B reads, `T2`/A adds anthropic, `T3`/B adds google, `T4`/A saves, `T5`/B saves → anthropic update silently lost). Same race on the personal-key path (per-user `user_provider_keys` row, multi-tab edits). Fix: `for_update=True` kwarg on `repo.get_config` and `repo.get_admin_provider_keys` adds `.with_for_update()` to the SELECT; row locked from read until commit. SQLite no-op (test backend); PostgreSQL row lock enforced. 4-test forbid-list contract at [`backend/tests/security/test_admin_key_save_race.py`](backend/tests/security/test_admin_key_save_race.py) — compiled-SQL on PG dialect + source-scan on both paths + signature kwarg.

- **CV-247 — AuthMiddleware enforces `sandbox_expires_at`** (commit `cb3b48da`). JWT exp is up to 30 min (`SANDBOX_ACCESS_TOKEN_EXPIRE_MINUTES`) but `sandbox_expires_at` is a per-user DB field. `decode_access_token` validated only the JWT exp claim, so a sandbox trial could expire with up to 30 min of JWT lifetime remaining — the user kept hitting AI endpoints, falling back to admin's provider list (not the sandbox-restricted set), consuming admin's pooled quota for up to 30 min after their trial ended (audit NEW-7). `is_sandbox_user(user)` returned False for expired sandbox so the downstream sandbox-mode restrictions didn't fire and the user looked like a regular caller. Fix: AuthMiddleware now checks `user.is_sandbox` (raw flag, not the expiry-aware helper) and parses `user.sandbox_expires_at` defensively (malformed timestamps fail-open per the existing helper's pattern; naive timestamps normalised to UTC). Expired sandbox JWTs return 401 with `X-Auth-Status: sandbox-expired` — distinct from `session-expired` (per CLAUDE.md rule #18), so the client's global logout interceptor does NOT fire and the frontend can route the user to the upgrade screen, preserving demo state. 7-test regression guard at [`backend/tests/security/test_sandbox_jwt_expiry_middleware.py`](backend/tests/security/test_sandbox_jwt_expiry_middleware.py) covering happy / negative / boundary / edge-case (malformed timestamps).

- **CV-249 — row lock on sandbox FP counter to close 49→51 race** (commit `4bb53334`). `SandboxAbusePrevention` middleware read `get_sandbox_daily_usage(session, fp, date)` then called `increment_sandbox_usage(...)` separately. Two concurrent replicas raced past the limit (audit NEW-8 — both read 49/50, both allow, both UPDATE → counter ends at 51 with the user having gotten 51 ops through limit=50). Fix: `for_update=True` kwarg on `repo.get_sandbox_daily_usage` adds `SELECT ... FOR UPDATE` so the row is locked from read until commit; replica B's SELECT blocks at lock acquisition until A's UPDATE commits, then sees count=50 and rejects. Trade-off documented: lock works on EXISTING rows. The first request of the day for a fingerprint has nothing to lock; the unique constraint `uq_sandbox_fp_date` catches duplicate INSERTs at commit time. Overflow window is 1 request on the day's first concurrent burst — accepted vs the complexity of portable `INSERT...ON CONFLICT...RETURNING` across SQLite/PG. IP-level check intentionally not locked (its `SUM`-across-rows query can't be locked simply, and the dominant abuse vector is now serialised at the FP layer). 3-test forbid-list contract at [`backend/tests/security/test_sandbox_fp_counter_race.py`](backend/tests/security/test_sandbox_fp_counter_race.py).

- **CV-252 — invalidate `UserKeyResolver` client cache on personal-key PUT** (commit `3ce95a7b`). `PUT /api/ai-keys/{provider}?scope=personal` updated the DB row at `Config(name='user_provider_keys', user_id=...)` but the `UserKeyResolver` cached the BUILT CLIENT OBJECT for 300 seconds ([`backend/ai/user_keys.py:83`](backend/ai/user_keys.py#L83)). After a key change, the resolver kept returning the OLD client until the TTL expired — users experienced "my new key isn't being used" for up to 5 minutes (audit ORIG-1). LESSON-061's gateway hydrate hook does NOT close this gap — that hook reloads Config rows for admin-keys/model-prefs/routing, a different layer from the resolver's per-user client cache. Fix: after `save_config_db(name='user_provider_keys', ...)`, call `ai_service._gateway._resolver.invalidate_cache(user_id, provider_id)` per-user, per-provider (not a coarse global wipe). Defensive try/except — invalidation failure logs and proceeds (cache self-heals at TTL); the save itself is the load-bearing part. 3-test forbid-list contract at [`backend/tests/security/test_personal_key_resolver_cache_invalidation.py`](backend/tests/security/test_personal_key_resolver_cache_invalidation.py) — any save to `user_provider_keys` Config in `routes/cv_analysis.py` must be followed within 25 lines by an `invalidate_cache(user_id=...)` call.

- **CV-254 — `_effective_tier()` at all gating sites + forbid-list contract** (commit `9e86eabe`). CV-238's defence-in-depth `_effective_tier(user)` helper catches expired Pro at every quota gate by checking `tier_expires_at`. It was correctly called from `repositories.check_usage_limit`, `repositories.get_usage_limits`, and `ai/token_budget.py::_resolve_limits` — but two sites still read `user.tier` directly: [`backend/usage_middleware.py:128`](backend/usage_middleware.py#L128) (Pro bypass for per-op caps) and [`backend/routes/usage.py:85`](backend/routes/usage.py#L85) (`/api/usage` Pro bypass). A Pro user past `tier_expires_at` slipped through these gates until the nightly demotion job (CV-241) ran (audit ORIG-3). Fix: replace direct reads with `_effective_tier(user)`. Forbid-list contract test [`backend/tests/security/test_effective_tier_coverage.py`](backend/tests/security/test_effective_tier_coverage.py) AST-scans all backend files for `<obj>.tier` reads outside an explicit allowlist (helper definition, schema declaration, demotion job, admin write endpoints, Stripe webhook). New gating sites that need raw tier must add themselves to the allowlist with a documented reason — encodes the bad outcome (LESSON-033) rather than specific implementation strings.

- **CV-255 — BYO-key exemption extended to TokenBudget.check_budget** *(code landed in commit `ff770f0b` under a deploy-titled message; this entry restores attribution)*. The BYO-key exemption added 2026-04-23 (commit `d6319433`) lived only in `UsageLimitMiddleware.dispatch` — it exempted free-tier users with personal API keys from the per-op caps (`ai_calls:10`, `cv_scores:3`, etc) but `TokenBudget.check_budget()` still applied the free-tier 100k daily token wall to the same user. Anti-monetisation: BYO users are the lowest cost-risk users (system bears no token cost — they pay upstream) and they hit walls fastest. Audit reference: [`ClaudeReports/audits/2026-04-28-audit-ai-key-model-tier-management.md`](ClaudeReports/audits/2026-04-28-audit-ai-key-model-tier-management.md) issue ORIG-4.
  - **Helper relocation**: `_user_has_byo_key` body moved from `backend/usage_middleware.py` to `backend/repositories.py` as `user_has_byo_key`. usage_middleware retains a back-compat shim. Both layers now reference one source of truth — no drift over time.
  - **Token-budget exemption**: `TokenBudget.check_budget()` now consults `repo.user_has_byo_key(session, user_id)` after the admin / zero-budget bypass, before the daily/session cap evaluation. BYO free users at any token volume return `True`.
  - **Defensive failure mode**: a DB blip on the BYO lookup is logged with `event_type=byo_check_failed` and falls through to the regular cap check (slightly user-hostile, but the call still works — silent-exception ratchet compliant).
  - **Regression guards** (4-test contract — [`backend/tests/security/test_byo_exemption_token_budget.py`](backend/tests/security/test_byo_exemption_token_budget.py)):
    - Helper relocation: `repo.user_has_byo_key` is importable
    - Behavioural: BYO free user @ 200k tokens → `check_budget` returns True
    - Regression guard: non-BYO free user @ 200k tokens → False (cap holds)
    - No-duplication: usage_middleware delegates to `repo.user_has_byo_key` (forbid-list, LESSON-033)
  - **Multi-agent attribution note**: this fix and a parallel deploy-pipeline session committed simultaneously, and the parallel session's `git add` swept up these staged files into commit `ff770f0b` (titled `fix(deploy): checkout repo in verify job — provider smoke step needs the script`). The pattern is documented in [`memory/project_multi_agent_double_commit_pattern.md`](memory/project_multi_agent_double_commit_pattern.md). All CV-255 code lives in that commit; this CHANGELOG entry restores discoverability for `git log --grep="CV-255"` audits.

- **CV-241 follow-up — `DATABASE_URL` secret name in nightly tier-expiry cron** (commit `920391e7`). CV-241 (commit `9c8f64ed`, 2026-04-25) shipped `tier-expiry.yml` with `DATABASE_URL: ${{ secrets.PROD_DATABASE_URL }}` but the actual repo secret is named `DATABASE_URL` (last updated 2026-03-23, used by deploy). `PROD_DATABASE_URL` never existed. GitHub Actions silently injects empty string for missing secrets, so every nightly run since 2026-04-25 crashed at SQLAlchemy URL parse — 3 nights of missed demotions (04-26, 04-27, 04-28). The DB rows themselves still showed `tier='pro'` for users whose `tier_expires_at` had passed; admin reports + admin Users tab showed stale state. CV-238's defence-in-depth read-time `_effective_tier()` check had been catching them at runtime, so user-visible impact was moderate, but the silent cron failure is exactly the alert pattern the workflow header warned about ("a bad night doesn't block deploys ... wire it in Grafana off `event_type=tier_expiry_job_failed`"). Surfaced from session-start cycle-plan signals (not from the audit). Fix: rename to `secrets.DATABASE_URL`; inline comment added pointing back to this incident so the typo class doesn't recur on the next secret-named workflow.

- **CV-262 — explicit None branch on `get_config` + `save_config_db`** (commit `fad8b6fe`). Both helpers used a truthy check `if user_id:` which treated `None` as "no filter" rather than "match the global (`user_id IS NULL`) row". When a per-user Config row happened to share a name with the intended global row, the helpers picked up the per-user row first and either returned its data (get) or overwrote it (save). Currently safe in production because `save_admin_provider_keys` and `get_admin_provider_keys` use explicit `user_id.is_(None)` filtering (LESSON-042) and bypass the generic helpers — but the footgun was sitting there for any future per-user Config that called the generic helper with `user_id=None`. Audit issue NEW-17. Fix: explicit None branch (`if user_id is None: filter(Config.user_id.is_(None))`) in both helpers. 3-test adversarial contract at [`backend/tests/security/test_config_user_id_filter.py`](backend/tests/security/test_config_user_id_filter.py) seeds a per-user row, calls save with `user_id=None`, asserts the per-user row is untouched and a NEW global row is created.

- **CV-263 — refresh `MODEL_ENV_DEFAULTS` to current-generation slugs** (commit `576fc328`). Two defaults had drifted to legacy-generation slugs: `openai: gpt-3.5-turbo` (current default elsewhere is `gpt-4o-mini`) and `anthropic: claude-3-haiku-20240307` (Haiku 4.5 is current). Personal-key users with no env-var override silently received a sub-optimal model — not broken (the slug was still valid in `PROVIDER_MODELS`) but quality-degrading. Audit issue NEW-18. Fix: refreshed both to current standard-tier slugs that match their corresponding `PROVIDER_MODELS` entries (openai → `gpt-4o-mini` matching github default; anthropic → `claude-haiku-4-5-20251001` matching pluribus pairing). New regression contract at [`backend/tests/ai/test_model_env_defaults_in_catalogue.py`](backend/tests/ai/test_model_env_defaults_in_catalogue.py) asserts every default IS in its catalogue — catches the strong drift class where a default points at a slug never registered (would 404 at provider). Existing `test_personal_key_uses_live_model.py` parametrize updated; the test was renamed from `test_in_code_defaults_match_legacy_os_getenv_defaults` to `test_in_code_defaults_match_current_generation` since openai + anthropic now intentionally differ from the legacy os.getenv values.

- **CV-259 — disallow indefinite Pro on admin promotion** *(code landed in commit `52cd1adb` under a UI-titled message; this entry restores attribution)*. Two interacting paths created indefinite Pro: `PATCH /api/admin/users/{id}/tier` accepted `tier='pro'` with no `tier_expires_at` (or `null`), AND the nightly demotion job filters `WHERE tier_expires_at IS NOT NULL`. Combined: admin promoted a test/contractor account to Pro without expiry → nightly job NEVER demoted them → silent indefinite Pro. `_effective_tier` reported "pro" because no expiry existed to compare. Unintended monetisation bypass via admin promotion. Audit issue NEW-12. Fix: Pro promotion with `expires_at` omitted defaults to a 30-day grace; explicit `expires_at: null` is rejected with 400; explicit ISO expiry is honoured as before. Free demotion still clears expiry (regression preserved). Existing rows with `tier='pro' AND tier_expires_at IS NULL` are NOT migrated by this PR — admin can fix them via the same PATCH (now idempotent and safe). 4-test contract at [`backend/tests/security/test_admin_pro_promotion_requires_expiry.py`](backend/tests/security/test_admin_pro_promotion_requires_expiry.py). **Multi-agent attribution note**: 4th occurrence this session of the parallel session's `git add` sweep — code is in `52cd1adb` (titled `ui(login): tighten copy -- drop italic descriptor, replace subhead with workflow-preview`). All CV-259 code lives in that commit; this CHANGELOG entry restores `git log --grep="CV-259"` discoverability.

- **CV-261 — `TokenBudget._session_usage` cleanup on user state transitions** (commit `fe2efd9f`). The in-memory dict keyed by user_id had no auto-cleanup. Three transitions left stale entries until container restart: sandbox auto-expiry deleted the user from DB but the counter persisted (NEW-14, RAM leak); logout cleared the frontend state but the backend session counter survived (NEW-15); free→Pro promotion didn't reset the counter so a free user at 45k tokens promoted to Pro 250k cap saw the new limit but kept the old counter (NEW-16). Fix: `TokenBudget.clear_user(user_id)` thread-safe + idempotent helper, called from sandbox cleanup loop after `repo.delete_user`, from `auth_logout` after `destroy_session`, and from the admin tier-change endpoint after `session.commit()` (only when old_values != new_values, no-op for idempotent re-PATCHes). Defensive try/except — cleanup failure logs with `event_type=token_budget_cleanup_failed` but never breaks the user's flow. LRU/TTL safety net for unbounded growth deferred (audit acceptance criterion noted but lower priority — dict size bounded by ~5000 active users × ~30 bytes = ~150KB memory). 5-test contract at [`backend/tests/security/test_token_budget_cleanup.py`](backend/tests/security/test_token_budget_cleanup.py) — helper signature/behaviour/idempotency + 3 source-scan forbid-list contracts (sandbox cleanup loop + logout + admin tier change must each call clear_user).

- **CV-258 — per-user-op cost ceiling on retry × fallback × truncation chain** (commit `b74d4350`). Three retry layers compose multiplicatively in the gateway: primary `_call_with_retry` (3 attempts on 429), truncation retry (3 more at 2x budget), fallback chain (~9 providers). Worst case ~27 provider calls per user op. Pathological inputs (huge CV, near-truncation responses) amplified token cost; per-call latency degraded the deployment; provider-side rate limits compounded. Audit issue NEW-11. Fix: new class constant `AIGateway.MAX_PROVIDER_CALLS_PER_OP = 8` (calibrated to allow normal "primary 3x rate-limit retries → 2 fallback providers each 1-2 attempts" without false-tripping while clamping pathological storms); new dedicated exception `RetryStormCapped` (distinguishable from rate-limit / quota — means "stop, our cost-control fired" not "the upstream is throttling us"); counter on thread-local `self._context.provider_call_count`, reset to 0 at every `gateway.call()` entry, incremented in `_call_provider` AFTER the ceiling check; structured telemetry `event_type=retry_storm_capped` with `provider_call_count`/`ceiling`/`task`/`user_id` for ops dashboards. 4-test forbid-list contract at [`backend/tests/security/test_retry_storm_ceiling.py`](backend/tests/security/test_retry_storm_ceiling.py).

- **LESSON-057 — AI routing for personal-key users** *(code landed in commit `906dd897` under a UI-titled message; this entry restores attribution)*. user2's CV Analysis silently fell back to keyword matching despite a valid OpenRouter key because three downstream surfaces bypassed the LESSON-056 router/gateway union fix:
  - **33 service-layer callers** across `cv_analysis.py`, `career_insights.py`, `document_gen.py`, `job_matching.py`, `scoring.py`, `search_helpers.py` called `self.router.select_provider("task", provider)` with NO `user_id` — the router's `_available_providers_for(uid)` fell through to admin-only mode, and user2's personal-key providers were invisible. Fixed: every caller now passes `user_id=user_id`.
  - **Service post-checks** (`if provider not in self._registry.clients: provider = list(...)[0]`) were admin-scoped and would silently swap a user-only provider for admin's first client. Replaced with `if not provider:` (router has already validated against the user-aware union).
  - **Gateway fallback loop** ([`backend/ai/gateway.py`](backend/ai/gateway.py)) reached user-only providers via the LESSON-056 `candidate_ids` union but called `_call_provider` without a `client_config`. `_call_provider` did `config = client_config or self.registry.clients[provider]` — KeyError for user-only providers (admin has no entry). Fixed: fallback now resolves `client_config` from the per-user resolver before each `_call_with_retry` attempt.
  - **Cross-page chip mismatch** (Bug A) — `/api/ai-routing` returned admin-scoped `active_provider` so chips on Applications/Job Search showed "Powered by Google Gemini" while CV Analysis correctly showed OpenRouter. Fixed: `_build_filtered_routing_info` recomputes `active_provider` user-scoped via `select_provider(user_id=...)` for non-admin/non-sandbox users.
- **Regression guards added (3 layers)**:
  - [`backend/tests/infrastructure/test_services_pass_user_id.py`](backend/tests/infrastructure/test_services_pass_user_id.py) — AST forbid-list source-scan over `backend/ai/services/*.py` requiring `user_id=` kwarg on every `self.router.select_provider(...)` call. Adversarially verified.
  - [`backend/tests/ai/test_user_provider_selection.py::TestGatewayFallbackResolvesUserOnlyClientConfig`](backend/tests/ai/test_user_provider_selection.py) — uses REAL `_call_provider` with SDK-level mock so the `clients[provider]` KeyError branch executes for real (no mocks at the bug surface, fixing the LESSON-056 test gap).
  - [`backend/tests/api/test_analyze_cv_user_routing.py::test_full_pipeline_user2_scenario_no_keyerror`](backend/tests/api/test_analyze_cv_user_routing.py) — full HTTP → route → service → gateway → resolver → SDK pipeline via TestClient. Asserts OpenRouter SDK is actually invoked AND returns real analysis (not keyword fallback).
- **CLAUDE.md auto-correction rule #48** added — names all three required behaviours (user_id threading, gateway fallback `client_config` resolution, service post-check removal) so future contributors catch the regression at PR time.
- **Verification**: 1951 of 1951 backend AI + infra + integration tests pass, 0 failures.
- **Multi-agent attribution note**: this fix and a parallel UI session committed simultaneously, and the parallel session's `git add` swept up these staged files into commit `906dd897` (titled `ui(login): restructure right panel into 7 zones per v0.6.2 spec`). The pattern is documented in [`memory/project_multi_agent_double_commit_pattern.md`](memory/project_multi_agent_double_commit_pattern.md). All LESSON-057 code lives in that commit; this CHANGELOG entry restores discoverability for `git log --grep="LESSON-057"` audits.

## [0.6.1] - 2026-04-26

### Added
- **Pro tier — Phase 1 user-facing path (CV-236)** — admin-assigned Pro promotion ahead of Phase 2 Stripe integration. The MON-recommended `free → pro` tier now has a user-facing path:
  - New `PATCH /api/admin/users/{id}/tier` endpoint. Admin-only, sandbox users rejected with 400, audit-logged, idempotent. 13 backend tests covering happy / negative / boundary / edge.
  - **Admin Users tab** gains a "Plan" column with Free / Pro badges and an inline "Set Plan" picker; sandbox rows intentionally have no picker (backend rejects them).
  - **TopNav** displays a small gold "PRO" badge next to the user name when `currentUser.tier === 'pro'`.
  - `tier` and `tier_expires_at` now flow through all three auth response builders (`/api/auth/status`, `/api/auth/me`, `/api/auth/me/profile`) with regression-guard tests.
  - Closes the LESSON-054 anti-pattern: CV-093 shipped the data layer + gating logic in February 2026 but no user-facing path existed for ~2 months.
- **Pro tier — Phase 2 Stripe scaffolding** — initial billing plumbing landed behind the existing tier gating:
  - `feat(billing): Stripe checkout-session scaffold` (CV-239 / #408) — server-side checkout-session creation endpoint.
  - `feat(billing): Stripe webhook handler scaffold` (CV-240 / #410) — webhook receiver for subscription lifecycle events.
  - `feat(billing): nightly demotion job for expired Pros` (CV-241 / #412) — cron job demotes `tier='pro'` users whose `tier_expires_at` has passed back to `tier='free'`.
  - `feat(billing): tier-aware token budget — Pro 5x, Sandbox 0.5x` (CV-244 / #421) — daily AI token budgets now scale with tier.
  - `fix(billing): read-time expiry guard for Pro tier` (CV-238 / #406) — per-request expiry check so a paused subscription stops conferring Pro features even before the nightly demotion runs.
- **CV-237 Style preset dropdown on CV tailoring** — Conservative / Balanced / Creative presets steer the tailoring prompt without exposing the full prompt template. Includes 4-test coverage close-out (cover-letter regression, route helper, source-scan contract, frontend invalid-userStorage).
- **Login landing redesign** — full landing page + Sign In modal (replaces the inline split layout):
  - Two-column split on desktop with mobile-first brand hero, larger CViper logo (desktop 72→104px, mobile 108→144px on landing; 72→108px in the brand hero).
  - "What you get" benefits moved into the navy hero column to kill wasted space.
  - Visual mass on navy hero — 3-tile stat grid + AI-routing diagram backdrop.
  - "Get Started" micro-heading + bigger icon on the white column.
  - Founder bio broadened — industry-specific framing dropped.
- **Settings — Model dropdown for disconnected AI providers** — pre-select a model before connecting a key.
- **Usage — BYO-key quota exemption + binding counter in usage pill** — users on their own API key are exempt from CViper's daily AI quota; the pill now reflects this clearly.
- **Diagnostic redaction helper for user diagnostic bundles (CV-235, commit 1/4)** — redacts PII before bundles are exported.

### Changed
- **AI fallback loop respects user-configured providers (LESSON-051 / Auto-correction Rule #44)** — for non-admin users with personal keys, the gateway no longer iterates the full admin registry. User-facing error chains, fallback attempts, and billing all stay scoped to the user's own provider set, never silently billing the admin on a successful fallback. Forbid-list contract test in `backend/tests/infrastructure/test_admin_registry_leaks.py` source-scans `gateway.py` and fails CI if a `for X in self.registry.clients` loop in user-facing code drops the filter.
- **AI gateway detects max_tokens truncation + retries at 2x budget (LESSON-053)** — refactored to a class-level truncation closure shared across all 5 provider paths, with opt-in retry. Avoids silently shipping a half-completed CV optimisation when the model hits its token cap.
- **AI gateway detects empty provider content at source + JSON-mode + BulletScorer soft-hide** — empty-content detection moved upstream so a single guard catches every provider; BulletScorer renders a soft hide rather than a broken card when the upstream payload is empty.
- **Marketing copy: replaced "Know why you're being rejected" overclaim** — switched to a warmer, outcome-focused framing per the user-saved feedback note (no accusatory marketing copy).

### Fixed
- **Personal-key users get the live model from the registry, not the stale env-var (LESSON-055 / Auto-correction Rule #46)** — `ProviderRegistry.build_client_config` previously read each provider's model directly from `os.getenv("X_MODEL", default)`, bypassing every runtime model-update mechanism (`PUT /api/ai-provider-model/{id}`, `handle_retired_model` autoheal, `hydrate_model_preferences` multi-replica sync). For any user with their own API key, Settings could show the autoheal'd model while every CV analysis call kept hitting the retired slug — producing `model_not_found` 404s and silent fallback to keyword matching. Fix landed in two commits: `30b3f20a` migrated `build_client_config` (per-user path) to `self._resolve_model_for(provider_id)`; the follow-up `a67ffdbf` extended the same migration to **`_init_single_client`** (was wiping autoheal'd models when admin re-saved their key), **`_init_clients`** (boot-time hydration), and the `get_available_providers` all-known list (admin display drift for unconfigured providers). AST forbid-list contract test in `test_provider_model_source.py` fails CI if any factory writes `os.getenv("..._MODEL", ...)` directly. 28 tests including end-to-end autoheal-then-rebuild simulation.
- **Simple AI card — Active + Not Connected cannot coexist on a provider card (LESSON-052 / Auto-correction Rule #45)** — for ~24 hours, `SimpleAISetupCard` showed Google with both badges simultaneously because the "Active" badge was wired to `/api/ai-priority.active_provider` (admin-scoped) while the "Connected" chip read `/api/ai-keys[i].configured` (user-scoped). For an asymmetric scope (admin-only key + user has only a different one), the two endpoints' fields disagreed with no cross-check. Fix: badge derivation reconciles the user-scoped field with the backend value as a constraint. Vitest contract `describe('contract: Active + Not Connected cannot coexist on the same card')` fuzzes the realistic prop combinations and asserts the contradiction class cannot recur.
- **`/api/ai-keys` "Remove" button no longer reports phantom "failed to activate" error (CV-234)** — the post-remove auto-rotate logic ran a redundant activate call that always 404'd because the just-removed key was already gone.
- **`backlog_sync.py pull` adds missing GitHub issues** — `cmd_pull` previously warned-and-skipped any GitHub issue without a matching YAML id, producing 82-item drift since 2026-04-10. Pull now derives a new item from the issue and appends it. Live re-pull added 83 items; 234 YAML == 234 GitHub. 6 unit tests cover add-path, update-path, closed-issue=done, no-duplicates, category default, title-prefix stripping.

### Infrastructure
- **Empty-commit guard** — `chore(infra)` pre-push check rejects commits whose tree is identical to the parent (catches the multi-agent double-commit pattern documented in `project_multi_agent_double_commit_pattern`).

### Auto-correction rules added
- **#44** user-facing enumerations must not leak admin-scoped internal state (LESSON-051)
- **#45** UI status badge derived from two independent API responses requires a contradiction-class contract test (LESSON-052)
- **#46** per-user provider client builder must read `model` from the canonical helper, not from `os.getenv` (LESSON-055)

### Doc versions
- BRD 0.6.1, FSD 0.6.1, TSA 0.6.1, Test-Plan 0.6.1 — all four aligned to app version.

## [0.6.0] - 2026-04-23

### Added
- **CV Optimisation Pipeline complete** — the final feature of the pipeline ships: a **Bullet Quality Scorer** (heuristic CAR-pattern scorer on every CV bullet, deterministic, no AI calls, 0-100 score with weakness tags) rendered on the CV Analysis tab. Complements the AI-powered `optimize_base_cv_bullets` (which only rewrites weak bullets) by scoring *every* bullet with colour-coded feedback. New endpoint `POST /api/cv/score-bullets`.
- **ATS Format Validator expanded** to 6 categories and 23 individual checks (up from the initial 5 checks). Each check returns a pass/warn/fail status with a targeted fix suggestion.
- **One-click "Optimise for this job"** modal — `POST /api/cv/optimize-for-job` runs the full pipeline (health + keywords + bullets) and renders results across 4 tabs (Summary / Health / Keywords / Bullets) with prioritised actions.
- **CV-219 Missing-field chip** — inline edit affordance for Unknown Company/Location on Applications. `PUT /api/saved-jobs/{id}` extended to accept `company` + `location`.
- **CV-230 AI Configuration Simple/Advanced mode split** — new users land on `SimpleAICard` (one provider for every task, low decision load); power users can opt into `AdvancedAIConfig` (priority ordering, per-task tier routing, local Ollama relay) via Settings → Preferences → Interface. New endpoint `PUT /api/user/config-mode`.
- **CV-207 Regional regulatory awareness** — CV tailoring now accounts for UK FCA/PRA/SRA/GDPR and equivalent regulatory frameworks when the target role is in a regulated sector.
- **CV-228 Cloud-user CV resolver** — the app now persists the full CV text so cloud-mode users (who have no local filesystem) can search without the `cv_folder` parameter. Two-step fallback: saved profile → saved CV analysis. Removes "CV folder path is required" errors for cloud users.

### Changed
- **Live OpenRouter model fetching (LESSON-049 / CV-232)** — `fetch_openrouter_models()` now hits the live `/models` endpoint with a 1-hour cache instead of trusting a hardcoded slug list. When OpenRouter retires a slug (they rotate weekly), the dropdown reflects reality instead of handing out a dead id. Static list stays as a safety-net for network errors. Retired `google/gemini-2.0-flash-exp:free` from the hardcoded fallback.
- **Provider-model drift — defence in depth (LESSON-050 / CV-233)** — nightly CI canary runs `scripts/check_provider_model_drift.py` against every configured provider and opens an issue on regression; runtime self-heal auto-swaps to the next-healthy model when a model returns a permanent error.
- **AI routing — state-drift prevention registry (LESSON-048)** — every persistent config that drives runtime behaviour now re-hydrates at startup. Contract test in `test_state_drift_prevention.py` enumerates the registered pairs and fails if any read endpoint forgets the hydrate call.
- **Typography registry** — 4 new tokens extracted to `utils/surfaces.js` (`LINK`, `HERO_STAT_*`, `PROGRESS_CARD_HEADING`); 9 call sites migrated to the shared `TYPOGRAPHY.LINK` token (Tier 3).

### Security
- **LESSON-044 Layer 1+2 AI-string sanitisation** — every user-renderable AI string field is sanitised at both the schema layer and the render layer. `POST /api/add-job-link` now strips HTML from AI-returned company/location fields; sibling surfaces updated.

### Testing
- **Read-read parity registry (LESSON-043 / CV-223 / CV-224)** — every pair of GETs that expose the same field must add a `ReadPair` entry in `test_read_read_parity.py`. Chip-propagation E2E verifies the model indicator stays in sync across CV Analysis, Applications, and Job Search after a model change.
- **JSX `\uXXXX` literal leak guard (LESSON-046)** — new render-and-walk Vitest guard in `noUnicodeEscapeLeak.test.jsx` catches source-level `→`-style escapes that JSX text nodes treat as 6 literal characters instead of the intended unicode arrow.
- **Typography drift detector + pre-commit hook (Layer 9)** — Tier 2 guard flags inline style objects that duplicate surface/typography patterns already in `utils/surfaces.js`.
- **Pre-push Layer 8 soft nudge (CV-225)** — warns when a substantive fix commit (not ui/test/docs scope) ships without a tracking reference (`#NNN`, `CV-NNN`, or `LESSON-NNN`).
- **Bullet Scorer test matrix** — 21 unit + 11 endpoint + 10 component tests, full 7-row coverage (happy / negative / boundary / edge / unicode / metric variants / regression).

### Auto-correction rules added
- **#43** state-drift prevention for (DB Config ↔ in-memory) pairs
- **#44** user-renderable AI string sanitisation (Layer 1+2)
- **#45** cloud-user CV resolver contract (full-text persistence + 2-step fallback)

### Doc versions
- BRD 0.6.0, FSD 0.6.0, TSA 0.6.0, Test-Plan 0.6.0 — all four aligned to app version.

## [0.5.5] - 2026-04-21

### Added
- **"Rejection Intelligence" login hero + SEO pivot** — new marketing-surface copy leading with the rejection-analysis value proposition; updated meta tags.
- **Simplified AIAssistantCard + provider health test endpoint.** `POST /api/ai-providers/{id}/test` lets users verify a configured provider end-to-end from the UI.
- **CV-157 Public Case Study page** — portfolio-grade case study accessible without login.
- **CV-159 UK regulatory framework chips** on job cards (FCA, PRA, SRA, GDPR, etc.) for financial-services roles.
- **CV-160 Rejection Analysis — evidence-first output.** Tightened prompt structure; every conclusion must cite a specific piece of the CV or JD.
- **CV-163 Recruiter-view one-page CV DOCX export.** New condensed layout optimised for recruiter skim reads.
- **CV-155 Tier-gated advanced features** for progressive disclosure (wizard/first-visit users see a focused surface).
- **Job Boards favourites row** and promoted LinkedIn CTA on Search.
- **Task-aware "Powered by" chip** — the chip in page headers now reflects the specific model that ran the most recent task, not just the tier default.
- **Status page — SRE-grade upgrade.** Embedded live Grafana panels on the admin dashboard, Prometheus scraping wired up in Azure, CViper logo + favicon, restored Grafana/CI/staging deeplinks on `/?token=...` bookmarks.
- **`@smoke` golden-path E2E spec** running against live demo.
- **ConfirmDialog** component replacing browser `alert`/`confirm`/`prompt`.
- **Structured 429 for token-budget exhaustion** + `ai_meta` attribution on AI error responses.
- **CV Analysis auto-runs ATS Format Validator** using the already-loaded file (no second upload).
- **Saved Analyses + Saved Job Searches** merged into a segmented card.
- **User-scope parity framework (CV-197)** — repository-level guard for Rule #24 preventing cross-user data leaks via missing `user_id=` arguments.

### Changed
- **Applications density sweep** + cross-page UX consistency pass.
- **Date column consistency** across Search, Applications, Posted Date.
- **"View Job" → "View Original ↗"** with source-aware tooltip.
- **"Application in Progress" → "In Progress"** in status labels.
- **"KW" → "Keyword"** with clearer cost-vs-accuracy tooltip.
- **AboutProject + About page** — AI partnership copy reframed; stuck diagram overlay fix.
- **Documents + Career Insights** empty-states brought to gold-standard pattern.
- **Search Source Results** collapsed into chip strip + expandable details.
- **Saved badge + View →** merged into a single chip.
- **Score display centralised** (`utils/scoreDisplay.js`) — contract test locks it in.
- **AI Assistant priority list** now returns the same filtered shape on PUT as on GET — LESSON-035 (Rule #36).

### Fixed
- **Cloud-mode search without cv_folder (LESSON-037 / Rule #37)** — `/api/search` now auto-resolves a saved profile or saved CV-analysis when the frontend omits `cv_folder` in cloud mode; prevents a blocking "CV folder path is required" 400 on first search for any cloud-mode non-sandbox user.
- **AI Priority reorder drops providers** — the ◀/▶ button in the AI Routing list no longer silently truncates hidden-but-present entries from `priority_order`; two-layer fix (backend PUT/GET parity + frontend reorder helper working on the full list).
- **Admin cannot update cross-user jobs** — `PUT /api/jobs/{id}` now honours admin scope to match DELETE parity.
- **500s leaking `str(e)`** in error responses — standardised redaction + silent-catch wiring on the frontend.
- **AI-key actionable errors** — replaces generic failures with provider-specific guidance.
- **"Configured implies model" contract** — both UI and HTTP layer now enforce that an AI key marked configured has a model populated (eliminates the `provider configured but model=null` limbo state).
- **Grafana embedded dashboards unblocked** (CSP + iframe permissions).
- **Mobile + demo UX sweep** — CV action row, card layout, demo Settings visibility, usage 404, model fallback display.
- **More menu dismisses on tab change**, top hamburger hidden on mobile, StatusBar auto-hides when scrolling content.
- **Provider display-name** consistency + DocumentCentre mobile layout repair.

### Infrastructure
- **Prometheus scraping** wired to `cviper-backend` in Azure.
- **Cloudflare Workers** setup committed; showcase SVG stamps refreshed.
- **`.env.example`** files committed at root, `backend/`, and `frontend/`.
- **Pre-push hook** guards against untracked relative imports (`b76535b7`).
- **Observability**: `event_type` + `exc_info` added to AI endpoint + AI routing GET error logs.
- **Migration idempotency** guards maintained (030, 031 from 0.5.4 continue to pass).

### Testing
- **Coverage sweep** — closed 7 P2/P3 gaps from the 48h audit (CV-192 through CV-201).
- **CLOUD_MODE explicit regression test** paired with Rule #37.
- **User-scope parity framework** replaces ad-hoc per-endpoint isolation checks.

### Docs
- **Drift fix**: `Testing-Strategy-and-Architecture.md` header version corrected from `0.2.4` to `0.5.5` (history table had claimed 0.5.2 but header was never stamped).
- **Test-Plan.md** stamped to `0.5.5` (had drifted to `0.5.1`).
- **All doc versions aligned** — BRD/FSD/TSA/Test-Plan all now at `0.5.5`.

## [0.5.4] - 2026-04-19

### Added
- **WCAG 2.1 AA accessibility compliance + axe-core CI gate.** Automated accessibility testing against axe-core; CI fails on violations.
- **Token usage pill in AI model indicator.** Real-time per-model token usage visible in page headers; hover tooltip explains calls vs tokens and includes efficiency recommendations.
- **50 direct employer career pages across 9 new industries.** Expanded search coverage for employer-direct listings (previously ATS-routed).
- **Consistent AI model indicator in page headers across all tabs.** Unified provider/tier chip formatting everywhere.
- **Authenticated load-test profile.** New `AuthenticatedUser` class in `tests/loadtest/locustfile.py` exercising the real hot path with a bearer token; documented and bundled with the pre-launch audit.

### Changed
- **Registration closed to invite-only** (pre-launch). Public signup surface hidden; existing users unaffected.
- **AI model links** in page-header indicators always open the AI Provider tab in Settings (consistent navigation).
- **Post-deploy bundle verification** added to the deploy workflow — detects stale Docker images before accepting a revision.
- **CDN cache purge** hardened to a hard failure if Cloudflare secrets are missing (prevents silent stale-asset drift).

### Fixed
- **AI Assistant Priority: provider disappears on ◀/▶ click** — two-layer bug: (1) `PUT /api/ai-routing` returned raw `get_task_routing_info()` while `GET` applied `pv.filter_routing_info()`, so non-admin users got a response whose `available_providers` disagreed with what the UI just rendered; (2) `moveProvider` sent only the visible subset back to the server, causing any hidden-but-present entry in `priority_order` to be silently dropped by the wholesale overwrite in `update_priority`. Fix wraps the PUT response in the same filter and extracts reorder logic to a pure helper (`aiPriorityReorder.js`) that operates on the full `priority_order`. Regression guards: backend parity test `test_regular_user_put_response_has_same_keys_as_get` + 8 helper tests covering hidden-entry preservation, length invariance, and boundary no-ops. See LESSON-035.
- **Migration idempotency (030, 031)** — `audit_logs`, `anomaly_alerts`, and `usage_daily_summary` now use the dialect-split `IF NOT EXISTS` pattern. Prevents container crash-loop on fresh-environment deploys where `init_db()` runs `create_all()` before Alembic. New guard test `test_create_table_and_index_is_idempotent` in `test_schema_drift.py` enforces the pattern for all migrations from 025 onward.
- **iPad login logo cropped + Applications header misaligned** at iPad-width breakpoints.
- **Mobile horizontal overflow** and multiple component regressions from mobile screenshot audit.
- **Lever / SmartRecruiters / Eightfold ATS handlers** — repaired parsers after provider-side HTML changes.
- **NatWest** marked as dead in search dead-list; accompanied by ATS health-check test suite.
- **Cross-screen drift** — dates, spelling, icons, and test references aligned across tabs.
- **Login header tightened** — smaller logo, reduced spacing, cleaner hierarchy.
- **Mobile P0 + P1 iPhone UX** — nav, FAB, labels, banner, dropzone text, safe-area padding, floating overlaps, toolbar overflow.

### Infrastructure
- **Grafana custom domain** (`grafana.cviper.uk`) wired up via Bicep (CV-190 follow-up).
- **Staging Bicep** scaffolding added (note: staging ingress remains internal-only; see pre-launch audit).
- **Revision management** extracted into a composite GitHub Action for reuse across deploy jobs.

## [0.5.3] - 2026-04-18

### Added
- **CV Analysis progress card.** Animated full-width card with gradient header, CViper logo spinner, 5-stage step pills, and progress bar replaces the tiny status span during CV analysis. Auto-hides after completion.
- **ATS Format Validator auto-file.** Validator now automatically uses the file loaded in the drag & drop zone instead of requiring a separate upload. One-click "Check Format" button.
- **Shared job table components.** Extracted `SalaryDisplay`, `SourceBadge`, `DescriptionPreview`, and `StatusBadge` from duplicated inline rendering in `ApplicationCard` and `SearchResultsList`. Single source of truth prevents style drift between Search and Applications views.
- **Contract rate display in Search Results.** Day/hourly rates with annual equivalent (e.g. "£450-£500/day / £104k-£115k equiv.") now shown in Job Search — matching Applications table format.

### Fixed
- **Check Listing Status always returning expired.** `_find_expired_signal()` scanned raw HTML including `<script>` tags where SPAs embed template strings like "this job is no longer available" even for active jobs. Fix: strip `<script>`/`<style>`/`<noscript>`/`<template>` blocks before scanning.
- **Job Search vs Applications visual inconsistency.** Aligned salary (green text, rate conversion), source (inline styled, career page icon), date (`formatDate()`), company/location (overflow handling), title (flex layout, badges), and description preview between the two views.

### Changed
- **Badge styling centralised.** All inline badge styles (SCORED, Active, Expired, NEW, SAVED, contract type) replaced with `StatusBadge` component using variant/size/outline props and a centralised colour palette.

## [0.5.2] - 2026-04-17

### Added
- **Usage tracking and Free/Pro tiers (CV-093).** Full-stack metering system: `User.tier` column, `UsageDailySummary` table, `UsageLimitMiddleware` enforcing daily limits on 15 AI endpoints, `GET /api/usage` with per-operation breakdown, `GET /api/usage/limits`, `UsageBadge` with per-operation tooltip, `UpgradeModal` on 429. Free tier: 10 AI calls, 3 CV scores, 5 salary estimates, 2 doc generations per day. Pro/Admin: unlimited. 75 tests.
- **Job alerts — live search integration (CV-082).** `AlertService._run_profile_search()` now calls the real job search engine, filters via `seen_jobs` dedup, and creates in-app notifications via `NotificationBell`. Frontend alert toggle on active search profiles. Background loop runs every 900s. 18 tests.
- **AI bias audit Phase 1 (CV-187).** 31 synthetic CV profiles across 5 bias dimensions (name origin, university prestige, career gaps, graduation year, gendered language). 17 prompt-level tests verify FAIRNESS_GUARDRAIL presence, prompt invariance across demographics, and job-relevant-only scoring dimensions.
- **CV analysis result caching (15-minute TTL).** Content-based cache keys prevent redundant AI calls for the same CV+job pair. 7 backend tests.
- **Inline progress card during CV analysis.** TaskProgressCard component shows real-time status of AI operations with provider attribution.
- **Branded loader SVG** for all AI-driven loading states across the app.
- **Contract rate display in Applications tab.** Day/hourly rates shown alongside annual equivalent for contractor roles. 2 contractor roles added to sandbox seed data.
- **Professional showcase SVG overhaul.** All 6 architecture diagrams redesigned with orthogonal arrows, accurate stats, and brand-consistent styling. Screenshot gallery added.
- **Real-backend E2E test suite.** 5 specs (login+CV, demo session, scoring, doc export, auth refresh) running against Docker Compose with SQLite. Soft CI gate.
- **Critical E2E CI gate.** `@critical-regression` tagged specs now block PR merges.
- **Auto-sync public docs** to cviper-docs repo via GitHub Actions.
- **API contract tests (LESSON-035).** Validates real endpoint response keys match frontend expectations.
- **Benchmark separation guard (LESSON-036).** Asserts filtered queries never leak cross-type benchmarks.

### Fixed
- **Demo mode random redirect to Job Search.** Two race conditions in the wizard/DemoTour system forced tab changes after manual user navigation: signal-driven auto-advance called `onNavigateTab` when data loaded asynchronously, and 8-second timer-driven auto-advance yanked users to the next step's tab. Fix: auto-advance updates step counter only; DemoTour cancels timer on manual nav away.
- **Salary comparison dropdowns empty.** Backend key mismatch (`"location"` vs `"name"`).
- **Contractor rates returning blended data.** Missing `role_type` filter in `get_benchmarks_for_role()`.
- **CI stale branch noise.** Session-start hook reported failures from closed PR branches.
- **Docker E2E permission crash.** Self-healing entrypoint + tmpfs removal.
- **CI gate jobs blocking on skipped checks.** Added gate jobs so conditional steps don't block PR merges.

### Changed
- **Applications table polish.** Consolidated 5 header zones (stats, analytics, toolbar, add-job, reminders) into 2 cards. Row density reduced (~149px to ~120px), alternating row striping, sticky bulk selection bar inside table card, btn-compact conflict resolved (44px duplicate removed), pipeline status as coloured badges, ghost danger buttons for Archive/Delete.
- **Page heading density.** All page headers reduced from 28px/700 to 18px/600 with tighter margins. Content-section and main-container vertical padding reduced.
- **Wizard completion banner removed.** Auto-dismissed via useEffect — nav already provides Create Account CTA.
- **Coin stack salary icon** replaces generic currency circle in Job Search filters.
- **Navy gradient on floating buttons.** Help & Feedback FAB and AboutPill updated from teal to navy/blue colour scheme.
- **Login page polish.** Preview banner, about card, demo CTA redesign.
- **Feature branch policy relaxed** to soft warning until production launch.
- **4 redundant E2E specs removed** (subsumed by -full counterparts). Coverage parity maintained.
- **Full test suite audit.** 5,812 tests across 511 files reviewed. Audit report in `ClaudeReports/audits/`.
- **DPIA updated** with bias audit Phase 1 results, usage tracking limits, solicitor action summary.

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
