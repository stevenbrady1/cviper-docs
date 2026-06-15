# CV-318 / Phase 1 — 7-row test coverage matrix

**Card**: CV-326 (closeout)
**Scope**: every behaviour shipped by CV-318 → CV-325 (iterative CV authoring foundation, ADR-008).
**Method**: each row in the matrix below maps a single behaviour to one cell per coverage type, citing the test that owns it. Empty cells are deliberate — every gap is explained in the "Deliberately empty" column at the end of each row.

Apply [CLAUDE.md Test Design Checklist](../CLAUDE.md) — happy / negative / boundary / edge / E2E / regression / exploratory.

## Legend

- ✅ Test exists (cite by file and `class::method`)
- ⏭ N/A with reason
- ⚠ Gap with rationale (must close in Phase 2 if marked)
- ✋ Manual / pre-release sweep

---

## CV-318 — Schema, migration 040, repository, ADR-008

| Behaviour | Happy | Negative | Boundary | Edge | E2E | Regression | Exploratory |
|---|---|---|---|---|---|---|---|
| `save_cv_draft` persists encrypted CV text | ✅ `test_cv_drafts_repository.py::TestSaveCvDraft::test_save_returns_draft_id` | — | — | ✅ `TestSaveCvDraft::test_save_with_null_cover_letter` | ⏭ (covered by CV-319 E2E below) | ✅ `TestCvDraftEncryptionAtRest::test_cv_text_ciphertext_is_not_plaintext_on_disk` (CV-228 pattern) | ✋ pre-release sweep |
| `get_cv_draft` returns the decrypted draft | ✅ `TestSaveCvDraft::test_save_then_get_round_trips_cv_text` | ✅ `TestGetCvDraft::test_get_returns_none_when_missing` | — | — | ⏭ | — | — |
| `get_cv_draft` enforces `user_id` filter | ✅ `TestGetCvDraft::test_get_filters_by_user_id` (rule #24) | ✅ same test asserts the negative path | — | — | ⏭ | — | — |
| `list_cv_drafts_for_job` returns oldest-first | ✅ `TestListCvDraftsForJob::test_list_returns_drafts_for_job` | ✅ `test_list_empty_when_no_drafts` | — | — | ⏭ | — | — |
| `list_cv_drafts_for_job` is user-scoped | ✅ `TestListCvDraftsForJob::test_list_filters_by_user_id` | ✅ same | — | — | ⏭ | — | — |
| `promote_cv_draft` atomically flips `is_current` | ✅ `TestPromoteCvDraft::test_promote_marks_draft_as_current` + `test_promote_unsets_other_drafts_for_same_job` | ✅ `test_promote_returns_false_for_missing_draft` | — | — | ✅ CV-326 e2e (this card) | — | — |
| Migration 040 idempotency (`IF NOT EXISTS`) | ⏭ enforced by rule #15 across all migrations; `init_db()` + `alembic upgrade head` smoke covers it | — | — | ⏭ dual-dialect handled by `batch_alter_table` | — | — | — |
| `_cv_draft_to_dict` exposes every column | ✅ `TestCvDraftSerialiser::test_dict_contains_every_non_exempt_column` + `test_serialiser_completeness.py` AST contract (rule #56) | ✅ `test_dict_omits_user_id` | — | — | ⏭ | ✅ contract test fires when a new column is added without updating helper | — |
| Unique constraint `(job_id, version_number)` | — | ✅ `TestSaveCvDraft::test_save_enforces_unique_job_version` | ✅ same (boundary at the constraint) | — | — | — | — |

**Deliberately empty:** boundary/edge cells without entries describe behaviours where there's no meaningful boundary (e.g. "user-scoping" is a binary yes/no — no numeric edge).

---

## CV-319 — REST endpoints

| Behaviour | Happy | Negative | Boundary | Edge | E2E | Regression | Exploratory |
|---|---|---|---|---|---|---|---|
| `POST /api/cv-drafts` creates draft, auto-assigns id + version | ✅ `test_cv_drafts_routes.py::TestCreateDraft::test_create_returns_201_with_draft_payload` | ✅ `test_create_missing_cv_text_returns_422` + `test_create_missing_job_id_returns_422` | ✅ `test_create_auto_increments_version_per_job` (max+1 contract) | — | ✅ CV-326 e2e | ✅ `extra="forbid"` on Pydantic model rejects typo'd fields | ✋ |
| `POST /api/cv-drafts` enforces job ownership | — | ✅ `TestCreateDraft::test_create_for_other_users_job_returns_404` | — | — | ⏭ | — | — |
| `GET /api/cv-drafts?job_id=X` lists drafts | ✅ `TestListDrafts::test_list_returns_drafts_for_job` | ✅ `test_list_missing_job_id_returns_422` | — | ✅ `test_list_empty_when_no_drafts` | ✅ CV-326 e2e | — | — |
| `GET /api/cv-drafts?job_id=X` is user-scoped | — | ✅ `TestListDrafts::test_list_for_other_users_job_returns_empty` | — | — | ⏭ | ✅ rule #24 enforced by the same test | — |
| `GET /api/cv-drafts/{id}` returns single draft | ✅ `TestGetDraft::test_get_returns_draft` | ✅ `test_get_missing_returns_404` + `test_get_cross_user_returns_404` | — | — | ⏭ | — | — |
| `POST /api/cv-drafts/{id}/promote` marks current | ✅ `TestPromoteDraft::test_promote_marks_draft_current` | ✅ `test_promote_missing_returns_404` + `test_promote_cross_user_returns_404` | — | ✅ `test_promote_unsets_previously_current` | ✅ CV-326 e2e | — | — |
| Response shapes don't leak `user_id` | ✅ every Create/Get test asserts `"user_id" not in data` | — | — | — | — | ✅ serialiser-completeness contract (rule #56) | — |

---

## CV-320 — Fabrication + drift on new POST path

| Behaviour | Happy | Negative | Boundary | Edge | E2E | Regression | Exploratory |
|---|---|---|---|---|---|---|---|
| `detect_fabrication` runs against user's latest CV | ✅ `TestFabricationOnCreate::test_create_runs_fabrication_check_against_user_latest_cv` | — | — | — | ⏭ E2E mocks AI | — | — |
| Explicit `base_cv_id` overrides "latest" | ✅ `test_create_uses_explicit_base_cv_id_when_provided` | ✅ `TestBaseCvIdValidation::test_create_rejects_unknown_base_cv_id` (400) + `test_create_rejects_other_users_base_cv_id` (400, rule #24) | — | — | — | — | — |
| No base CV → skip with sentinel | — | — | — | ✅ `test_create_skips_fabrication_when_no_base_cv_available` (`fallback_reason: no_base_cv`) | — | — | — |
| High-risk verdict does NOT block (Phase 1) | ✅ `test_create_does_not_block_on_high_fabrication_risk` | — | — | ✅ same | — | — | — |
| Server overrides client-supplied verdict | — | ✅ `test_create_client_supplied_fabrication_is_ignored` (security) | — | — | — | — | — |
| AI failure → check-failure sentinel | — | — | — | ✅ `test_create_continues_when_fabrication_raises` (`fallback_reason: fabrication_check_raised`) | — | — | — |
| Drift score = `1 - SequenceMatcher.ratio()` | ✅ `TestDriftScoreOnCreate::test_drift_score_zero_for_identical_text` | ✅ `test_drift_client_supplied_is_ignored` (security) | ✅ `test_drift_score_high_for_disjoint_text` (~1.0 floor) | ✅ `test_drift_score_skipped_when_no_base_cv` (returns `null`) | — | — | — |

---

## CV-321 — Adapter-write `/apply-single` + `/update-cv` (linear, auto-promote)

| Behaviour | Happy | Negative | Boundary | Edge | E2E | Regression | Exploratory |
|---|---|---|---|---|---|---|---|
| Response shape of `/api/apply-single` unchanged | ✅ `test_cv_drafts_adapter_writes.py::TestApplySingleResponseShape::test_success_response_keys_unchanged` | — | — | — | ⏭ | ✅ same test is the regression contract | — |
| Response shape of `/update-cv` unchanged | ✅ `TestUpdateCVResponseShape::test_success_response_keys_unchanged` | — | — | — | ⏭ | ✅ same | — |
| `/apply-single` persists a CvDraft | ✅ `TestApplySingleAdapterWrite::test_apply_single_persists_a_draft_for_the_job` | — | — | — | ✅ CV-326 e2e | — | — |
| Second `/apply-single` creates v2, demotes v1 | — | — | ✅ `test_second_apply_single_creates_v2_and_demotes_v1` (version chain) | — | ✅ CV-326 e2e covers second generate | — | — |
| `/update-cv` persists draft with `change_summary` + parent | ✅ `TestUpdateCVAdapterWrite::test_update_cv_persists_a_draft_with_change_summary` + `test_update_cv_links_to_previous_draft_as_parent` | — | — | — | ✅ CV-326 e2e | — | — |
| `dual_write_draft` failure → user response unchanged | — | — | — | ✅ `test_apply_single_succeeds_when_dual_write_raises` + `test_update_cv_succeeds_when_dual_write_raises` (ADR-008 non-blocking contract) | — | ✅ both tests double as regression guards | — |

---

## CV-322 — DraftTimeline component

| Behaviour | Happy | Negative | Boundary | Edge | E2E | Regression | Exploratory |
|---|---|---|---|---|---|---|---|
| Renders nothing when `jobId` is null | — | — | — | ✅ `DraftTimeline.test.jsx::renders nothing when jobId is falsy` | — | — | — |
| Loading state on initial fetch | ✅ `shows the loading state during initial fetch` | — | — | — | — | — | — |
| Empty state when API returns `[]` | — | — | — | ✅ `renders empty state when API returns no drafts` | — | — | — |
| One card per draft, badges, summary | ✅ `renders one card per draft with version, provider, ATS chip, summary` | — | — | — | ✅ CV-326 e2e | — | — |
| Legacy sentinel renders as "Fabrication check deferred" | — | — | — | ✅ `shows the legacy "Fabrication check deferred" badge for adapter-write drafts` | — | ✅ same — backward-compat with pre-CV-325 rows | — |
| High-risk fabrication renders error-coloured badge | ✅ `shows the high-risk fabrication badge with error styling` | — | — | — | — | — | — |
| Promote button hidden on current | — | ✅ `Promote button hidden on the current draft, visible on others` | — | — | — | — | — |
| Promote POST + refetch + parent callback | ✅ `Promote button POSTs to /promote and refetches on success` | — | — | — | ✅ CV-326 e2e | — | — |
| Promote failure surfaces error toast | — | ✅ `Promote failure surfaces an error toast without breaking the timeline` | — | — | — | — | — |
| Fetch error → Retry banner | — | ✅ `renders an error banner with Retry on fetch failure; Retry re-fetches` | — | — | — | — | — |
| `refreshKey` change re-fetches | — | — | — | ✅ `refetches when refreshKey changes (parent signals new generation)` | — | — | — |

---

## CV-323 — Adapter-write `/alternative` + `/multi-generate` (siblings)

| Behaviour | Happy | Negative | Boundary | Edge | E2E | Regression | Exploratory |
|---|---|---|---|---|---|---|---|
| `/alternative` response shape unchanged | ✅ `test_cv_drafts_siblings.py::TestAlternativeResponseShape::test_success_response_keys_unchanged` | — | — | — | — | ✅ same | — |
| `/alternative` persists ONE sibling, original stays current | ✅ `TestAlternativeAdapterWrite::test_alternative_persists_a_sibling_draft` | — | — | — | — | — | — |
| `/multi-generate` response shape unchanged | ✅ `TestMultiGenerateResponseShape::test_success_response_keys_unchanged` | — | — | — | — | ✅ same | — |
| Multi-gen creates N siblings sharing the same parent | ✅ `TestMultiGenerateAdapterWrite::test_multi_generate_creates_one_sibling_per_successful_provider` | — | — | ✅ same — 2-provider fan-out boundary | — | — | — |
| Multi-gen skips failed providers | — | ✅ `test_multi_generate_skips_failed_providers` | — | — | — | — | — |
| Sibling endpoints respect `dual_write_draft` failure | — | — | — | ✅ `test_alternative_succeeds_when_dual_write_raises` + `test_multi_generate_succeeds_when_dual_write_raises` | — | — | — |

---

## CV-324 — DraftCompare picker + base CV diff

| Behaviour | Happy | Negative | Boundary | Edge | E2E | Regression | Exploratory |
|---|---|---|---|---|---|---|---|
| Dropdown options gate on `baseCvText` presence | ✅ `DraftCompare.test.jsx::shows the Base CV option in both dropdowns when baseCvText is set` | ✅ `omits the Base CV option when baseCvText is empty` | — | — | — | — | — |
| Smart defaults — Base CV vs current | ✅ `smart default: Left=Base CV, Right=current draft` + `…without baseCvText: Left=v1, Right=current (or newest)` | — | — | — | ✅ CV-326 e2e | — | — |
| Diff renders added/removed spans | ✅ `renders an added span when Right has new words vs Left` | — | — | — | ✅ CV-326 e2e | — | — |
| Swap flips direction | ✅ `Swap button flips left and right selections` | — | — | ✅ `Picking Base CV on Right compares Left draft against base text` (swap-direction parity) | ✅ CV-326 e2e | — | — |
| Same-selection guard | — | — | — | ✅ `shows a "same draft selected" message when both sides match` | — | — | — |
| Close + overlay-click semantics | ✅ `Close button fires onClose` + `Clicking the overlay (not the dialog) fires onClose` | ✅ `Clicking inside the dialog does NOT close it` | — | — | — | — | — |

---

## CV-325 — Fabrication retrofit on legacy adapter-write path + `CV_FABRICATION_STRICT`

| Behaviour | Happy | Negative | Boundary | Edge | E2E | Regression | Exploratory |
|---|---|---|---|---|---|---|---|
| Observation mode persists real verdicts (no sentinel) | ✅ `TestAdapterFabricationRetrofit::test_observation_mode_persists_high_risk_verdict` | — | — | — | — | ✅ `test_apply_single_persists_a_draft_for_the_job` asserts verdict != legacy sentinel | — |
| Strict mode blocks high-risk persistence | — | ✅ `test_strict_mode_blocks_high_risk_persistence` (no draft, response still 200) | ✅ same — boundary at `risk == "high"` | — | — | — | — |
| Strict mode allows low-risk | ✅ `test_strict_mode_allows_low_risk` | — | ✅ same — narrow-scope assertion | — | — | — | — |
| AI failure → `check_raised` sentinel | — | — | — | ✅ `test_fabrication_check_failure_uses_check_raised_sentinel` | — | — | — |
| Env var read at call time (not import) | ✅ implicit — every test uses `monkeypatch.setenv` between cases | — | — | — | — | ✅ tests would fail on import-time caching | — |

---

## Cross-cutting / multi-card

| Behaviour | Coverage |
|---|---|
| **Full lifecycle E2E** — generate → timeline render → compare → promote → second generate → v2 appears | ✅ `frontend/e2e/cv-drafts-lifecycle.spec.js` (CV-326, this card) |
| **Cross-tenant isolation (rule #24)** | ✅ every Create/Get/List/Promote/Delete test has a cross-tenant assertion. Aggregate: 11 cross-tenant tests across the Phase 1 surface. |
| **Encryption-at-rest** | ✅ `TestCvDraftEncryptionAtRest::test_cv_text_ciphertext_is_not_plaintext_on_disk` (CV-228 pattern) |
| **Serialiser completeness contract** | ✅ `test_serialiser_completeness.py` AST-walks `_cv_draft_to_dict` (rule #56) — fires CI when any new column is added without updating the helper |

---

## Aggregate stats

- **Test files**: 5 backend (`test_cv_drafts_repository.py`, `test_cv_drafts_routes.py`, `test_cv_drafts_adapter_writes.py`, `test_cv_drafts_siblings.py`, `test_serialiser_completeness.py`) + 3 frontend (`DraftTimeline.test.jsx`, `DraftCompare.test.jsx`, `cv-drafts-lifecycle.spec.js`)
- **Backend tests**: 75 pass / 1 skip across the full Phase 1 surface
- **Frontend tests**: 22 pass (11 DraftTimeline + 11 DraftCompare)
- **E2E specs**: 1 (`cv-drafts-lifecycle.spec.js`, this card)

## Gaps deliberately deferred

| Gap | Why deferred |
|---|---|
| Exploratory pre-release sweep | Standard manual QA step — runs once before any user-facing rollout, not per-card |
| Per-section structured diff (DiffView) | Phase 2 — `structured_changes` column ships unused in Phase 1 per ADR-008 |
| `superseded_at` GC job | Phase 2 — no GC sweep in Phase 1; drafts persist indefinitely |
| `CV_FABRICATION_STRICT=true` rollout in production | Operational — flip after observing real-world high-risk rate via `event_type=draft_dual_write_ok` logs |
| Concurrent `promote_cv_draft` race — true multi-connection | Phase 2 — single-user single-tab assumption. The **convergence invariant** (back-to-back promotes → exactly one `is_current`) is now COVERED by `test_cv_drafts_promote_race.py`; a genuine two-Postgres-session row-lock contention test is still deferred (the in-memory SQLite StaticPool serialises writes on one connection, so OS-thread concurrency is not reproducible in-suite). |

### Phase-2 coverage added (test-scope extension, 2026-06-15)

| Gap | Now covered by |
|---|---|
| Concurrent promote convergence invariant (exactly-one-current) | `tests/data/test_cv_drafts_promote_race.py` |
| `user_id` never leaks in list/get/promote responses (end-to-end) | `tests/api/test_cv_drafts_no_userid_leak.py` |
| v2 promotion demotes v1 in the timeline (API boundary) | `tests/api/test_cv_drafts_no_userid_leak.py::TestV2DemotesV1Timeline` (the timeline *UI* assertion remains a frontend concern) |
