# CViper End-to-End Test Plan

## CViper - AI-Powered Job Search & Application Platform

---

| Field | Value |
|-------|-------|
| **Document ID** | TP-CVIPER-001 |
| **Version** | 0.6.0 |
| **Status** | Active |
| **Author** | CViper Project Team |
| **Date** | 2026-04-23 |
| **Classification** | Internal |
| **Owner Persona** | QAE |
| **Related** | Testing-Strategy-and-Architecture.md (the *how*) |

### Version History

| Version | Date | Changes |
|---------|------|---------|
| 0.6.0 | 2026-04-23 | **CV Optimisation Pipeline journeys** — Bullet Quality Scorer (on CV Analysis tab, scores every bullet deterministically), one-click "Optimise for job" modal (Health + Keywords + Bullets + Summary tabs), ATS Format Validator (6 categories / 23 checks). **New regression scenarios documented**: OpenRouter slug retirement fallback to safety-net list (LESSON-049), read-read parity for every pair of GETs exposing the same field (LESSON-043), state-drift prevention for every (DB Config ↔ in-memory) pair at startup (LESSON-048), JSX `\uXXXX` literal leak render-walk (LESSON-046), inline Missing-field editor on Applications (CV-219), Simple/Advanced AI mode toggle (CV-230), regional regulatory awareness in CV tailoring (CV-207), cloud-user CV text resolver (LESSON-045). Coverage closed for CV-207 through CV-233. Commit: 9534f137. |
| 0.5.5 | 2026-04-21 | Stamped to match app v0.5.5 (doc version had drifted to 0.5.1 while app advanced through 0.5.2, 0.5.3, 0.5.4). New regression scenarios documented: AI Priority reorder preserves hidden entries (Rule #36 / LESSON-035), cloud-mode search auto-resolves saved profile / CV-analysis when no cv_folder is provided (Rule #37 / LESSON-037), admin cross-user job update parity, ConfirmDialog replaces browser alert/confirm/prompt across the app. `@smoke` golden-path spec running against live demo added. User-scope parity framework (CV-197, Rule #24) documented as generative coverage framework rather than per-endpoint tests. Commit: 11778d0b. |
| 0.5.1 | 2026-04-14 | Aligned with app v0.5.1. Added formal test plans for Search (35 cases), Applications (38), Registration (19), Settings (16), Companies (14), Documents (13). Added cross-cutting suites: accessibility, responsive, GDPR, AI failover, cross-browser, localisation, performance, PWA. Total E2E specs: 106. RTM created. |
| 0.1.0 | 2026-04-11 | Initial test plan — 7 journeys, 48 scenarios, 100% E2E coverage. |

---

## Purpose & Scope

This document defines **what user journeys CViper tests** — the functional
coverage targets, regression risk areas, and test type requirements for each
journey. It is the canonical answer to "do we test X?".

The companion document `Testing-Strategy-and-Architecture.md` covers the
**how** (pytest, Vitest, Playwright, CI pipeline structure). This Test Plan
covers the **what** (which user journeys, in what depth, with which test
types).

This document is enforced by:

- **QAE persona** — applies the Test Design Checklist (CLAUDE.md) to every
  PR and produces a 7-row coverage matrix that references this plan
- **Auto-correction rule #26** — blocks code changes that touch a journey
  here without negative + boundary tests
- **PR template** — Test Design Coverage section requires reviewers to
  confirm which journey(s) the PR affects
- **`update docs` command** — flags this doc as stale when workflows change

---

## 1. Core User Journeys

These are the primary flows a real user takes. Every release MUST verify
all of them. Each journey lists its required test types.

### 1.1 Login & Authentication

| # | Scenario | Type | Status |
|---|---|---|---|
| 1.1.1 | User enters valid credentials → redirected to dashboard | Happy-path | Required |
| 1.1.2 | Invalid credentials → correct error message | Negative | Required |
| 1.1.3 | Session timeout → user logged out cleanly | Edge case | Required |
| 1.1.4 | **Switching users → previous user's data fully cleared** | Regression (LESSON-029) | Critical |
| 1.1.5 | Try Demo (sandbox) login → no fetch-storm 401 logout | Regression (LESSON-028) | Critical |
| 1.1.6 | OAuth redirect (Google / Microsoft / LinkedIn) | Happy-path + Edge case | Required |
| 1.1.7 | Account lockout after N failed attempts | Boundary | Required |

### 1.2 Uploading a CV

| # | Scenario | Type | Status |
|---|---|---|---|
| 1.2.1 | Upload supported file (PDF, DOCX) | Happy-path | Required |
| 1.2.2 | Upload at max size (e.g., 10 MB) — and at 9.9 / 10.1 MB | Boundary | Required |
| 1.2.3 | Upload unsupported file type (.exe, .zip) → clear error | Negative | Required |
| 1.2.4 | Upload corrupted file → safe failure, no crash | Negative | Required |
| 1.2.5 | Upload extremely large CV (100+ pages) | Edge case | Required |
| 1.2.6 | Upload empty file (0 bytes) | Negative | Required |
| 1.2.7 | Upload CV with images only (no text) | Edge case | Required |

### 1.3 AI CV Analysis

| # | Scenario | Type | Status |
|---|---|---|---|
| 1.3.1 | CV processed and results displayed correctly | Happy-path | Required |
| 1.3.2 | Analysis timeouts handled gracefully | Edge case | Required |
| 1.3.3 | AI provider error → fallback to keyword analysis | Negative + Regression | Critical |
| 1.3.4 | **Switching users → previous user's analysis hidden** | Regression (LESSON-029) | Critical |
| 1.3.5 | Multi-provider comparison shows distinct results | Happy-path | Required |
| 1.3.6 | Results survive page refresh (persistence) | Edge case | Required |

### 1.4 Editing & Improving CV

| # | Scenario | Type | Status |
|---|---|---|---|
| 1.4.1 | User edits sections and saves changes | Happy-path | Required |
| 1.4.2 | AI suggestions appear correctly | Happy-path | Required |
| 1.4.3 | Undo / redo works | Edge case | Recommended |
| 1.4.4 | Invalid edits handled safely (XSS, very long text) | Negative + Boundary | Required |
| 1.4.5 | Concurrent edits across tabs / devices | Edge case | Recommended |

### 1.5 Exporting the CV

| # | Scenario | Type | Status |
|---|---|---|---|
| 1.5.1 | Export to PDF | Happy-path | Required |
| 1.5.2 | Export to DOCX | Happy-path | Required |
| 1.5.3 | Export with missing fields → clear validation | Negative | Required |
| 1.5.4 | Export on mobile (touch + responsive layout) | E2E + Edge case | Critical |
| 1.5.5 | Export Close + Download buttons always visible (rule #25) | Regression | Critical |
| 1.5.6 | Export retains formatting (regression for prior bugs) | Regression | Critical |

### 1.6 Job Application Tracker

| # | Scenario | Type | Status |
|---|---|---|---|
| 1.6.1 | Add a job (manual + via search) | Happy-path | Required |
| 1.6.2 | Edit a job | Happy-path | Required |
| 1.6.3 | Delete a job | Happy-path | Required |
| 1.6.4 | Status updates persist | Happy-path | Required |
| 1.6.5 | Sorting and filtering | Happy-path + Edge case | Required |
| 1.6.6 | Data persists across sessions | Regression | Required |
| 1.6.7 | Delete a non-existent job → clean 404 | Negative | Required |
| 1.6.8 | **Switching users → only own jobs visible** | Regression (LESSON-029) | Critical |

### 1.7 Job Search

| # | Scenario | Type | Status |
|---|---|---|---|
| 1.7.1 | Search returns results from enabled boards | Happy-path | Required |
| 1.7.2 | SSE streaming progress works | Happy-path | Required |
| 1.7.3 | All boards disabled → clear empty state | Negative | Required |
| 1.7.4 | Provider failure → other providers continue | Edge case | Required |

---

## 2. Test Type Coverage Requirements

Every journey above must consider all 7 approaches from the Test Design
Checklist (CLAUDE.md). Use this matrix to score each release:

| Approach | Required for | Notes |
|---|---|---|
| **Happy-path** | All 7 journeys | The main expected workflow |
| **Negative** | All 7 journeys | Invalid input, wrong state, unauthorised |
| **Boundary** | 1.2 (file size), 1.4 (text length), 1.6 (job count), 1.1.7 (lockout) | Limits and thresholds |
| **Edge case** | 1.1.3, 1.2.5–1.2.7, 1.3.2, 1.5.4, 1.7.4 | Rare but possible scenarios |
| **E2E** | All Scenarios in §4 | Full multi-step workflows |
| **Regression** | All "Critical" rows above | Each tied to a LESSON-NNN or rule # |
| **Exploratory** | Pre-release for major features | Manual, document in `ClaudeReports/` |

---

## 3. Critical Regression Areas

These have caused incidents historically. They MUST run on every deploy
gate (tagged `@critical-regression` in Playwright specs).

| Area | Linked lesson / rule | Scenario IDs |
|---|---|---|
| User switching (cross-user data leak) | LESSON-029, LESSON-030 | 1.1.4, 1.3.4, 1.6.8 |
| Try Demo / fetch-storm 401 logout | LESSON-028, rule #19 | 1.1.5 |
| Post-login bearer token attachment | LESSON-028, rule #22 | 1.1.1 (any auth handler) |
| Export formatting | LESSON-024 (showcase SVGs) | 1.5.6 |
| Mobile UI alignment | rule #23 (floating elements) | 1.5.4, 1.7.* |
| AI prompt regressions / fallback | LESSON-022 (sandbox providers) | 1.3.3 |
| Job tracker data persistence | rule #24 (per-user scoping) | 1.6.6, 1.6.8 |
| Essential controls always visible | rule #25 | 1.5.5 |
| PWA installability | (when enabled) | TBD |

---

## 4. End-to-End Scenarios (Full Workflows)

These compose multiple journeys into realistic user sessions. Each
scenario has a Playwright spec (or a backlog item if missing).

### Scenario 1 — First-time user
1. Register → 2. Login → 3. Upload CV → 4. Run analysis →
5. Edit CV → 6. Export → 7. Add job to tracker → 8. Logout

### Scenario 2 — Returning user
1. Login → 2. View previous CV → 3. Make edits →
4. Export again → 5. Update job applications → 6. Logout

### Scenario 3 — Mobile user
1. Login on phone → 2. Upload CV from phone → 3. Analyse →
4. Zoom in / out (UI stability) → 5. Export → 6. Add job →
7. Install PWA (when enabled)

### Scenario 4 — Error-handling workflow
1. Upload invalid file → 2. Upload corrupted file →
3. Trigger AI timeout → 4. Try exporting without a CV →
5. Try adding invalid job data

### Scenario 5 — Cross-user isolation (LESSON-029 regression)
1. Login as User A → 2. Upload CV → 3. Save jobs →
4. Logout → 5. Login as User B (no logout in between OK too) →
6. Verify NO User A data visible anywhere (CV, jobs, profiles, history)

---

## 5. Coverage Tracking

The journey-to-spec mapping lives at `frontend/e2e/test-plan-coverage.md`.
Each row maps a journey scenario above to its Playwright spec (or marks it
as a gap with a backlog item ID).

QAE updates the matrix whenever:
- A new journey is added here
- A new spec file is added under `frontend/e2e/`
- A scenario moves from gap → covered

---

## 6. Platform Coverage

| Platform | Required | Notes |
|---|---|---|
| Desktop Chrome | All scenarios | Primary CI target |
| Desktop Firefox | Smoke + Critical regression | Cross-browser sanity |
| Mobile Chrome (375 px) | All "Mobile" + Critical regression | Real device or emulation |
| Mobile Safari (iOS) | Smoke + Critical regression | When PWA work begins |
| Slow network | Scenario 4 (errors) | Throttled to 3G |
| Offline | PWA scenarios | When PWA work begins |

---

## 7. Maintenance

- **Refresh trigger**: New feature, new user journey, new regression area,
  or new LESSON-NNN with prevention test
- **Stale detection**: `update docs` command checks this against the
  Documentation Registry
- **Owner**: QAE persona (with PDM for cross-cycle tracking)
- **Versioning**: Bump version on any structural change
