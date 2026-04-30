# Requirements Traceability Matrix (RTM)

**Document ID**: RTM-CVIPER-001
**Version**: 0.6.2
**Date**: 2026-04-30
**Status**: Active

> Maps every Business Requirement → Functional Spec → Test Plan Journey → E2E Test Case.
> Provides audit-ready evidence of full requirement coverage.

---

## Traceability Chain

```
BRD Requirement (BR-XXX)
    ↓
FSD Feature (FR-XXX) with Acceptance Criteria
    ↓
Test Plan Journey (1.1–1.7, S1–S5)
    ↓
E2E Scenario (1.1.1–1.7.4) → Playwright .spec.js
    ↓
Formal Test Plan (TC-XXX) → Backend + E2E test cases
```

---

## Core Requirements

| BRD Req | Description | FSD Feature | Test Journey | E2E Spec(s) | Formal Plan | Status |
|---|---|---|---|---|---|---|
| BR-001 | Multi-source job search (15+ boards, 80+ career pages) | FR-001, FR-007 | 1.7 Job Search | `job-search-flow.spec.js`, `search-streaming.spec.js`, `search-formal-plan.spec.js` | TC-KS-01–05, TC-LOC-01–05, TC-SAL-01–04, TC-DATE-01–04 | **Covered** |
| BR-002 | AI-powered CV analysis & skill extraction | FR-002 | 1.2 CV Upload, 1.3 AI Analysis | `cv-upload.spec.js`, `cv-analysis-ai.spec.js`, `cv-analysis-full.spec.js` | TC-CV-01–32 | **Covered** |
| BR-003 | Job-to-CV fit scoring (0-100) | FR-003 | 1.3 AI Analysis | `cv-analysis-ai.spec.js`, `job-scoring.spec.js` | — | **Covered** |
| BR-004 | Tailored CV & cover letter generation | FR-004 | 1.4 CV Editing, 1.5 CV Export | `cv-editing-export.spec.js`, `cv-export-download.spec.js`, `documents-formal-plan.spec.js` | DOC-01–45 | **Covered** |
| BR-005 | Application status tracking | FR-005 | 1.6 Job Tracker | `job-tracker-crud.spec.js`, `applications-lifecycle.spec.js`, `applications-formal-plan.spec.js` | APP-01–26, E2E-01–04 | **Covered** |
| BR-006 | Multi-provider AI with failover | FR-006 | 1.3 AI Analysis | `ai-failover.spec.js` | — | **Covered** |
| BR-007 | Salary estimates via benchmarks & AI | FR-008 | — | `company-salary.spec.js`, `companies-formal-plan.spec.js` | SAL-01–45 | **Covered** |
| BR-010 | Multi-user accounts with RBAC & OAuth2 | FR-027 | 1.1 Login & Auth | `login-registration-full.spec.js`, `auth-register.spec.js`, `registration-formal-plan.spec.js` | CA-01–92 | **Covered** |
| BR-011 | Rate limiting | — | — | Backend: `test_rate_limiting.py` | CA-42 | **Covered** |
| BR-012 | SSRF prevention | — | — | Backend: `test_url_validation.py` | — | **Covered** |
| BR-016 | Job deduplication | — | 1.7 Job Search | `search-deduplication.spec.js` | — | **Covered** |
| BR-032 | Cross-user data isolation | FR-037 | S5 Cross-User | `cross-user-isolation.spec.js` (LESSON-029/030) | — | **Covered** |
| BR-046 | Sandbox abuse prevention | FR-020, FR-030 | 1.1 Login (Demo) | `smoke.spec.js`, `demo-mode-navigation.spec.js`, `try-demo-fetch-storm.spec.js` | — | **Covered** |
| BR-048 | 7-step onboarding registration | FR-022 | 1.1 Login | `auth-register.spec.js`, `login-registration-full.spec.js` TC-LR-13–17 | CA-70–72, CA-90 | **Covered** |
| BR-062 | Hybrid JWT authentication | FR-027 | 1.1 Login | `login-registration-full.spec.js`, `auth-refresh.spec.js` (real-backend) | CA-50–52 | **Covered** |
| BR-093 | Free / Pro tier system + per-tier daily quotas | FR-093 | 1.6 Job Tracker / Settings | `usage-tracking.spec.js`, `pro-tier-*.spec.js` | — | **Covered** (CV-093, CV-236, CV-244) |
| BR-219 | Inline edit affordance for Unknown Company/Location on Applications | FR-219 | 1.6 Job Tracker | `applications-missing-field.spec.js` | — | **Covered** (CV-219) |
| BR-228 | Cloud-user CV resolver — full-text persistence + 2-step fallback | FR-228 | 1.7 Job Search | `cloud-mode-search-fallback.spec.js` | — | **Covered** (CV-228, LESSON-045, Rule #45) |
| BR-230 | AI Configuration Simple/Advanced mode split | FR-230 | Settings | `ai-config-mode.spec.js` | — | **Covered** (CV-230) |
| BR-234 | AI provider key remove flow without phantom errors | FR-234 | Settings | `ai-keys-remove.spec.js` | — | **Covered** (CV-234) |
| BR-235 | Diagnostic bundle redaction (PII) | FR-235 | Settings / Admin | Backend: `test_diagnostic_redaction.py` | — | **Covered** (CV-235) |
| BR-236 | Pro tier — admin-assigned promotion (Phase 1) | FR-236 | 1.1 Login / Admin | `admin-set-tier.spec.js`, `topnav-pro-badge.spec.js` | — | **Covered** (CV-236, LESSON-054) |
| BR-237 | Style preset on CV tailoring (Conservative / Balanced / Creative) | FR-237 | 1.4 CV Editing | `cv-tailoring-style-preset.spec.js` | — | **Covered** (CV-237) |
| BR-238 | Pro tier — read-time expiry guard | FR-238 | (Backend gate) | Backend: `test_pro_expiry_guard.py` | — | **Covered** (CV-238) |
| BR-239 | Stripe checkout-session endpoint | FR-239 | Settings / Billing | Backend: `test_billing_checkout.py` | — | **Covered** (CV-239) |
| BR-240 | Stripe webhook handler (subscription lifecycle) | FR-240 | (Backend) | Backend: `test_billing_webhook.py` | — | **Covered** (CV-240) |
| BR-241 | Nightly demotion job for expired Pros | FR-241 | (Scheduled) | Backend: `test_pro_demotion_job.py` | — | **Covered** (CV-241) |
| BR-244 | Tier-aware token budgets (Pro 5×, Sandbox 0.5×) | FR-244 | (Cross-cutting) | Backend: `test_tier_token_budget.py` | — | **Covered** (CV-244) |

---

## Non-Functional Requirements

| Requirement | Type | Test Spec(s) | Formal Plan | Status |
|---|---|---|---|---|
| Page load <5s | Performance | `performance.spec.js` | — | **Covered** |
| 100 jobs render <10s | Performance | `performance.spec.js` | — | **Covered** |
| Tab switch <2s | Performance | `performance.spec.js` | — | **Covered** |
| No horizontal scroll at 375px | Responsive | `responsive-mobile.spec.js` | — | **Covered** |
| Touch targets ≥24px | Responsive | `responsive-mobile.spec.js` | — | **Covered** |
| Keyboard navigation (Tab/Enter/Escape) | Accessibility | `accessibility.spec.js` | — | **Covered** |
| Heading hierarchy | Accessibility | `accessibility.spec.js` | — | **Covered** |
| Buttons have accessible names | Accessibility | `accessibility.spec.js` | — | **Covered** |
| CJK/RTL/emoji text renders | Localisation | `localisation-edge.spec.js` | — | **Covered** |
| Multi-currency display (£/€/$) | Localisation | `localisation-edge.spec.js` | — | **Covered** |
| Offline resilience | PWA | `pwa-install.spec.js`, `applications-negative-advanced.spec.js` | — | **Covered** |
| Service worker registration | PWA | `pwa-install.spec.js` | — | **Covered** |
| GDPR export (Art. 15/20) | Compliance | `gdpr-account.spec.js`, `data-export-gdpr.spec.js` | — | **Covered** |
| GDPR erasure (Art. 17) | Compliance | `gdpr-account.spec.js`, `data-export-gdpr.spec.js` | — | **Covered** |
| XSS prevention | Security | `edge-cases-advanced.spec.js`, backend: `test_registration.py` | TC-EDGE-05, CA-40 | **Covered** |
| SQL injection prevention | Security | `edge-cases-advanced.spec.js`, backend: `test_registration.py` | TC-EDGE-05, CA-41 | **Covered** |

---

## Journey → Spec Mapping

### Journey 1.1 — Login & Authentication (27 scenarios)

| Scenario | Spec File | Test Case |
|---|---|---|
| 1.1.1 Valid credentials → dashboard | `smoke.spec.js`, `login-registration-full.spec.js` TC-LR-01/04 | Happy-path |
| 1.1.2 Invalid credentials → error | `auth-negative.spec.js`, TC-LR-05/08 | Negative |
| 1.1.3 Session timeout → clean logout | `auth-negative.spec.js`, TC-LR-32 | Edge |
| 1.1.4 User switching → no data leak | `cross-user-isolation.spec.js` | Regression (LESSON-029) |
| 1.1.5 Try Demo → no fetch-storm | `try-demo-fetch-storm.spec.js` | Regression (LESSON-028) |
| 1.1.6 Try Demo → all nav visible | `demo-mode-navigation.spec.js` | Regression |
| 1.1.7 OAuth redirect | `oauth-and-folder.spec.js`, `oauth-callback.spec.js` | Happy-path |
| 1.1.8–27 | `login-registration-full.spec.js` TC-LR-02–40 | Full coverage |

### Journey 1.2 — CV Upload (7 scenarios)

| Scenario | Spec File | Test Case |
|---|---|---|
| 1.2.1 Upload PDF/DOCX | `cv-upload.spec.js`, `cv-analysis-full.spec.js` TC-CV-01/02 | Happy-path |
| 1.2.2 Boundary: 9.9/10/10.1 MB | `cv-upload-boundaries.spec.js`, TC-CV-05 | Boundary |
| 1.2.3 Unsupported file type | `cv-upload-boundaries.spec.js`, TC-CV-03 | Negative |
| 1.2.4–7 | `cv-upload-boundaries.spec.js`, `cv-analysis-full.spec.js` | Edge |

### Journey 1.3 — AI Analysis (6 scenarios)

| Scenario | Spec File |
|---|---|
| 1.3.1 CV processed, results displayed | `cv-upload.spec.js`, `cv-analysis-full.spec.js` |
| 1.3.2 Analysis timeouts handled | `cv-analysis-ai.spec.js` |
| 1.3.3 Provider error → fallback | `cv-analysis-ai.spec.js`, `ai-failover.spec.js` |
| 1.3.4 User switching → analysis hidden | `cross-user-isolation.spec.js` |
| 1.3.5 Multi-provider comparison | `cv-analysis-ai.spec.js` |
| 1.3.6 Results survive refresh | `cv-analysis-ai.spec.js` |

### Journey 1.4 — CV Editing (5 scenarios)

| Scenario | Spec File |
|---|---|
| 1.4.1 Edit + save | `document-editing.spec.js` |
| 1.4.2 AI suggestions | `cv-editing-export.spec.js` |
| 1.4.3 Undo/redo | `cv-editing-export.spec.js` |
| 1.4.4 XSS / long text | `cv-editing-export.spec.js` |
| 1.4.5 Concurrent edits | `cv-editing-export.spec.js` |

### Journey 1.5 — CV Export (6 scenarios)

| Scenario | Spec File |
|---|---|
| 1.5.1 Export to PDF | `cv-export-download.spec.js` |
| 1.5.2 Export to DOCX | `cv-export-download.spec.js` |
| 1.5.3 Export with missing fields | `cv-editing-export.spec.js` |
| 1.5.4 Export on mobile (375px) | `essential-controls.spec.js` |
| 1.5.5 Close + Download always visible | `essential-controls.spec.js` |
| 1.5.6 Export retains formatting | `cv-editing-export.spec.js` |

### Journey 1.6 — Job Tracker (8 scenarios)

| Scenario | Spec File |
|---|---|
| 1.6.1 Add a job | `job-search-flow.spec.js`, `add-job-link.spec.js` |
| 1.6.2 Edit a job | `job-tracker-crud.spec.js` |
| 1.6.3 Delete a job | `job-tracker-crud.spec.js` |
| 1.6.4 Status updates persist | `job-tracker-crud.spec.js` |
| 1.6.5 Sorting and filtering | `job-tracker-crud.spec.js` |
| 1.6.6 Persistence across sessions | `job-tracker-crud.spec.js` |
| 1.6.7 Delete non-existent → 404 | `job-tracker-crud.spec.js` |
| 1.6.8 User switching → own jobs | `cross-user-isolation.spec.js` |

### Journey 1.7 — Job Search (4 scenarios)

| Scenario | Spec File |
|---|---|
| 1.7.1 Search returns results | `job-search-flow.spec.js` |
| 1.7.2 SSE streaming progress | `search-streaming.spec.js` |
| 1.7.3 All boards disabled → empty | `search-boards-disabled.spec.js` |
| 1.7.4 Provider failure → others continue | `search-provider-failure.spec.js` |

### Multi-Journey Scenarios (5)

| Scenario | Spec File |
|---|---|
| S1 First-time user | `auth-register.spec.js`, `multi-journey.spec.js` |
| S2 Returning user | `returning-user-flow.spec.js` |
| S3 Mobile user | `essential-controls.spec.js`, `multi-journey.spec.js` |
| S4 Error handling | `error-handling-chain.spec.js` |
| S5 Cross-user isolation | `cross-user-isolation.spec.js` |

---

## Formal Test Plans (Page-Level)

| Page | Plan File | Backend Tests | E2E Tests | Total |
|---|---|---|---|---|
| Job Search | `test_search_formal_plan.py` + `search-formal-plan.spec.js` | 22 | 15 | 37 |
| Applications | `test_applications_formal_plan.py` + `applications-formal-plan.spec.js` | 28 | 23 | 51 |
| Registration | `test_registration_formal_plan.py` + `registration-formal-plan.spec.js` | 16 | 3 | 19 |
| Settings | `settings-formal-plan.spec.js` | — | 16 | 16 |
| Companies | `companies-formal-plan.spec.js` | — | 14 | 14 |
| Documents | `documents-formal-plan.spec.js` | — | 13 | 13 |

---

## Cross-Cutting Test Suites

| Category | Spec File | Tests |
|---|---|---|
| Accessibility | `accessibility.spec.js` | 7 |
| Responsive/Mobile | `responsive-mobile.spec.js` | 17 |
| Performance | `performance.spec.js` | 5 |
| AI Failover | `ai-failover.spec.js` | 5 |
| Cross-Browser | `cross-browser-edge.spec.js` | 6 |
| Localisation | `localisation-edge.spec.js` | 6 |
| PWA | `pwa-install.spec.js` | 10 |
| GDPR/Export | `data-export-gdpr.spec.js` + `gdpr-account.spec.js` | 11 |
| Edge Cases | `edge-cases-advanced.spec.js` + `applications-negative-advanced.spec.js` | 13 |
| Real-Backend | `e2e/real-backend/*.spec.js` | 22 |

---

## Coverage Summary

| Metric | Value |
|---|---|
| BRD requirements mapped | 40+ (all Must Have + Should Have, incl. Pro tier surface) |
| FSD features mapped | 50+ |
| Test Plan journeys | 7 + 5 multi-journey = 12 |
| E2E scenarios | 48/48 = **100%** |
| E2E spec files | 144 |
| Backend test files | 302 |
| Frontend test files | 171 |
| Total automated tests | **8,700+** |
| Formal test plans (page-level) | 6 pages covered |
| Cross-cutting suites | 10 categories |
| Critical regression guards | 13+ tests (deploy gate, `@critical-regression`) |
| Lessons with prevention | 55 (all applied; 46 auto-correction rules in CLAUDE.md) |

> Counts auto-sourced from `python scripts/generate_stats.py` (`frontend/src/data/stats.json`). Last refreshed at v0.6.1.

**Full traceability maintained from BRD → FSD → Test Plan → E2E → Test Cases.**
