# Business Requirements Document (BRD)

## CViper - AI-Powered Job Search & Application Platform

---

| Field | Value |
|-------|-------|
| **Document ID** | BRD-CVIPER-001 |
| **Version** | 0.6.4 |
| **Status** | Pre-Release |
| **Author** | CViper Project Team |
| **Date** | 2026-05-07 |
| **Classification** | Internal |

### Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.6.4 | 2026-05-07 | CViper Project Team | **Cross-app header consistency** — every top-level tab (11 of them: CV Analysis, Job Search, Applications, Settings, Admin, Companies, Career Insights, Skills, News Feed, FAQ, My Requests) now renders title / subtitle / AI-pill through a single `<PageHeader>` component. The structural fix replaces ~10 hand-rolled copies of the same JSX, eliminating the drift that produced inconsistent headers across the Insights tabs. **Insights tabs cleanup** — Companies & Salary Estimates: AI provider attribution standardised on the slim "Powered by …" pill (replacing the legacy fat radio-tile picker — last surviving caller of that legacy component, now removed); UK Regional Salary Comparison normalised to the standard `.card` markup (no AI star — it's a deterministic backend lookup, not AI-powered); filter bar merged into the Estimates Table card-header (was a floating orphan); Market Benchmarks moved to the bottom of the page (reference data). **Skills tab hero dismissible** — the navy "Skills Development Pipeline" hero now respects a localStorage dismissal flag with a "What's this?" inline link in the subtitle to restore. **News Feed compact strip → standard header** — bespoke gradient strip with stat chips replaced with the canonical centred header for cross-tab consistency. **Empty env-var crash class prevention** — RCA'd CV-241 nightly tier-expiry cron failure (silent monetisation leak: 5 nights of expired Pro users retaining Pro entitlements) was rooted in `int(os.environ.get("SMTP_PORT", "587"))` returning `int("")` on empty env vars. Migrated all 22 vulnerable sites in the project source tree to the safe `or` form, plus a CI contract that fails if the anti-pattern reappears. The contract caught 2 sites in `monitoring.py` that the RCA's hand-listed audit missed — proving the structural fix's value over manual enumeration. **Out of scope**: CV-241 cron still failing in production due to Azure PG firewall blocking GitHub Actions runner IPs (Issue #479) — requires an infrastructure fix (Azure CLI / Bicep / self-hosted runner) outside code-only PR scope. **Drift contracts shipped**: PageHeader registry-driven consistency contract (22 tests covering 11 standardised tabs + 9 documented exemptions); empty-env-var anti-pattern forbid-list (3 tests including adversarial positive + adversarial negative). **Tooling**: `version-docs.sh` bug-fix — preserve commit-to-version map across runs, three-tier App-Version resolver (node → python → grep) so the manifest stamps a real version regardless of which tools are on PATH. Commit: 6921a085. |
| 0.6.3 | 2026-05-07 | CViper Project Team | **Job Search structural overhaul** — Search Criteria card split into three structurally honest cards: Search Criteria (location, salary, titles, filters that drive the search), Match Scoring (CV Skills used to rank returned jobs, with explicit "does not affect what's searched" sub-line), Job Sources (boards + direct employers). Asymmetric default behaviour between Job Titles (opt-in, each title multiplies API calls) and CV Skills (opt-out, free scoring inputs) is now self-documenting via purpose-lines under each label. Health-dot legend (working/fragile/broken/unchecked) hoisted from inside Direct Employers to Job Sources level so a single key serves both grids. Duplicate "Search LinkedIn" header button removed (the bottom action-bar CTA already exists). **Mobile experience cleanup** — StatusBar pinned to bottom on mobile (≤768px) above mobile-bottom-nav so progress stays visible while scrolling, replacing the previous top-pinned position that competed with content. Search action buttons inline side-by-side on mobile (was stacked); Search Source Results detail card default-collapsed on mobile; Help & Feedback FAB dims to 0.7 opacity until tap; tighter mobile card padding (~5-10% vertical reclaim); redundant "search complete" success toast removed (sticky in-page banner already conveys this). **Direct Employers parity fixes** — favourites now sort to the top of the grid (mirrors Job Boards behaviour); industry-dropdown count matches the rendered grid for any combination of filter flags (single-source-of-truth predicate in `companyBoardVisibility.js` + 15-permutation parity contract). **Readiness badge rename** — Apply Now → Strong Match, Tailor CV → Needs Tailoring, Upskill → Skills Gap (purely descriptive adjective phrases that mirror the Strong/Fair/Weak score-tier vocabulary; no longer reads as a clickable button). **Industry News Feed hero strip** — tall ~170px centred hero replaced with a 60px compact strip showing trend count, skill count, generated-at chip + Refresh button. **About page polish** — Financial Services tag-cloud collapsed behind a Show details toggle. **Cross-user data hardening** — per-replica cache bypass for user-scoped Config reads + row-locked read-modify-write on user-scoped POSTs (LESSON-080 / Rule #55); `_user_to_dict` aligned with `/api/auth/me` field set so Pro users render correctly in the Plan column (CV-287 / Rule #56); salary-estimate route auto-renames slug on cross-user collision (CV-289). **Diagram pipeline overhaul** — hand-crafted showcase SVGs replaced with JSON-spec generator + Playwright visual regression (LESSON-081 / Rule #57); cache-buster chain contract test (LESSON-082 / Rule #58). **Manual Search Boards panel** — new UI for non-scrapeable boards, paired with a backend untracked-import guard. **Job Search column-width parity** — search-results expansion no longer clips right-side action badges off-screen (Rule #54). Commit: e79a2baf. |
| 0.6.2 | 2026-04-30 | CViper Project Team | **Insights News Feed personalisation fix (LESSON-072)** — endpoint now falls back to `CvAnalysis` when no `SearchProfile` exists, so users who uploaded a CV but never ran a saved search see a personalised industry briefing instead of the generic "0 years experience" copy. Footer nudge copy corrected to "Upload your CV in the CV Analysis tab" (was "Create a search profile…" — wrong action target). Forbid-list contract added to prevent the 5th-surface recurrence of the same bug class (LESSON-037 / LESSON-067 / LESSON-072). **Direct Employers parallel health check (LESSON-073)** — sequential 17-minute scan over 212 boards replaced with `asyncio.gather` + `Semaphore(10)` + per-board 8s timeout, completes in ~170s. Partial results persist via try/finally + on_result callback even if the request is cancelled. New `useEffect` auto-fires the health check on Search-page mount when at least one enabled source is unset/stale (>24h). **Login footer Privacy/Terms/Status links (LESSON-076)** — auth-gate `PUBLIC_TABS` allowlist generalises the hardcoded `?tab=score` bypass; new `StatusPage` component pulls live data from `/api/health/detailed` with a fallback to `/api/health`; "Back to CViper" header on the unauthenticated path so visitors aren't trapped. Footer-link parity contract source-scans `LoginScreen.jsx` for tab links and asserts each is in the bypass allowlist. **Multi-agent infra hardening** — staged-set snapshot guard in pre-commit (LESSON-074) catches commit-absorption races; LESSON-number collision guard in commit-msg hook (LESSON-077). Commit: 80fae424. |
| 0.6.1 | 2026-04-26 | CViper Project Team | **Pro tier user-facing path (CV-236)** — admin-assigned promotion: `PATCH /api/admin/users/{id}/tier` endpoint, "Plan" column + Set Plan picker on Admin Users tab, gold "PRO" badge in TopNav, `tier`/`tier_expires_at` flow through all auth response builders. Closes the LESSON-054 anti-pattern (Feb 2026 data layer with no UI for ~2 months). **Pro tier — Phase 2 Stripe scaffolding** (CV-238/239/240/241/244) — checkout-session endpoint, webhook handler, nightly demotion job for expired Pros, read-time expiry guard, tier-aware token budgets (Pro 5×, Sandbox 0.5×). **CV-237 Style preset dropdown** — Conservative / Balanced / Creative on CV tailoring. **Login landing redesign** — full landing + Sign In modal with two-column desktop split, mobile-first brand hero, larger logo, AI-routing diagram backdrop, 3-tile stat grid; founder bio broadened. **AI fixes** — LESSON-051 fallback loop respects user-configured providers (Rule #44); LESSON-052 Simple AI card "Active + Not Connected" contradiction (Rule #45); LESSON-053 max_tokens truncation detect+retry; LESSON-055 personal-key users get live model from registry (Rule #46). **Settings polish** — Model dropdown for disconnected providers, BYO-key quota exemption, "Remove" button phantom-error fix (CV-234), diagnostic redaction helper (CV-235). **Marketing copy** — replaced "Know why you're being rejected" overclaim with warmer outcome-focused framing. **Infra** — empty-commit guard (multi-agent double-commit pattern); `backlog_sync.py pull` now adds missing GitHub issues (resolves 82-item drift). Commit: a32272f6. |
| 0.6.0 | 2026-04-23 | CViper Project Team | **CV Optimisation Pipeline complete** — Bullet Quality Scorer (heuristic CAR-pattern per-bullet scoring, deterministic, no-AI, #3 of pipeline) rendered on the CV Analysis tab; ATS Format Validator expanded to 6 categories / 23 checks; one-click "Optimise for job" modal with Health / Keywords / Bullets / Summary tabs. **CV-219 Missing-field chip** — inline edit affordance for Unknown Company/Location on Applications. **CV-230 AI Configuration Simple/Advanced mode split** — SimpleAICard for new users (one provider for all tasks), AdvancedAIConfig for power users (priority ordering + per-task routing + local Ollama relay). **CV-207 Regional regulatory awareness** — CV tailoring accounts for UK FCA/PRA/SRA/GDPR and equivalent frameworks when the target role is in a regulated sector. **CV-228 Cloud-user CV resolver** — persistent full CV text + auto-resolve removes "CV folder path is required" errors for users without local filesystem access (LESSON-045). Commit: 9534f137. |
| 0.5.5 | 2026-04-21 | CViper Project Team | Rejection Intelligence hero + SEO pivot, simplified AI Assistant card + provider health test endpoint, CV-157 public case study, CV-159 UK regulatory chips, CV-160 evidence-first rejection analysis, CV-163 recruiter-view one-page DOCX, CV-155 tier-gated progressive disclosure, CV-197 user-scope parity framework, cloud-mode search fallback (Rule #37 / LESSON-037), AI priority PUT/GET parity fix (Rule #36 / LESSON-035), SRE-grade status page upgrade with embedded Grafana panels, ConfirmDialog replaces browser dialogs, task-aware Powered-by chip, full UX consistency sweep across Applications/Documents/Career Insights. Doc versions re-aligned (TSA drift 0.2.4 → 0.5.5; Test-Plan 0.5.1 → 0.5.5). Commit: 11778d0b. |
| 0.5.2 | 2026-04-17 | CViper Project Team | Usage tracking and Free/Pro tiers (CV-093), job alerts live search integration (CV-082), AI bias audit Phase 1 (CV-187), CV analysis caching, Applications UI overhaul, showcase SVG redesign, real-backend E2E, full test suite audit (5,812 tests). |
| 0.2.2 | 2026-03-27 | CViper Project Team | Version reset to align with application semver (pre-release). Consolidates all prior work (formerly v1.0–v2.1). Full history archived in `docs/Archive/`. |
| 0.2.3 | 2026-03-27 | CViper Project Team | Sandbox Gemini quota fix (permanent quota detection, circuit breaker force-open, user-facing fallback message). Alembic migration 013 batch fix for SQLite compatibility. CI pipeline upgraded to PostgreSQL 16 service container for schema drift checks. |
| 0.5.1 | 2026-04-13 | CViper Project Team | Career Intelligence UI (role discovery, career progression, training plans), UK regional salary comparison, legal markdown rendering, AI provider settings redesign, progressive disclosure, wizard mode, E2E guards. All doc versions aligned to 0.5.1. |
| 0.3.2 | 2026-04-10 | CViper Project Team | CV Optimisation Pipeline (keyword injection, ATS format validator, one-click optimise-for-job). Training Provider Foundation (8 providers, certification mapping, skill progress tracking). AI Ethics & Fairness (fairness guardrails in all AI prompts, confidence scores, Challenge This Score, AI transparency disclosure). Growth Readiness (Open Graph/Twitter meta tags, robots.txt/sitemap.xml, PWA install prompt, Plausible analytics). Cross-user Data Isolation (3-layer prevention: backend scoping, frontend state reset, userStorage namespacing). API Contract Tests (14 contract tests + shared schema for response shape validation). All P0 security items verified (CSP headers, CORS guards, secret encryption, ToS, subprocess hardening). |
| 0.3.1 | 2026-04-07 | CViper Project Team | Phase 0 security hardening complete: encrypted `.env` secrets via Fernet master key, SecurityHeadersMiddleware (HSTS, CSP report-only, X-Frame-Options, Referrer-Policy, Permissions-Policy), fatal production guards for SECURE_COOKIES and CORS_ORIGINS wildcards, all `shell=True` removed from backend with AST ratchet test. OAuth providers extended to LinkedIn, Google, and Microsoft (Entra ID, common tenant). New `?tab=` deep-link support for external URLs (e.g. `/?tab=privacy`, `/?tab=terms`). Terms of Service document published. Public-route registry refactor (single source of truth for auth middleware allowlist). News Feed tab full-width layout fix. Stale chunk prevention: index.html always revalidates, lazyRetry wrapper on all React.lazy imports, Cloudflare CDN cache purge on deploy. PR-based backlog sync to respect branch protection. |

> **Note:** Versions prior to 0.2.2 used an independent numbering scheme (v1.0–v2.1). Those documents are preserved in `docs/Archive/` for reference. From this version onward, document versions track the application version in `package.json`.

---

## 1. Executive Summary

### 1.1 Purpose

This document defines the business requirements for **CViper**, an AI-powered job search and application management platform designed for UK-based technology and financial services professionals. CViper automates the most time-consuming aspects of job hunting: searching across multiple boards, tailoring CVs to specific roles, scoring job-to-candidate fit, and managing the full application lifecycle.

### 1.2 High-Level Goals

- Reduce the time spent on manual job searching by 70% through multi-source aggregation and AI-powered filtering
- Increase CV-to-interview conversion rates by generating ATS-optimised, role-tailored application documents
- Provide data-driven career insights (salary benchmarks, skills gap analysis, market trends) to support informed decision-making
- Deliver a secure, multi-user platform suitable for individual professionals and small recruitment teams

### 1.3 Background and Business Context

The UK technology and financial services job market is characterised by:

- **Fragmented job sources**: Roles are spread across 9 supported job boards (Reed, Adzuna, Jooble, Remotive, Findwork, Freelancer, LinkedIn, eFinancialCareers, Jobserve) and 200+ individual company career pages, each with different ATS platforms (Workday, Greenhouse, Lever, SmartRecruiters, etc.)
- **High application overhead**: Tailoring a CV and cover letter for a single role takes 30-60 minutes; professionals applying to 10+ roles per week spend 5-10 hours on document preparation alone
- **Opaque market data**: Salary ranges, skill demand trends, and company hiring patterns are not easily accessible in a consolidated view
- **ATS gatekeeping**: 75% of CVs are rejected by Applicant Tracking Systems before a human reviews them, often due to keyword mismatches rather than candidate quality

CViper addresses these pain points through automation, AI analysis, and a unified workflow from search through to application tracking.

---

## 2. Business Objectives

### 2.1 Measurable Goals

| ID | Objective | Metric | Target |
|----|-----------|--------|--------|
| OBJ-001 | Reduce job search time | Hours per week spent searching | < 2 hours (from ~8 hours) |
| OBJ-002 | Increase application throughput | Applications submitted per week | 15+ tailored applications |
| OBJ-003 | Improve ATS pass rate | % of applications passing ATS screening | > 60% (from ~25% industry average) |
| OBJ-004 | Salary awareness | % of tracked jobs with salary data | > 80% (actual or AI-estimated) |
| OBJ-005 | Multi-provider AI resilience | System uptime when primary AI is unavailable | 99.5% via automatic provider fallback |

### 2.2 Success Criteria

- Users can search 15+ job boards and 80+ company career pages in a single operation
- AI-generated CVs score 70+ on ATS compatibility checks
- Fit scores correlate with interview invitation rates (validated via user feedback)
- Platform operates without AI provider lock-in (8+ providers supported, automatic fallback)

### 2.3 Key Performance Indicators (KPIs)

| KPI | Description | Measurement |
|-----|-------------|-------------|
| Search Coverage | Number of unique job sources queried per search | Count of enabled boards + career pages |
| Scoring Accuracy | Correlation between AI fit score and user-reported interview success | User feedback vs. score percentile |
| Document Quality | ATS compatibility score of generated CVs | Mean ATS score across generated documents |
| Provider Availability | Percentage of AI requests successfully served | (Successful calls / Total calls) x 100 |
| User Productivity | Jobs tracked, scored, and applied to per session | Application funnel metrics |

---

## 3. Stakeholder Analysis

### 3.1 Stakeholder List

| Stakeholder | Role | Interest | Influence |
|-------------|------|----------|-----------|
| Job Seekers (Primary Users) | End users — technology and finance professionals | High | High |
| Recruitment Teams | Secondary users — manage candidate pipelines | Medium | Medium |
| Platform Administrator | Root user — manages users, defaults, system config | High | High |
| AI Provider Partners | External — OpenAI, Anthropic, Google, etc. | Low | Medium |
| Job Board Operators | External — Reed, Indeed, LinkedIn, etc. | Low | Low |
| Company HR/ATS Systems | External — Workday, Greenhouse, Lever, etc. | Low | Low |

### 3.2 Roles and Responsibilities

| Role | Responsibilities |
|------|-----------------|
| **Job Seeker** | Configure search criteria, review results, manage applications, generate documents, track progress |
| **Administrator (Root)** | Manage user accounts, set default job boards/career pages, configure system-wide settings, monitor system health, perform database maintenance |
| **System** | Aggregate job listings, dispatch AI requests, generate documents, track scores, enforce rate limits, manage sessions |

### 3.3 RACI Matrix

| Activity | Job Seeker | Administrator | System |
|----------|-----------|---------------|--------|
| Configure search criteria | R, A | | |
| Execute job search | R | | A |
| Score job-to-CV fit | R | | A |
| Generate tailored CV | R | | A |
| Track application status | R, A | | |
| Manage user accounts | | R, A | |
| Set default configurations | | R, A | |
| Monitor system health | I | R, A | |
| AI provider failover | | I | R, A |
| Data backup and cleanup | | R | A |

*R = Responsible, A = Accountable, C = Consulted, I = Informed*

---

## 4. Scope Definition

### 4.1 In-Scope Features

| ID | Feature | Phase |
|----|---------|-------|
| F-001 | Multi-source job search (15+ boards, 80+ career pages) | 1-7 |
| F-002 | AI-powered CV analysis and skill extraction | 2-3 |
| F-003 | Job-to-CV fit scoring (AI + keyword hybrid) | 3 |
| F-004 | Tailored CV and cover letter generation (DOCX/PDF) | 4 |
| F-005 | ATS compatibility scoring | 4 |
| F-006 | Skills gap analysis and trending skills dashboard | 5 |
| F-007 | Interview preparation content generation | 6 |
| F-008 | Application status tracking and follow-up reminders | 6 |
| F-009 | Saved search profiles and deduplication | 7 |
| F-010 | Job comparison (side-by-side) and Kanban board | 8 |
| F-011 | Multi-user authentication with role-based access | 9 |
| F-012 | Rate limiting, SSRF prevention, URL validation | 9 |
| F-013 | Company salary estimation and benchmarking | Post-core |
| F-014 | AI provider routing (priority-based with fallback) | Post-core |
| F-015 | Service monitoring dashboard with Grafana integration | Post-core |
| F-016 | Prompt Lab (compare AI responses, template editing) | Post-core |
| F-017 | Admin panel (user management, database maintenance) | Post-core |
| F-018 | Automated Grafana deployment with preconfigured dashboards, data sources, and access control | Post-core |
| F-019 | Automated Ollama deployment with model installation, API exposure, and resource management | Post-core |
| F-020 | Simplified CV Analysis page with inline AI provider indicator | Post-core |
| F-021 | Guided onboarding stepper (4-step workflow progress) | Post-core |
| F-022 | CV-to-Search data flow visibility (badges, links, suggested criteria) | Post-core |
| F-023 | Collapsible Job Sources section with smart auto-expand | Post-core |
| F-024 | Advanced Mode toggle to show/hide developer tools (Monitoring, Prompt Lab) | Post-core |
| F-025 | Settings sub-navigation (General, AI Providers, Search & Keywords, Preferences) | Post-core |
| F-026 | Contextual empty states with icons and CTAs across primary tabs | Post-core |
| F-027 | Adzuna affiliate job search with click tracking for revenue reconciliation | Post-core |
| F-028 | 7-step onboarding registration (industry, role, boards, career pages, privacy policy) | Post-core |
| F-029 | Password reset flow (forgot password, email-based token validation and reset) | Post-core |
| F-030 | Employment type classification (Contract/Permanent) on salary estimates | Post-core |
| F-031 | Expanded industry sector coverage (19 UK sectors with 374 role benchmarks) | Post-core |
| F-032 | Provider signup links with free tier hints in AI API Keys configuration | Post-core |
| F-033 | Git commit hash display for build identification across UI | Post-core |
| F-034 | iPhone and mobile safe area rendering with dynamic viewport units | Post-core |
| F-035 | AI model selection per provider with persistent model preferences | Post-core |
| F-036 | Onboarding bulk selection (Select All / Clear All) for job boards and career pages | Post-core |
| F-037 | Admin CLI tools for password reset and system diagnostics | Post-core |
| F-038 | Hybrid JWT authentication with access tokens in memory and refresh tokens in httpOnly cookies | Post-core |
| F-039 | LinkedIn and Google OAuth2 social login via authlib | Post-core |
| F-040 | Per-session sandbox with unique user per demo and Alex Morgan BA persona seed data | Post-core |
| F-041 | Frontend AuthContext provider for centralised auth state management | Post-core |
| F-042 | Sandbox UI components (SandboxBanner with session timer, ExampleBadge, SignUpPrompt modal) | Post-core |
| F-043 | Step-by-step API key guide for novice users (FAQ + landing page collapsible panel) | Post-core |
| F-044 | AI-generated content disclaimer (in-app, FAQ, and Privacy Policy Section 5) | Post-core |
| F-045 | Token explainer FAQ entry for non-technical users | Post-core |
| F-046 | Prompt Lab read-only mode for demo/sandbox users | Post-core |
| F-047 | iCIMS and Taleo ATS handlers for career page discovery | Post-core |
| F-048 | Concurrent discovery pipeline with board health monitoring | Post-core |
| F-049 | Infrastructure dependency map and pre-deploy config validation | Post-core |
| F-050 | Server-side path redaction (Unix paths never exposed to clients) | Post-core |
| F-051 | PWA support with stale-while-revalidate service worker | Post-core |
| F-052 | Landing page redesign: demo first, API key guide, login below | Post-core |
| F-053 | Icon completeness test preventing missing icon text fallback | Post-core |
| F-054 | CV Optimisation Pipeline: keyword injection suggestions, ATS format validator, one-click optimise-for-job | Post-core |
| F-055 | Training Provider Foundation: 8 providers (4 free, 4 paid), certification mapping, skill progress tracking via Skills & Training tab | Post-core |
| F-056 | AI Ethics & Fairness: fairness guardrails in AI prompts, confidence scores on all AI outputs, Challenge This Score button, AI transparency disclosure | Post-core |
| F-057 | Growth Readiness: Open Graph/Twitter meta tags, robots.txt, sitemap.xml, PWA install prompt, Plausible analytics integration | Post-core |
| F-058 | Cross-user Data Isolation: 3-layer prevention (backend user_id scoping, frontend state reset on user switch, userStorage namespacing per user) | Post-core |
| F-059 | API Contract Tests: 14 contract tests with shared schema file for response shape validation across all major endpoints | Post-core |
| F-060 | Security hardening verification: all P0 security items verified (CSP headers, CORS guards, secret encryption, Terms of Service, subprocess hardening) | Post-core |

### 4.2 Out-of-Scope Items

| Item | Reason |
|------|--------|
| Automated job application submission | Legal/ethical — requires explicit user action per application |
| Real-time job alerts/notifications | Not required for MVP; future enhancement |
| Mobile native application | Web-first approach; responsive design sufficient |
| Recruitment CRM features | Platform targets individual job seekers, not recruiters at scale |
| Headless browser scraping | JS-rendered career pages (Capgemini, IBM) require Selenium/Puppeteer — deferred |
| Payment/subscription system | Free tool; no monetisation in current scope |

### 4.3 Assumptions

| ID | Assumption |
|----|-----------|
| A-001 | Users have access to at least one AI provider API key (free tiers available from Google, GitHub, etc.) |
| A-002 | Job boards and ATS APIs remain publicly accessible without authentication (Greenhouse, Lever, SmartRecruiters) |
| A-003 | Users provide their own CV text for analysis and document generation |
| A-004 | SQLite is sufficient for local development; CI uses PostgreSQL 16 for schema drift checks; production uses Azure PostgreSQL Flexible Server for concurrent multi-user access |
| A-005 | The platform runs on Azure Container Apps with scale-to-zero capability (1-3 backend replicas) |

### 4.4 Constraints

| ID | Constraint |
|----|-----------|
| C-001 | AI provider free-tier rate limits (e.g., Google Gemini: 15 RPM, OpenAI: varies by plan) |
| C-002 | ATS API availability — some platforms (Workday, Oracle HCM) have undocumented or restricted APIs |
| C-003 | GDPR compliance for UK users — personal data (CV text, application history) must be handled appropriately |
| C-004 | No modification of theme.css — all UI changes must use existing CSS classes or inline styles |
| C-005 | SQLite single-writer constraint applies to local development only; production uses PostgreSQL with connection pooling |

---

## 5. Business Requirements

### 5.1 High-Level Requirements

| ID | Requirement | Priority | Category |
|----|------------|----------|----------|
| BR-001 | The system shall aggregate job listings from multiple public job boards and company career pages, returning all results in a single search operation | Must Have | Search |
| BR-002 | The system shall analyse a user's CV to extract skills, experience, job titles, and qualifications using AI | Must Have | CV Analysis |
| BR-003 | The system shall score each job listing against the user's CV profile to indicate fit quality (0-100) | Must Have | Scoring |
| BR-004 | The system shall generate role-tailored CVs and cover letters optimised for ATS systems | Must Have | Documents |
| BR-005 | The system shall track application status through the full lifecycle (not applied, applied, interviewing, rejected, offer) | Must Have | Tracking |
| BR-006 | The system shall support multiple AI providers with automatic failover when the primary provider is unavailable | Must Have | Resilience |
| BR-007 | The system shall provide salary estimates using curated benchmarks, market data APIs, and AI estimation | Should Have | Insights |
| BR-008 | The system shall identify skills gaps between the user's CV and their target roles | Should Have | Insights |
| BR-009 | The system shall generate interview preparation materials tailored to specific job applications | Should Have | Insights |
| BR-010 | The system shall support multiple user accounts with data isolation, role-based access (user/admin), hybrid JWT/session authentication, and OAuth2 social login | Should Have | Security |
| BR-011 | The system shall enforce rate limits to prevent abuse and respect external API quotas | Must Have | Security |
| BR-012 | The system shall prevent SSRF attacks by validating all external URLs before making outbound requests | Must Have | Security |
| BR-013 | The system shall deploy and configure Grafana automatically with preconfigured dashboards (provider health, usage vs limits, token consumption, request logs, error rates, latency), Infinity datasource auto-provisioned, and admin access control. Security panels (auth events, failed logins, RBAC denials, token budget, AI usage by provider, key source distribution) and alert rules (brute force, budget exceeded, RBAC spike, key decryption failure) shall be included | Must Have | Monitoring |
| BR-014 | The system shall allow administrators to manage user accounts, set default configurations, and perform database maintenance | Could Have | Admin |
| BR-015 | The system shall export job data and application documents in standard formats (CSV, XLSX, DOCX, PDF, ZIP) | Should Have | Export |
| BR-016 | The system shall deduplicate job listings across sources to prevent showing the same role multiple times | Must Have | Search |
| BR-017 | The system shall allow users to save and reuse search profiles (criteria, boards, keywords) | Should Have | Workflow |
| BR-018 | The system shall provide trending skills analysis based on recent search results | Could Have | Insights |
| BR-019 | The system shall deploy and configure Ollama as the local LLM inference engine, with automatic model installation, API availability, and resource allocation | Must Have | AI Runtime |
| BR-020 | The system shall auto-save configuration changes to eliminate manual save operations | Should Have | UX |
| ~~BR-021~~ | *Consolidated into BR-013 (Grafana dashboard panels)* | — | — |
| ~~BR-022~~ | *Consolidated into BR-013 (Infinity datasource provisioning)* | — | — |
| BR-023 | Ollama shall automatically download and load the configured model (default: llama3.2) on first startup, and expose its API endpoint to the application backend | Must Have | AI Runtime |
| BR-024 | The monitoring infrastructure (Grafana, Prometheus, Loki) and local AI runtime (Ollama) shall be included in Docker Compose, Azure Bicep IaC, and CI/CD pipeline definitions so they deploy alongside the application with no manual intervention | Must Have | DevOps |
| BR-025 | All users — existing, new, and regardless of role — shall have immediate, equal access to Ollama with no per-user provisioning, enablement, or configuration | Must Have | AI Runtime |
| ~~BR-026~~ | *Consolidated into BR-025 (new user access)* | — | — |
| ~~BR-027~~ | *Consolidated into BR-025 (role-agnostic access)* | — | — |
| BR-028 | The production database shall use Azure Database for PostgreSQL Flexible Server to support concurrent multi-user access, managed backups, and horizontal scalability | Must Have | Infrastructure |
| BR-029 | All cloud infrastructure shall be defined as Infrastructure-as-Code using Azure Bicep templates, enabling repeatable, auditable deployments from a single command | Must Have | DevOps |
| BR-030 | Production deployments shall require manual approval via a workflow_dispatch-triggered GitHub Actions workflow, with a preflight check that verifies CI has passed on the target commit | Must Have | DevOps |
| BR-031 | The application shall be accessible via a custom domain (cviper.ai) with a managed SSL certificate, deployed to Azure Container Apps in UK South for data residency compliance | Must Have | Infrastructure |
| BR-032 | The system shall enforce complete data isolation between users: every data table with a `user_id` column shall filter queries by the authenticated user, preventing cross-user data leakage | Must Have | Security |
| BR-033 | User sessions shall enforce idle timeout (default: 60 minutes) in addition to absolute expiry, and cookies shall support the `Secure` flag for HTTPS deployments | Must Have | Security |
| BR-034 | Users shall be able to configure their own AI provider API keys, which are resolved per-request (user key → server default fallback) without affecting other users | Should Have | Security |
| BR-035 | Admin-only endpoints shall be enforced at the router level via a centralised RBAC dependency, not inline per-endpoint checks | Must Have | Security |
| BR-036 | User API keys stored in the database shall be encrypted at rest using Fernet symmetric encryption with a server-managed master key, and the system shall support master key rotation without downtime | Must Have | Security |
| BR-037 | The system shall enforce per-user daily AI token budgets (configurable via `USER_DAILY_TOKEN_LIMIT`, default: 100,000 tokens), with root users exempt and clear error messaging when budgets are exceeded | Should Have | Security |
| ~~BR-038~~ | *Consolidated into BR-013 (security panels and alert rules)* | — | — |
| BR-039 | The CV Analysis page shall display AI provider status as a compact inline indicator next to the Analyze button, not as a full configuration card; provider configuration shall remain exclusively in Settings | Should Have | UX |
| BR-040 | The system shall provide guided onboarding via WizardMode (auto-starts for first-visit/demo users, constrains tabs to one at a time, auto-advances on completion signals) and ProgressStepper (ambient 5-step tracker). Wizard uses separate step sets for registered users (Upload → Search → Score → Save) and demo users (Explore CV → See Scores → Try Search → Your Turn). Welcome modals enhance but do not gate the wizard. | Should Have | UX |
| BR-041 | The Job Search tab shall surface data-flow links from CV Analysis (e.g., "Suggested from CV" badges, "analyze your CV" links) to make the connection between CV analysis and search criteria visible | Should Have | UX |
| BR-042 | The Job Search form shall group Job Boards and Career Pages into a collapsible "Job Sources" section, auto-expanded when sources are loaded, to reduce cognitive overload on the primary search interface | Should Have | UX |
| BR-043 | Navigation uses progressive tab disclosure with 3 tiers: focused (4 core tabs for new users), standard (+ Companies, Insights, Skills after CV + search), full (all tabs after applying or "Show all tabs" toggle). Desktop tabs grouped into "Insights" and "More" dropdown menus. Developer tabs (Monitoring, Prompt Lab) additionally gated by Advanced Mode toggle. Both preferences persisted via `userStorage.getItemGlobal` and backend config. | Should Have | UX |
| BR-044 | The Settings page shall use sub-navigation tabs (General, AI Providers, Search & Keywords, Preferences) to organise content into logical sections instead of a single scrollable page | Should Have | UX |
| BR-045 | All primary tabs (CV Analysis, Job Search, Companies) shall display meaningful empty states with contextual icons, descriptive messages, and call-to-action buttons when no data is present | Should Have | UX |
| BR-046 | The system shall prevent sandbox account abuse through a 5-layer strategy: browser fingerprinting (canvas + screen + timezone SHA-256), per-fingerprint daily session limits (50/day), per-IP daily session limits (100/day), dedicated sandbox provider routing (sandbox_google + sandbox_openrouter with daily quotas), AI response truncation for sandbox users (capped scores, skills, rationale), and 1-hour session auto-expiry with background cleanup | Must Have | Security |
| BR-047 | The system shall integrate Adzuna as an affiliate job search source, using redirect URLs for commission tracking, with a click-tracking endpoint (`POST /api/track-click`) storing URL hashes for revenue reconciliation | Should Have | Search |
| BR-048 | User registration shall follow a 7-step onboarding flow: (1) credentials, (2) personal details, (3) industry selection, (4) role selection (filtered by chosen industries), (5) job boards, (6) career pages, (7) privacy policy accept/decline. Declining the privacy policy shall reset to the login screen with an explanatory message | Must Have | UX |
| BR-049 | The system shall provide a password reset flow: forgot password link on login, email-based token generation (`POST /api/auth/forgot-password`), token validation, and password reset (`POST /api/auth/reset-password`) | Should Have | Security |
| BR-050 | Salary estimates shall include an employment type classification (Contract or Permanent), displayed as a sortable colour-coded badge in the Companies tab with a Type dropdown in the new estimate form | Should Have | Insights |
| BR-051 | The system shall optimise end-to-end performance through async database calls (`asyncio.to_thread`), GZip response compression, OpenTelemetry distributed tracing, lazy-loaded modals, progressive rendering with pagination, and Azure VNet + PgBouncer connection pooling | Should Have | Performance |
| BR-052 | The system shall support 19 UK industry sectors for onboarding role selection and salary benchmarking, with a curated set of 374 role benchmark entries across all sectors | Should Have | Insights |
| BR-053 | The application shall display the git commit hash (short SHA) alongside the version number in TopNav, Sidebar, and LoginScreen to identify which build is running, injected at build time via Vite define | Should Have | UX |
| BR-054 | The frontend shall render correctly on iPhone and mobile Safari using `viewport-fit=cover`, dynamic viewport units (`100dvh`), and `env(safe-area-inset-*)` padding to prevent content being obscured by the notch, home indicator, or rounded corners | Should Have | UX |
| BR-055 | Users shall be able to select the AI model for each configured provider via a dropdown in Settings, with model lists hardcoded per provider (Phase 1) and the selected model persisted to the database config table as `provider_model_preferences` | Should Have | AI |
| BR-056 | The registration onboarding flow shall provide Select All / Clear All toggle buttons on the Job Boards (Step 5) and Career Pages (Step 6) steps to allow bulk selection | Should Have | UX |
| BR-057 | The registration flow shall not collect a separate Display Name; the username shall be used as the display identifier throughout the application | Should Have | UX |
| BR-058 | The Service Control endpoints shall be restricted to admin (root) users only, enforced at the route level via `Depends(require_root)`. The frontend Admin tab hides Service Control from non-root users; sandbox users are additionally blocked by middleware | Must Have | Security |
| BR-059 | The system shall provide admin CLI tools for password reset (`reset_password.py`) and system diagnostics (`diagnose.py`) that operate directly against the database without requiring the web server | Should Have | Admin |
| BR-060 | The frontend shall use CSS custom properties (`var(--*)`) from theme.css for all colour references; inline hardcoded hex values shall be replaced with CSS variable references for consistency and maintainability | Should Have | UX |
| BR-061 | The ProgressStepper component shall use a compact design with reduced padding, smaller step indicators, and tighter spacing to minimise vertical space consumption on the main workflow view | Should Have | UX |
| BR-062 | The system shall support hybrid JWT authentication: short-lived access tokens (15 min) stored in React state (never localStorage) and long-lived refresh tokens (7 days) stored as httpOnly cookies, with automatic silent refresh before expiry | Must Have | Security |
| BR-063 | The system shall support LinkedIn and Google OAuth2 social login via authlib, creating user accounts automatically on first OAuth login and merging profile data on subsequent logins | Should Have | Security |
| BR-064 | Sandbox sessions shall create a unique per-session user (not a shared account), seeded with realistic Alex Morgan Senior Business Analyst persona data (4 jobs, 3 companies, 15 benchmarks, 1 pre-analysed CV), expiring after 1 hour | Must Have | UX |
| BR-065 | Sandbox users who create a real account (via registration or OAuth) shall have their sandbox data migrated to the new account, preserving all jobs, companies, estimates, and analyses | Should Have | UX |
| BR-066 | The frontend shall provide a centralised AuthContext provider managing JWT tokens, OAuth callback handling, silent refresh, and auth state (user, isAuthenticated, isSandbox, isAdmin) for all components | Must Have | Security |
| BR-067 | The sandbox banner shall display a real-time countdown timer showing remaining session time, with visual urgency (colour change) when under 5 minutes, and auto-logout on expiry | Should Have | UX |
| BR-068 | The system shall maintain full backward compatibility with existing username/password authentication and session-cookie flow alongside the new JWT/OAuth system | Must Have | Security |
| BR-069 | The system shall provide AI-powered guided base CV bullet optimization, analysing PROFESSIONAL EXPERIENCE bullet points and suggesting CAR-pattern rewrites with quantification and strong action verbs | Should Have | CV Analysis |
| BR-070 | Service control endpoints (`/api/service/backend`, `/api/service/frontend`) shall enforce admin-only access via router-level `require_root` dependency, blocking both standard users and sandbox users. Advanced tabs (Monitoring, Prompt Lab) shall not auto-display for sandbox users; sandbox users must enable Advanced Mode like standard users | Must Have | Security |
| BR-071 | The system shall provide a CV Optimisation Pipeline: keyword injection suggestions (identifying missing ATS keywords from job descriptions), ATS format validation (checking CV structure against ATS compatibility rules), and one-click optimise-for-job (automated CV tailoring for a specific role) | Should Have | CV Analysis |
| BR-072 | The system shall provide a Training Provider Foundation with 8 curated providers (4 free: freeCodeCamp, Coursera Audit, edX Audit, Khan Academy; 4 paid: Udemy, Pluralsight, LinkedIn Learning, Codecademy Pro), certification mapping to skills, and skill progress tracking accessible via a Skills & Training tab | Should Have | Insights |
| BR-073 | All AI-powered scoring and analysis features shall include fairness guardrails in prompts (preventing bias based on name, age, gender, ethnicity), display confidence scores alongside results, provide a "Challenge This Score" button allowing users to request a re-evaluation with reasoning, and include an AI transparency disclosure explaining how scores are generated | Must Have | AI |
| BR-074 | The application shall include growth-readiness features: Open Graph and Twitter Card meta tags for social sharing, robots.txt and sitemap.xml for search engine discoverability, a PWA install prompt for mobile users, and Plausible analytics integration for privacy-respecting usage tracking | Should Have | UX |
| BR-075 | The system shall enforce complete cross-user data isolation through a 3-layer prevention strategy: (1) backend repository functions filter all queries by `user_id`, (2) frontend React state resets all user-scoped data on user switch via `handleLogout` and `useEffect` watching `currentUser?.id`, (3) frontend localStorage values use `userStorage` namespacing (`cviper:u:<userId>:<key>`) with `purgeAllUserScopedStorage()` on user switch | Must Have | Security |
| BR-076 | The system shall maintain API contract tests (14 tests minimum) with a shared schema file validating response shapes across all major endpoints, ensuring backward compatibility of API responses | Must Have | Quality |
| BR-077 | All P0 security controls shall be verified and maintained: Content Security Policy headers, CORS origin guards, secret encryption at rest via Fernet master key, Terms of Service publication, and subprocess hardening (no `shell=True` in backend, enforced by AST ratchet test) | Must Have | Security |

#### 5.1.1 RBAC Permission Matrix

The following matrix defines the authoritative access levels for all user roles. All enforcement is layered: backend route dependencies and middleware are the primary controls; frontend conditional rendering is defence-in-depth only.

| Feature / Endpoint | Admin (root) | Standard User | Sandbox User | Enforcement |
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
| Cross-user data access | Full (all users) | Own data only | Own data only | Repository-level `user_id` filter |
| **AI Provider Access** | | | | |
| All providers | Visible | All except local-only and sandbox_ prefixed | sandbox_google, sandbox_openrouter, keyword only | `provider_visibility.py` |
| Ollama / Pluribus (local only) | Full | Full | Blocked | Backend MW + frontend |
| **Infrastructure** | | | | |
| `/api/service/*` | Full | 403 | 403 | `require_root` route dependency |
| `/api/ollama/*` | Full | Full | Blocked | Backend middleware |
| `/api/admin/*` | Full | 403 | 403 | `require_root` router dependency |

### 5.2 Business Rules

| ID | Rule |
|----|------|
| BRL-001 | A job listing is considered a duplicate if its URL matches a previously seen URL for the same user |
| BRL-002 | Fit scores use a hybrid approach: 70% AI score + 30% keyword score when both are available |
| BRL-003 | When no AI provider is available (including permanent quota exhaustion), the system falls back to keyword-based matching with a concise user-facing message (no hard dependency on AI) |
| BRL-004 | Salary values are normalised to annual GBP: daily rates x 230 working days, hourly rates x 1,840 working hours |
| BRL-005 | AI provider routing follows 3 tiers: Premium (complex generation), Standard (structured analysis), Light (simple extraction) |
| BRL-006 | Company career pages with JS-rendered SPAs that cannot be scraped are marked as "dead" and removed from all user configs |
| BRL-007 | Every company in the seed data must have both an industry tag and either an ATS provider or explicit null (no missing data) |
| BRL-008 | Generated documents are stored as plain text with DOCX/PDF export on demand |
| BRL-009 | User sessions expire after 24 hours (or 30 days with "remember me") |
| BRL-010 | Admin users (role: root) can access all user data; standard users can only access their own data |
| BRL-011 | Ollama is a shared infrastructure service requiring no per-user provisioning (see BR-025) |
| BRL-012 | User API keys are encrypted with Fernet before storage, using an `ENC:` prefix to distinguish encrypted values from legacy plaintext keys |
| BRL-013 | Token budget enforcement checks daily cumulative `total_tokens` from `AIPromptLog` against the per-user limit; budget resets at UTC midnight |
| BRL-014 | Structured security log events use an `event_type` field for Loki/Grafana filtering: `auth_login_success`, `auth_login_failure`, `auth_logout`, `auth_register`, `auth_sandbox_login`, `auth_password_change`, `auth_session_expired`, `rbac_denial`, `token_budget_warning`, `token_budget_exceeded`, `ai_call_complete`, `key_resolution` |
| BRL-015 | Sandbox users are identified by browser fingerprint (canvas + screen + timezone SHA-256); daily usage tracked per fingerprint and per IP with configurable limits |
| BRL-016 | Adzuna job results use affiliate redirect URLs (`redirect_url` field) for commission tracking; outbound clicks are logged as URL hashes (not full URLs) to preserve privacy |
| BRL-017 | Onboarding career pages endpoint merges admin-default and global career page configs, deduplicating by company name (case-insensitive) |
| BRL-018 | Password reset tokens expire after a configurable period; token validation is required before allowing password change |
| BRL-019 | Salary estimates carry an `employment_type` flag (`Contract` or `Permanent`) to distinguish rate-based vs salaried compensation |
| BRL-020 | Session tokens are hashed with SHA-256 before database storage; lookup uses the hash, not the plaintext token |
| BRL-021 | Display name defaults to the username at registration; no separate Display Name field is collected during onboarding |
| BRL-022 | AI model selection per provider is persisted to the `config` table as a JSON value under the key `provider_model_preferences`; model validation checks against `PROVIDER_MODELS` hardcoded dict |
| BRL-023 | Service control endpoints (`/api/service/*`) require admin (root) access via route-level `Depends(require_root)`. Sandbox users are additionally blocked by middleware (`_is_sandbox_restricted`). Standard users receive 403 at the route level |
| BRL-024 | JWT access tokens encode user ID, username, is_sandbox, is_admin, and provider claims; sandbox tokens exclude email and display_name for privacy |
| BRL-025 | Refresh tokens are rotated on each use: the old token is revoked and a new one issued, preventing token reuse attacks |
| BRL-026 | OAuth users have NULL password_hash/salt columns; authentication for OAuth users is handled exclusively via provider tokens |
| BRL-027 | Per-session sandbox users are identified by provider="sandbox" and is_sandbox=True; expired sessions are purged by scheduled maintenance |
| BRL-028 | Account migration (sandbox→real user) transfers all 14 user-scoped tables in dependency order, then deletes the sandbox user record |
| BRL-029 | Sandbox users do not auto-see advanced tabs (Monitoring, Prompt Lab); they must enable Advanced Mode like standard users. Only admin (root) users see advanced tabs by default |
| BRL-030 | All AI scoring prompts include fairness guardrails instructing the model to evaluate based on skills, experience, and qualifications only — not name, age, gender, ethnicity, or other protected characteristics |
| BRL-031 | AI confidence scores (0-100) are displayed alongside every AI-generated result to indicate the model's certainty; low-confidence results are flagged for user review |
| BRL-032 | The "Challenge This Score" feature triggers a fresh AI re-evaluation with explicit reasoning, using a different prompt variation to reduce single-evaluation bias |
| BRL-033 | Training providers are categorised as free (no payment required for core content) or paid (subscription or per-course fee required), with provider URLs and certification details stored in seed data |
| BRL-034 | Cross-user data isolation uses `userStorage` (namespaced `cviper:u:<userId>:<key>`) for all per-user localStorage values; `purgeAllUserScopedStorage()` wipes all namespaced keys plus legacy unprefixed keys on user switch |
| BRL-035 | API contract tests validate response shape (required keys, value types) against a shared schema file; contract violations fail CI as a hard gate |

### 5.3 Compliance and Regulatory Needs

| Area | Requirement |
|------|-------------|
| **Data Protection (UK GDPR)** | Personal data (CV text, application history) stored in local SQLite database under user control; no third-party data sharing except via user-initiated AI API calls. AI prompt logs (system_prompt, user_prompt, response_text) encrypted at rest with Fernet. IP addresses anonymised in request logs (last octet/segment zeroed). |
| **Data Subject Rights** | GDPR Article 15/20: data export via `GET /api/gdpr/export` (comprehensive JSON of all user data, rate-limited 2/hour). Article 17: right to erasure via `DELETE /api/gdpr/delete-account` (password-verified, cascading deletion of all user data). |
| **Consent Management** | Granular consent tracking via `UserConsent` table. Consent banner on first visit. Record/withdraw consent via `/api/gdpr/consents` endpoints. Privacy Policy page accessible from sidebar and config tab. |
| **Data Retention** | AI prompt logs and search history archived and purged after 180 days via scheduled maintenance. Archive files deleted after 30 days. |
| **AI Transparency** | All AI-generated content clearly labelled with provider and model; prompt logs retained for audit |
| **Credential Security** | API keys stored encrypted at rest (Fernet + master key with rotation support); passwords hashed with bcrypt + per-user salt; session tokens are cryptographically random; per-user keys isolated in DB with `ENC:` prefix |
| **Account Lockout** | 10 failed login attempts within 15 minutes triggers temporary account lockout (HTTP 429). Failed attempts tracked per-username, case-insensitive. Lockout clears on successful authentication. |
| **Rate Limiting** | Enforced at application level to prevent abuse and respect external API quotas |
| **Audit Trail** | Admin actions logged in AdminAuditLog table with old/new values for all mutation endpoints (user CRUD, defaults, data cleanup, DB edits); AI prompts logged in AIPromptLog table; security events (auth, RBAC, token budget) emitted as structured JSON logs for Grafana/Loki alerting |
| **Production Enforcement** | `MASTER_KEY` environment variable required in production (startup fails without it). Warning logged if `SECURE_COOKIES` is not enabled. |
| **Cross-Border Data** | Google Fonts self-hosted to eliminate cross-border data transfers. CSP headers updated to disallow external font sources. |
| **Incident Response** | Documented breach response plan with 72-hour ICO notification window. See `docs/INCIDENT_RESPONSE_PLAN.md`. |

---

## 6. Current vs. Future State

### 6.1 Current Process Overview (Without CViper)

Without CViper, professionals manually search 5-10 job boards individually, tailor each CV (30-60 min per role), track applications in spreadsheets, and research salaries and interview prep separately. This typically consumes **8-15 hours per week** to produce **3-5 tailored applications** with an **~25% ATS pass rate**.

### 6.2 Pain Points

| Pain Point | Impact | Severity |
|-----------|--------|----------|
| Searching multiple boards individually | 2-3 hours wasted per search session | High |
| Manual CV tailoring per application | 30-60 minutes per role | High |
| No ATS optimisation | 75% of CVs rejected before human review | Critical |
| Fragmented tracking (spreadsheets) | Lost applications, missed follow-ups | Medium |
| No salary benchmarking | Accepting below-market offers | Medium |
| No skills gap visibility | Applying to unsuitable roles | Medium |
| No interview prep support | Underprepared for specific roles | Low |

### 6.3 Target Future Process (With CViper)

```
CViper-Powered Workflow:
1. Configure search once (boards, career pages, keywords, location)
2. Run single search — CViper queries 15+ boards and 80+ career pages
3. AI scores all results against CV (0-100 fit score)
4. Review top matches, save interesting roles
5. Click "Apply" — CViper generates tailored CV + cover letter in seconds
6. ATS score shown immediately; iterate if needed
7. Track status via built-in application manager
8. AI provides salary estimates, skills gap analysis, interview prep
9. Dashboard shows career insights and trending skills

Total time per week: 2-3 hours
Applications per week: 15+ tailored
ATS pass rate: >60%
```

---

## 7. High-Level Process Flows

### 7.1 End-to-End Job Search & Application Workflow

```
                                    CViper Platform
                                    ===============

[User] ──> Upload CV ──> [CV Analysis] ──> Skills/Titles/Experience extracted
                              |
                              v
[User] ──> Configure Search ──> [Multi-Source Search]
              |                      |
              |    ┌─────────────────┼─────────────────┐
              |    v                 v                  v
              | [Job Boards]   [Career Pages]    [Saved Profiles]
              |  Reed, Indeed   Workday, GH       Reusable configs
              |  LinkedIn...    Lever, SR...
              |    |                |                   |
              |    └────────────────┼───────────────────┘
              |                     v
              |              [Result Aggregation]
              |              Deduplication + Normalisation
              |                     |
              v                     v
[User] <── Review Results <── [AI Fit Scoring]
              |                 0-100 score + breakdown
              |
              v
[User] ──> Save Jobs ──> [Application Tracker]
              |              Status: Not Applied → Applied → Interviewing → Offer/Rejected
              |
              v
[User] ──> Apply ──> [Document Generation]
              |          Tailored CV + Cover Letter
              |          ATS Score + Feedback
              |
              v
[User] ──> Interview Prep ──> [AI Insights]
              |                   Likely questions, company research
              |                   Salary analysis, red/green flags
              |
              v
[User] <── Career Dashboard <── [Analytics]
                                  Skills gap, trending skills
                                  Portfolio review, market insights
```

### 7.2 AI Provider Routing Swimlane

```
┌──────────┐  ┌──────────────┐  ┌──────────────┐  ┌───────────────┐
│  User    │  │  AIService   │  │  TaskRouter  │  │  AIGateway    │
│  Action  │  │  (Facade)    │  │  (Selection) │  │  (Dispatch)   │
└────┬─────┘  └──────┬───────┘  └──────┬───────┘  └───────┬───────┘
     │               │                 │                   │
     │ Score Job     │                 │                   │
     │──────────────>│                 │                   │
     │               │ select_provider │                   │
     │               │ ("match_job")   │                   │
     │               │────────────────>│                   │
     │               │                 │ Check user pref   │
     │               │                 │ Check tier config  │
     │               │                 │ Check availability │
     │               │  provider_id    │                   │
     │               │<────────────────│                   │
     │               │                 │                   │
     │               │ call(provider,  │                   │
     │               │  prompt, sys)   │                   │
     │               │─────────────────────────────────────>│
     │               │                 │                   │ Try primary
     │               │                 │                   │ provider
     │               │                 │                   │────┐
     │               │                 │                   │    │ If fail:
     │               │                 │                   │<───┘ try next
     │               │  AI response    │                   │      in chain
     │               │<─────────────────────────────────────│
     │               │                 │                   │
     │               │ Parse + Score   │                   │
     │  Fit Score    │                 │                   │
     │<──────────────│                 │                   │
     │  (0-100)      │                 │                   │
```

### 7.3 Document Generation Flow

```
[User selects job] ──> [Apply Button]
                            |
                            v
                    [Load CV Profile]
                    CV text + skills + experience
                            |
                            v
                    [Load Job Details]
                    Description + requirements + company
                            |
                    ┌───────┴───────┐
                    v               v
            [Generate CV]    [Generate Cover Letter]
            AI rewrites CV    AI personalises letter
            for specific     with matched skills
            role/company     and company research
                    |               |
                    v               v
            [ATS Score CV]   [Save Documents]
            Check keyword    TXT + DOCX + PDF
            coverage         in output folder
                    |
                    v
            [Return Results]
            Score + documents + feedback
```

### 7.4 Monitoring & AI Runtime Flows

Infrastructure-level sequence diagrams for Grafana monitoring and Ollama local AI runtime are detailed in the FSD (Sections 5.5 and 5.6). At a high level:

- **Grafana**: Auto-deployed via Docker Compose / Azure Bicep with preconfigured Infinity datasource and `cviper-unified` dashboard. Pulls live metrics from backend `/api/monitoring/*` endpoints.
- **Ollama**: Auto-deployed with model download on first start. Backend detects availability, registers as an AI provider, and routes Light-tier tasks (or any tier if user-configured) to Ollama via OpenAI-compatible API.

---

## 8. Cost, Benefit & Impact Analysis

### 8.1 Expected ROI

| Cost Category | Monthly Estimate |
|---------------|-----------------|
| AI Provider API Costs (free tiers) | $0 (Google Gemini free tier: 15 RPM) |
| AI Provider API Costs (paid usage) | $5-20 (at ~500 AI calls/month) |
| Ollama (local inference) | $0 (runs on existing hardware; no API fees) |
| Grafana (monitoring) | $0 (open-source; runs in Azure Container Apps) |
| Azure Container Apps (backend + frontend) | $7-23/month (scale-to-zero) |
| Azure Database for PostgreSQL (Burstable B1ms) | ~$11/month |
| Azure Container Registry (Basic) | ~$4/month |
| Azure Container Apps (Ollama, demo-only) | $0-60/month (scaled to 0 when not demoing) |
| Azure Container Apps (Grafana + Prometheus + Loki) | $5-15/month (scale-to-zero) |
| Development (open source) | $0 |
| **Total Monthly Cost (Ollama off)** | **$27-53** |
| **Total Monthly Cost (Ollama on)** | **$67-113** |

| Benefit | Value |
|---------|-------|
| Time saved per week | 6-12 hours |
| Additional applications per week | 10-12 more tailored applications |
| Improved ATS pass rate | 35% improvement (25% to 60%) |
| Salary insight for negotiations | Access to curated + AI-estimated benchmarks |
| Career trajectory clarity | Skills gap + trending skills analysis |

### 8.2 Operational Impact

- **Positive**: Unified workflow replaces 5+ separate tools (job boards, spreadsheets, word processors, salary sites, interview prep resources)
- **Positive**: AI failover means no single-provider dependency
- **Positive**: Local Ollama option enables fully offline operation
- **Consideration**: Users need to manage AI API keys (mitigated by free tiers and clear onboarding)

### 8.3 Change Management Considerations

| Change | Impact | Mitigation |
|--------|--------|------------|
| Shift from manual to AI-assisted CV writing | Users may distrust AI output initially | ATS scoring provides objective quality feedback; users can edit all generated content |
| Centralised application tracking | Users must migrate from spreadsheets | CSV/XLSX export maintains interoperability |
| Multi-provider AI configuration | Initial setup complexity | Default provider (Google Gemini free tier) works out of the box |

---

## 9. Risks & Mitigations

| ID | Risk | Probability | Impact | Mitigation |
|----|------|------------|--------|------------|
| R-001 | AI provider API changes or pricing increases | Medium | High | Support 8+ providers with automatic failover; keyword-only mode as ultimate fallback |
| R-002 | Job board/ATS API access restricted | Medium | Medium | Modular handler architecture allows adding/removing sources without code changes; seed data regularly validated via tests |
| R-003 | AI hallucination in generated documents | Low | High | ATS scoring validates keyword coverage; users review all output before submission; prompt injection prevention in place |
| R-004 | Database performance under concurrent load | Low | Low | **Mitigated**: Migrated from SQLite to Azure Database for PostgreSQL Flexible Server (Burstable B1ms) with managed backups, HA, and connection pooling. SQLite retained for local development only; CI uses PostgreSQL 16 service container for schema drift validation. Alembic migrations use `batch_alter_table()` for SQLite compatibility in local dev. |
| R-005 | Data loss | Low | Medium | **Mitigated**: Azure PostgreSQL provides automated daily backups with 7-day retention, point-in-time restore, and geo-redundant storage. Admin backup endpoint retained for on-demand exports. |
| R-006 | Security breach (API key exposure) | Low | Critical | Keys encrypted at rest; never transmitted to frontend; masked in API responses; SSRF prevention; rate limiting |
| R-007 | GDPR non-compliance | Low | High | All data stored locally under user control; no third-party data sharing; admin audit logging |
| R-008 | Career page scraping breaks (HTML changes) | High | Low | ATS handler architecture isolates changes; grey-box detection and SPA diagnostics; "dead company" pruning |
| R-009 | Free-tier AI rate limits exceeded | Medium | Low | Rate limiting at application level; permanent quota error detection (`_is_permanent_quota_error()`) skips retries on exhausted quotas; circuit breaker `force_open()` immediately disables exhausted providers; automatic provider rotation; sandbox users see concise fallback message ("Sandbox AI is temporarily busy — using keyword matching instead.") |
| R-010 | Ollama model download fails on first startup (network/registry issues) | Low | Medium | Retry logic on container startup; fallback to cloud providers; pre-built Docker image with model baked in for air-gapped deployments |
| R-011 | Grafana container fails to start or datasource misconfigured | Low | Low | Health check on Grafana container; backend gracefully degrades (monitoring panel shows "Grafana not configured" banner); automated provisioning via API on startup |
| R-012 | Ollama resource exhaustion (CPU/RAM on small instances) | Medium | Medium | Configurable model size (default llama3.2 ~2GB); Azure Container App allocates 2 vCPU / 4 GiB; TaskRouter assigns only Light-tier tasks to Ollama by default |
| R-013 | Grafana exposed without authentication in production | Low | High | Default admin password set via GRAFANA_ADMIN_PASSWORD env var; production deployments enforce password change; Azure Container Apps internal networking for Prometheus/Loki |
| R-014 | Master encryption key lost or corrupted | Low | Critical | Master key stored in `.secrets/master.key` with restricted file permissions; `rotate_master_key()` function supports safe re-encryption of all user keys; key decryption failures trigger Grafana critical alert |
| R-015 | Single user exhausts shared AI quota | Medium | Medium | Per-user daily token budgets enforced at gateway level; configurable via `USER_DAILY_TOKEN_LIMIT`; root users exempt; budget exceeded events trigger Grafana warning alert |
| R-016 | Brute force login attacks | Medium | High | Rate limiting on login endpoint (5/min); structured `auth_login_failure` events feed Grafana alert rule (>10 failures in 5 min = critical alert); account lockout after 10 failed attempts in 15 minutes |
| R-017 | Sandbox account abuse (excessive AI usage) | Medium | Medium | 5-layer prevention: fingerprint tracking, per-FP daily limits (50/day), per-IP daily limits (100/day), dedicated sandbox providers (sandbox_google 1000 req/day, sandbox_openrouter 50 req/day), AI response truncation, 1-hour session auto-expiry. Permanent quota errors on sandbox providers trigger immediate circuit breaker open and graceful fallback to keyword matching with a user-facing message |
| R-018 | Adzuna API availability or rate limits | Low | Low | Adzuna is one of 15+ job sources; if unavailable, search continues with remaining boards; no hard dependency |
| R-019 | OAuth provider service disruption (LinkedIn/Google outage) | Low | Low | Username/password and sandbox login remain available; OAuth is additive, not required |
| R-020 | JWT refresh token stolen via XSS | Low | High | Refresh token stored in httpOnly cookie (inaccessible to JS); access token never in localStorage; short 15-min expiry limits exposure window |
| R-021 | Standard user exploits unprotected service control endpoints to cause denial of service | Low | Critical | **Mitigated** (v2.1): Service control endpoints now enforce `require_root` at route level; RBAC test coverage verifies 403 for non-root users |

---

## 10. Glossary

| Term | Definition |
|------|-----------|
| **ATS** | Applicant Tracking System — software used by companies to manage job applications (e.g., Workday, Greenhouse, Lever) |
| **AuthContext** | A React context provider that centralises JWT token management, OAuth callback handling, and authentication state for all frontend components |
| **Fit Score** | A 0-100 rating indicating how well a job matches a candidate's CV profile |
| **CV Score** | A 0-100 rating indicating how well a CV is optimised for a specific job's ATS system |
| **Career Page** | A company's dedicated jobs/careers website, often powered by an ATS platform |
| **Grey Box** | A UI state where a career page is configured but returns no job listings |
| **Dead Company** | A career page entry that consistently fails to return data and has been removed from all configs |
| **Provider** | An AI service (OpenAI, Anthropic, Google Gemini, etc.) used for analysis and generation |
| **Tier Routing** | The system's 3-level AI task classification: Premium (complex), Standard (structured), Light (simple) |
| **Fallback** | Keyword-based matching used when no AI provider is available |
| **Seed Data** | Curated initial data (company boards, salary benchmarks) loaded on first run |
| **Silent Refresh** | The process of automatically renewing a JWT access token before it expires, using the httpOnly refresh cookie, without user interaction |
| **Search Profile** | A saved, reusable set of search parameters (criteria, boards, keywords) |
| **Deep Analysis** | A consolidated AI analysis covering salary, culture fit, and interview insights for a specific job |
| **Portfolio Review** | An AI analysis of all tracked jobs to identify career patterns and rejection trends |
| **Prompt Lab** | A feature for comparing AI responses across providers using the same prompt |
| **SSRF** | Server-Side Request Forgery — an attack where the server is tricked into making requests to internal resources |
| **WAL Mode** | Write-Ahead Logging — an SQLite journaling mode that allows concurrent reads during writes (local development only; production uses PostgreSQL) |
| **OAuth2** | An authorization framework allowing users to sign in via third-party providers (LinkedIn, Google) without sharing their password with CViper |
| **Ollama** | An open-source local AI inference server for running LLMs on-device |
| **Token Budget** | A per-user daily limit on AI token consumption, enforced at the AIGateway before dispatching calls |
| **Master Key** | A Fernet symmetric encryption key used to encrypt/decrypt user API keys at rest |
| **Key Rotation** | The process of re-encrypting all stored user API keys with a new master key |
| **RBAC** | Role-Based Access Control — enforced via router-level FastAPI dependency (`require_root`) |
| **Refresh Token Rotation** | A security practice where refresh tokens are single-use: each token refresh revokes the old token and issues a new one |
| **Circuit Breaker** | A pattern that skips failing AI providers temporarily to avoid cascading failures; supports `force_open()` to immediately open the circuit on permanent quota exhaustion without waiting for the failure threshold |
| **Event Type** | A structured log field (`event_type`) used by Grafana/Loki to filter and alert on security events |
| **Advanced Mode** | A user preference that shows/hides developer-oriented tabs (Monitoring, Prompt Lab) in the navigation bar |
| **Onboarding Stepper** | A 4-step progress indicator (Upload CV → Review Profile → Find Jobs → Apply) guiding users through the core workflow |
| **Empty State** | A UI pattern shown when a tab or section has no data, providing contextual guidance and call-to-action buttons |
| **Sub-Navigation** | Secondary tab navigation within a page (e.g., Settings sub-tabs: General, AI Providers, Search & Keywords, Preferences) |
| **Sandbox Account** | A per-session public trial account with restricted AI access (sandbox_google and sandbox_openrouter only), usage tracking via fingerprint and IP, truncated AI responses, no delete operations, no admin/service control access, and no auto-visibility of advanced tabs |
| **Browser Fingerprint** | A SHA-256 hash derived from canvas rendering, screen dimensions, and timezone — used to identify unique sandbox visitors without cookies |
| **Adzuna** | A job search API integrated as an affiliate source; job links use redirect URLs for commission tracking |
| **Affiliate Click** | An outbound click on a job listing that uses an affiliate redirect URL; tracked via `POST /api/track-click` with URL hashes |
| **Employment Type** | A classification on salary estimates distinguishing Contract (day/hourly rate) from Permanent (annual salary) positions |
| **PgBouncer** | A lightweight PostgreSQL connection pooler used in Azure production to reduce connection overhead (port 6432) |
| **OpenTelemetry** | An observability framework providing distributed tracing across FastAPI, SQLAlchemy, and httpx calls |
| **GZip Middleware** | FastAPI middleware compressing JSON responses to reduce bandwidth usage |
| **Hybrid JWT Auth** | An authentication pattern combining short-lived access tokens (in memory) with long-lived refresh tokens (httpOnly cookies) for security and seamless page refresh recovery |
| **Progressive Rendering** | A UI pattern that renders the first 100 search results immediately with a "Show more" button for the remainder |
| **Password Reset Token** | A time-limited token sent via email to verify identity during the forgot-password flow |
| **Per-Session Sandbox** | A sandbox mode where each "Try Demo" click creates a unique user with fresh seed data, rather than sharing a single sandbox account |
| **Safe Area Insets** | CSS environment variables (`env(safe-area-inset-*)`) used to pad content away from device hardware features (notch, home indicator, rounded corners) on iPhones and similar devices |
| **Dynamic Viewport Height (dvh)** | A CSS unit (`100dvh`) that accounts for mobile browser chrome (address bar, toolbar) resizing, unlike `100vh` which uses the initial viewport size |
| **PROVIDER_MODELS** | A hardcoded dictionary mapping each AI provider ID to its available model list (id, name, tier), used for model selection validation and dropdown population |
| **Admin CLI Tools** | Standalone Python scripts (`reset_password.py`, `diagnose.py`) that operate directly against the database for administrative tasks without requiring the running web server |
| **CV Optimisation Pipeline** | A multi-step workflow for improving CV quality: keyword injection (adding missing ATS keywords), format validation (checking structure), and one-click optimise (automated tailoring for a specific job) |
| **Training Provider** | An online learning platform (e.g., freeCodeCamp, Udemy, Pluralsight) offering courses and certifications mapped to skills tracked by the platform |
| **Fairness Guardrails** | Prompt-level instructions that prevent AI models from using protected characteristics (name, age, gender, ethnicity) when scoring or evaluating candidates |
| **Confidence Score** | A 0-100 rating indicating the AI model's certainty in its output; displayed alongside fit scores, salary estimates, and other AI-generated results |
| **Challenge This Score** | A user-initiated action that triggers a fresh AI re-evaluation of a score with explicit reasoning, using a different prompt variation |
| **AI Transparency Disclosure** | An in-app notice explaining how AI-generated scores and recommendations are produced, which provider/model was used, and the limitations of AI analysis |
| **userStorage** | A frontend utility that namespaces all per-user localStorage values under `cviper:u:<userId>:<key>` to prevent cross-user data leakage between accounts |
| **API Contract Test** | A test that validates the response shape (required keys, value types, structure) of an API endpoint against a shared schema, ensuring backward compatibility |
| **Plausible Analytics** | A privacy-respecting web analytics platform that tracks page views and usage patterns without cookies or personal data collection |
| **PWA Install Prompt** | A browser-native prompt that allows users to install the web application as a standalone app on their device's home screen |
| **Open Graph Tags** | HTML meta tags (`og:title`, `og:description`, `og:image`) that control how a page appears when shared on social media platforms |

---

*End of Business Requirements Document*
