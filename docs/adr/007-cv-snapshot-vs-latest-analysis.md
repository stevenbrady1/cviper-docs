# ADR-007: cv_snapshot vs latest CV analysis — single source of truth on the Job Search page

**Status**: accepted (implemented 2026-05-15)
**Date**: 2026-05-15
**Decision makers**: Project owner
**Related**: [CV-317](https://github.com/stevenbrady1/CViper/issues/547), LESSON-089, ADR-001 (dual-DB) for migration approach
**Implementation**: Slices 1-7 shipped 2026-05-15 (see "Implementation status" below)

## Context

`SearchProfile.cv_snapshot` is a JSON column on the [`SearchProfile` model](../../backend/database.py#L553) that holds a frozen copy of skills, suggested titles, summary, and experience years taken at the time the profile was saved. It exists to support three distinct callers:

1. **Backend search** ([`backend/routes/search.py:288-289`](../../backend/routes/search.py#L288-L289)) — when the user kicks off a search with `profile_id`, the backend uses `cv_snapshot` AS the profile being matched against jobs. Without it, search-with-saved-profile would have nothing to score with.
2. **Backend profile comparison** ([`backend/routes/search.py:691-694`](../../backend/routes/search.py#L691-L694)) — computes shared/unique skills across multiple saved profiles by intersecting `cv_snapshot.skills`.
3. **Frontend Job Search rehydration** ([`frontend/src/App.jsx:2222-2228`](../../frontend/src/App.jsx#L2222-L2228)) — when a profile is loaded into the form, copies `cv_snapshot` into in-memory `cvAnalysis` and `suggestedTitles`, which drive the visible "CV Skills" + "Suggested from CV" chips.

CV-317 (LESSON-089, commit `bc481329`) fixed the immediate symptom: `loadSearchProfiles(autoRestore=true)` was silently rehydrating `cvAnalysis` from `cv_snapshot` for every non-sandbox user with `lastActiveProfileId` in storage, overwriting the freshly-loaded `/api/cv-analyses/latest`. Users who re-analysed their CV saw stale chips on every Job Search visit.

That fix prevents the silent overwrite at startup. It does NOT close the underlying drift class: any code path that loads a profile into the form (manual selection, deep-link, future auto-restore feature) STILL replaces the live analysis with a frozen snapshot, with no UI signal that the snapshot may be stale or how stale it is. The same drift bug is one wrong click away from coming back, and there is no guard against future regressions on the manual-load path.

This ADR decides how to close the bug class structurally.

## Decision drivers

| Driver | Weight |
|---|---|
| Prevent silent staleness — user must never see CV chips that don't reflect what they expect | High |
| Preserve "search against an older CV variant" — explicit user intent must be honoured | High |
| Preserve profile-comparison feature | High |
| Preserve backend search-with-profile feature | Critical (blocks deletion of `cv_snapshot`) |
| Minimise migration risk — `SearchProfile` rows in production cannot be lost | High |
| Minimise UI cognitive load — every search shouldn't require a CV-source decision | Medium |
| Implementation cost — bug shipped, fix shipped, this is structural hardening | Low-medium |

## Options considered

### Option A — Drop `cv_snapshot` from the model entirely; rehydrate from `cv_analysis_id`

Profile stores `cv_analysis_id` (already exists, nullable) as the only CV reference. Backend resolves it to live analysis at search/load time. Snapshot column dropped via migration.

- **Pros**: single source of truth by construction; drift impossible
- **Cons (blocking)**: breaks backend search when the source CV analysis was deleted (orphan profile becomes unusable); breaks profile comparison if any compared profile's analysis is gone; eliminates "freeze a profile against a specific CV variant" use case which several users may rely on; non-trivial migration (existing profiles without `cv_analysis_id` need a synthesised one or graceful failure)
- **Verdict**: rejected — `cv_snapshot` is load-bearing for backend search, this option breaks more than it fixes

### Option B — Add `cv_snapshot_taken_at` column + drift-aware UI rehydration ⭐ recommended

Schema change: `SearchProfile.cv_snapshot_taken_at` (nullable timestamp, backfill from `created_at`). On `loadProfileIntoForm`, frontend compares against `cvAnalysisLatest.saved_at`. If the snapshot is older, the form still loads the snapshot (preserving the user's explicit intent) but shows a banner: *"This profile's CV is from {date}. Update to your current CV?"* with a one-click action that calls `PATCH /api/search-profiles/{id}` to re-snapshot from the latest analysis.

- **Pros**: drift becomes user-visible instead of silent; explicit user intent ("freeze against old CV") preserved by default; one-click escape hatch when staleness is unintended; backend stays unchanged; profile comparison stays unchanged; comparison endpoint can optionally surface the timestamp too ("Profile X's CV is 60 days old")
- **Cons**: schema migration (one nullable column, low risk); needs new component (DriftBanner); needs new endpoint or extends existing PATCH; ~3 days of work
- **Verdict**: best balance of correctness, preservation of features, and UX clarity

### Option C — Per-search "use saved profile's CV vs my current CV" toggle

UI surfaces an explicit choice on every search: search against the profile's saved CV, or use latest CV with profile's other filters (location, salary, etc.).

- **Pros**: maximally explicit; both modes first-class
- **Cons**: cognitive load on every search; the typical user just wants their saved-search filters back, not a CV-source meta-decision; doubles the test matrix for the search backend
- **Verdict**: rejected — solves the drift problem by making the user solve it on every search

### Option D — Status quo + clearer profile-load labelling

Keep schema and behaviour as-is post-CV-317. Add a banner on Job Search whenever a profile has been loaded: *"Showing skills from saved profile 'Senior BA' (saved 2026-03-02). Re-analyse CV to update."* No drift comparison, no automatic action — just label the source.

- **Pros**: zero schema change; cheapest possible (~half a day); CV-317 already prevents auto-overwrite
- **Cons**: relies on user reading the banner and acting; doesn't tell the user *whether* the snapshot is actually stale (they have to know their last CV-analysis date); doesn't prevent future-similar-bug because the latent overwrite path on manual load stays
- **Verdict**: useful as a complement to Option B, insufficient on its own

## Decision

**Adopt Option B (recommended).** Add `cv_snapshot_taken_at` to `SearchProfile`, surface a drift banner with a one-click "Update to current CV" action when the snapshot is older than `/api/cv-analyses/latest.saved_at`. Ship Option D's labelling improvements as part of the same PR — they're cheap and complementary.

Reject Option A on the grounds that `cv_snapshot` is load-bearing for backend search and profile comparison; reject Option C on grounds of UX cost.

## Implementation plan

1. **Schema** (1 migration, low-risk): add `cv_snapshot_taken_at` (nullable TIMESTAMP WITH TIME ZONE) to `SearchProfile`. Backfill from `created_at`. PostgreSQL `IF NOT EXISTS` guard per [auto-correction rule #15](../../CLAUDE.md).
2. **Backend write paths** ([`backend/routes/search.py:613-626` save](../../backend/routes/search.py#L613-L626) and the corresponding update endpoint): set `cv_snapshot_taken_at = utc_now()` whenever `cv_snapshot` is written. Add `PATCH /api/search-profiles/{id}/refresh-cv-snapshot` that re-snapshots from the user's latest analysis and updates the timestamp.
3. **Backend read path** ([`/api/search-profiles/{id}`](../../backend/routes/search.py#L649)): include `cv_snapshot_taken_at` in the response.
4. **Frontend rehydration** ([`App.jsx:2203-2255` `loadProfileIntoForm`](../../frontend/src/App.jsx#L2203-L2255)): after setting `cvAnalysis` from `cv_snapshot`, fetch `/api/cv-analyses/latest`; if `latest.saved_at > cv_snapshot_taken_at`, show a `<ProfileDriftBanner>` with the "Update to current CV" action.
5. **`<ProfileDriftBanner>` component**: copy with date, action button, dismissible. Goes between the Search Criteria card and the Match Scoring card.
6. **Tests**:
   - Unit: `ProfileDriftBanner` renders/hides correctly across the 4 cases (no profile / fresh snapshot / stale snapshot / no latest analysis).
   - Backend: `cv_snapshot_taken_at` is set on every write path; `PATCH refresh-cv-snapshot` updates both columns atomically; older profiles get the backfill correctly.
   - E2E: extend `frontend/e2e/job-search-no-profile-autoload.spec.js` with a new test that manually loads a stale-snapshot profile and asserts the banner appears + the "Update" action calls the right endpoint.
7. **Forbid-list contract** (LESSON-033 family): add `backend/tests/infrastructure/test_cv_snapshot_timestamp_invariant.py` that source-scans every backend write path setting `cv_snapshot` and fails CI if any of them omits `cv_snapshot_taken_at`. Pattern matches [`test_provider_model_source.py`](../../backend/tests/infrastructure/test_provider_model_source.py) (auto-correction rule [#46](../../CLAUDE.md)).

## Consequences

**Positive**:
- Drift becomes user-visible at the moment of profile load; no more silent staleness
- Explicit "freeze against this CV variant" use case preserved
- Backend search and profile comparison unchanged — minimal blast radius
- Forbid-list contract makes regressions structurally impossible at CI time
- Class of bug ("two backend sources hydrating the same in-memory state slot, late-running source wins") gets a documented prevention pattern reusable for similar surfaces

**Negative**:
- One schema migration (low risk; column-add only)
- New component to maintain (`ProfileDriftBanner`)
- New endpoint to test (`PATCH refresh-cv-snapshot`)
- Profile load now does an extra round-trip to compare timestamps (mitigatable by including `latest_cv_analysis_saved_at` in the `/api/auth/me` or the existing profile GET response)

**Costs**: ~3 days of implementation + tests, half a day of review.

**Migration risks**: backfilling `cv_snapshot_taken_at = created_at` for existing rows is approximate (the snapshot may have been refreshed via update without us tracking it). Acceptable because the worst case is "show drift banner when snapshot is actually current" — false positive that the user can dismiss, not a data corruption risk.

**Open question**: should the backend block searches against snapshots older than N months, or just warn? Defer to a separate decision once usage data shows how stale snapshots typically get in practice. Initial implementation: warn only, never block.

**Supersedes**: nothing.

**Superseded by**: nothing (current decision).

## Implementation status (2026-05-15)

All seven slices from the implementation plan shipped:

| Slice | Artifact | Status |
|---|---|---|
| 1 | [`backend/alembic/versions/039_add_cv_snapshot_taken_at.py`](../../backend/alembic/versions/039_add_cv_snapshot_taken_at.py) — column add + backfill from `created_at`. Idempotent (PG `IF NOT EXISTS`, SQLite introspection). | ✅ |
| 1 | [`backend/database.py:545-573`](../../backend/database.py#L545-L573) — `SearchProfile.cv_snapshot_taken_at` column. | ✅ |
| 2 | [`backend/repositories.py:save_profile`](../../backend/repositories.py) and `update_profile` — auto-stamp `cv_snapshot_taken_at` when `cv_snapshot` is written, no-op when not. Caller can override via explicit kwarg (used by migrations). | ✅ |
| 2 | [`backend/sandbox.py:_seed_search_profile`](../../backend/sandbox.py) — sets `cv_snapshot_taken_at` alongside snapshot. | ✅ |
| 2 | [`backend/routes/search.py`](../../backend/routes/search.py) — new `PATCH /api/search-profiles/{id}/refresh-cv-snapshot` endpoint. Atomic: re-snapshots from latest analysis, updates timestamp via repo's auto-stamp, returns refreshed profile. | ✅ |
| 3 | `repo.get_profile` and `repo.get_all_profiles` include `cv_snapshot_taken_at` in response. | ✅ |
| 4 | [`frontend/src/components/ProfileDriftBanner.jsx`](../../frontend/src/components/ProfileDriftBanner.jsx) — banner component with `isProfileSnapshotStale` helper. Defensive against missing/unparseable timestamps (no false positives). | ✅ |
| 5 | [`frontend/src/App.jsx:loadProfileIntoForm`](../../frontend/src/App.jsx) — fetches `/api/cv-analyses/latest` after snapshot load, computes drift, sets `profileDriftInfo`. New `refreshProfileSnapshot(profileId)` handler calls PATCH endpoint and re-loads. | ✅ |
| 5 | [`frontend/src/hooks/useCvAnalysisState.js`](../../frontend/src/hooks/useCvAnalysisState.js) — added `profileDriftInfo` and `refreshingProfileSnapshot` state slots. | ✅ |
| 5 | [`frontend/src/tabs/SearchTab.jsx`](../../frontend/src/tabs/SearchTab.jsx) — renders `<ProfileDriftBanner>` between PageHeader and SearchForm. | ✅ |
| 6 | Backend tests: [`backend/tests/search/test_cv_snapshot_drift.py`](../../backend/tests/search/test_cv_snapshot_drift.py) — 12 tests covering repo invariant + serializer + PATCH endpoint happy/edge cases. All pass. | ✅ |
| 6 | Frontend unit tests: [`frontend/src/components/ProfileDriftBanner.test.jsx`](../../frontend/src/components/ProfileDriftBanner.test.jsx) — 14 tests covering visibility, defensive rendering, refresh/dismiss callbacks, disabled-during-refresh contract. All pass. | ✅ |
| 6 | E2E: [`frontend/e2e/job-search-no-profile-autoload.spec.js`](../../frontend/e2e/job-search-no-profile-autoload.spec.js) — extended from 3 to 5 tests adding the drift banner + refresh + dismiss scenarios. All pass. | ✅ |
| 7 | Forbid-list contract: [`backend/tests/infrastructure/test_cv_snapshot_timestamp_invariant.py`](../../backend/tests/infrastructure/test_cv_snapshot_timestamp_invariant.py) — AST-based scanner of `repositories.py` and `sandbox.py`. Catches direct kwarg writes, setattr-with-literal, attribute writes, AND the dynamic-dispatch pattern (string-literal + setattr in same function). 8 tests including 4 adversarial RED→GREEN proofs. | ✅ |

**Open follow-ups** (deferred — not blockers):
- Profile-comparison endpoint could optionally surface snapshot age per profile in its response so the UI can show "Profile X's CV is N days older than Profile Y's". Low value until users actually use compare frequently.
- A future preference toggle "remember last loaded profile" can be wired to `userStorage.lastActiveProfileId` (which is still being written) without a migration. Deferred until requested.
