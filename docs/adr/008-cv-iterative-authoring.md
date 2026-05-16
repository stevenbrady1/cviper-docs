# ADR-008: Iterative CV/cover letter authoring via a persistent `cv_drafts` version DAG

**Status**: accepted (Phase 1 in progress)
**Date**: 2026-05-16
**Decision makers**: Project owner
**Related**: [CV-318](https://github.com/stevenbrady1/CViper/issues/549) (foundation), CV-319 → CV-326 (rollout cards), ADR-001 (dual-DB), ADR-007 (cv_snapshot SoT)
**Implementation**: Phase 1 = CV-318 → 326. Phase 2/3 reserved (see "Forward-looking columns" below).

## Context

CViper today generates a tailored CV + cover letter per job through four backend handlers:

- [`/api/apply-single`](../../backend/routes/jobs.py) — first generation for a saved job
- [`/api/update-cv`](../../backend/routes/documents.py) — regenerate against the same job (overwrites)
- [`/api/alternative`](../../backend/routes/documents.py) — produce an alternative variant
- [`/api/multi-generate`](../../backend/routes/documents.py) — produce one variant per configured provider

Each handler writes its output to the user's CV folder as filesystem files: `CV.docx`, `CV_v2.txt`, `CV_pre_update_<timestamp>.txt`, `CV_<provider>.txt`. The DocumentCentre tab presents a "variant selector" that scans the folder and picks the latest by mtime.

**Problems this creates:**

1. **Loss of authoring history** — `update-cv` overwrites the previous variant. The `_pre_update_*.txt` rescue files are a workaround, not a model.
2. **No version DAG** — you can't ask "which draft did the cover letter on disk belong to?" or "which CV did I generate first for this job?" because there's no relationship between variants.
3. **No cross-tab persistence** — variants live in the user's local CV folder; cloud-mode users have no folder, so the variant selector is empty on the web. The filesystem-only world is incompatible with the deployed product.
4. **No iteration affordance** — users can't say "regenerate this CV but keep the cover letter" or "diff version 2 against version 1" because the data model can't represent the relationship.
5. **Fabrication risk grows with every iteration** — each regenerate is its own AI call, with no guarantee that successive drafts stay anchored to the base CV. The `detect_fabrication()` helper exists but only runs on the *first* generation.

The product's value proposition is iterative authoring; the data model treats CVs as single-shot disposable artefacts. The gap is structural.

## Decision drivers

| Driver | Weight |
|---|---|
| Make iteration possible without changing what works today | Critical |
| Preserve every existing endpoint's response shape (no client breakage) | Critical |
| Keep fabrication risk bounded as draft count grows | High |
| Cloud-mode parity — drafts must live in Postgres, not the filesystem | High |
| Support both "siblings" (alternative variants) and "children" (refinements) | High |
| Migrate the four legacy handlers without a freeze | Medium |
| Don't ship dead schema — every column must be used in Phase 1 OR justified for Phase 2/3 | Medium |

## Decision

Introduce a `cv_drafts` table that models drafts as a **persistent version DAG**, and adapter-write the four legacy generation endpoints into it. The legacy handlers keep their existing response shapes; the new draft REST endpoints (CV-319) provide the iteration affordances.

### Data model — one draft = one row

Each row in `cv_drafts` is a single AI generation of a CV (with an optional cover letter), attached to a job and (transitively) to a user. The DAG is encoded by two nullable foreign keys:

- `parent_draft_id` (FK → `cv_drafts.id`, `SET NULL`): the previous version this draft refines. Null for the first draft on a job. Non-null forms the **child** edge.
- `base_cv_id` (FK → `cv_analyses.id`, `SET NULL`): the user's source CV the draft was tailored from. Anchors fabrication checks.

A draft is **promoted** when `is_current=True`. At most one draft per job can be current at a time; `promote_cv_draft` flips the bit atomically.

```
job
 ├── draft v1 (parent=NULL, base=cv-analysis-X)         ← initial /apply-single
 ├── draft v2 (parent=v1)                                ← /update-cv on top of v1
 ├── draft v3 (parent=v1, ai_provider=anthropic)         ← /alternative (sibling)
 └── draft v4 (parent=v1, ai_provider=openai)            ← /multi-generate (sibling)
                       ↑ is_current=True
```

A unique constraint on `(job_id, version_number)` prevents duplicate versions per job; an index on `(job_id, is_current)` makes the "what's current?" query trivially fast.

### Adapter-write, not rewrite

The four legacy generation endpoints keep their behaviour and response shapes. Inside each handler we **add** a non-blocking write to `cv_drafts` wrapped in `try/except`:

```python
try:
    draft_id = repo.save_cv_draft(session, {...}, user_id=user_id)
    repo.promote_cv_draft(session, draft_id, user_id=user_id)
except Exception as exc:
    log.warning("draft_dual_write_failed", extra={"event_type": "draft_dual_write_failed", "job_id": job_id, "err": str(exc)})
```

If the draft write fails for any reason — DB outage, schema mismatch, encryption-key rotation glitch — the user request completes normally with the filesystem artefacts. The failure surfaces as `event_type=draft_dual_write_failed` log entries, which become observability data for rollout.

This converts a P1 outage risk (broken /apply-single because of a new code path) into a P3 observability event. The cost is that drafts can briefly be missing from the DAG until the underlying issue is fixed; in exchange, we get a non-breaking migration path for production traffic.

### Fabrication check on the new path is **mandatory**

`detect_fabrication()` is mandatory on every `save_cv_draft` call from the new REST endpoints (CV-319). Result is persisted to the `fabrication_check` JSON column for later inspection.

For the legacy endpoints' adapter-writes, fabrication is gated by the `CV_FABRICATION_STRICT` env var:

- **`false`** (rollout default): runs the check, persists the result, but does **not** block the response. We observe real-world fabrication rates without risking generation failures.
- **`true`** (post-rollout, after we see how often the check trips): high-risk drafts (`fabrication_risk` ≥ 0.5) get an error response instead of being persisted.

The kill-switch shape means we can flip it after observing N hours/days of real fabrication-check telemetry, with confidence that the strict mode won't quietly break a high % of generations.

### Encryption — Fernet at the app layer

`cv_text` and `cover_letter_text` are encrypted via `encrypt_text()` / `decrypt_text()` from `backend/encryption.py`. Decryption happens in `_cv_draft_to_dict` and `get_cv_draft` only — the ORM column holds ciphertext.

This mirrors `CvAnalysis.cv_text` (LESSON CV-228) and means a Postgres backup leak does not expose plaintext CV/cover-letter content. The encryption key lives in env vars (`CV_TEXT_ENCRYPTION_KEY`), rotated independently of database creds.

### Forward-looking columns

Five columns ship in Phase 1 but are **unused** until later phases. Including them now avoids a second migration when we get there; each is nullable so legacy callers don't need to set them.

| Column | Phase | Purpose |
|---|---|---|
| `structured_changes` (JSON) | 2 | Per-section edit ops (`{section: "experience", op: "rewrite", from: "...", to: "..."}`). Enables block-level diffs and partial re-runs. |
| `prompt_token_count`, `completion_token_count` | 1 (telemetry) | Track per-draft AI cost. Initially observed for tier-cost analysis; later drives per-user budget caps. |
| `base_cv_id` (FK to `cv_analyses`) | 1 (anchoring) | Locks each draft to a known base CV — required for `detect_fabrication()` to have something to compare against. |
| `superseded_at` (DateTime) | 2 | Lifecycle marker for GC sweeps that clear non-current drafts older than N days. Phase 1 keeps every draft forever. |

If any of these turn out to be wrong-shape in Phase 2/3, dropping a nullable column with a migration is cheap. The reverse — discovering you need a column you don't have, mid-iteration, on production data — is expensive.

### Frontend rendering — DraftTimeline (CV-322)

The DocumentCentre's variant selector gets replaced by `<DraftTimeline>`, which renders the DAG as a vertical list of drafts grouped by parent edge. Each draft shows version number, AI provider/model, ATS score badge, change summary, and a "view diff" affordance that wires into the existing `DiffView` component (CV-324).

The legacy filesystem variant selector stays available in cloud mode for one phase as a fallback (cloud users have no folder, so it's empty anyway — the timeline becomes the only UX immediately).

## Options considered

### Option A — Adapter-write to `cv_drafts` while preserving legacy endpoints (chosen)

Adapter-write the four legacy endpoints into `cv_drafts` non-blockingly; introduce new REST endpoints for the draft DAG. Existing response shapes preserved. **Selected** — see decision drivers.

### Option B — Rewrite the four legacy endpoints to use `cv_drafts` only, no filesystem

Cleaner end-state but every cloud rollout that touches CV generation is a coordinated client + server change. One bad rollout breaks /apply-single for everyone. **Rejected** — too much risk for one phase. Phase 3 can revisit once drafts are battle-tested.

### Option C — Keep the filesystem, add a Postgres "index" table that references files

Avoids encryption-at-rest work but doesn't solve the cloud-mode problem (no filesystem in production containers), doesn't enable cross-device iteration, and re-introduces the same drift risk between disk and DB. **Rejected** — fixes nothing structural.

### Option D — Use a generic `documents` table instead of CV-specific `cv_drafts`

A `documents (id, type, body, parent_id, ...)` table where `type='cv_draft'` is one of many. Generalises but couples the schema's evolution to features we don't have (cover letter packs, portfolios, etc.). YAGNI for now. **Deferred** — the column set on `cv_drafts` is CV-specific (ATS score, fabrication check, drift-from-base). Generalising hides those behind a JSON blob and loses query power. Revisit if a second document type ships.

## Consequences

### Positive

- Every CV generation is queryable, diffable, and cross-device persistent.
- Fabrication check becomes mandatory on the new path immediately; legacy paths get telemetry without behaviour change.
- Cloud-mode users get a working version history for the first time.
- Phase 2 work (per-section edits, drift visualisations, cost dashboards) lands without another schema migration.
- The legacy filesystem flow keeps working — no big-bang cutover.

### Negative / costs

- Encryption-at-rest adds a small CPU cost per save and per read. CvAnalysis already demonstrates the pattern at scale, so the cost profile is known.
- Five forward-looking nullable columns ship unused in Phase 1. The cost is one entry in `_cv_draft_to_dict` and one row in stats.
- Two write paths (filesystem + DB) for one logical action introduce a possible drift class: filesystem says "v3 is latest", DB says "v2". Adapter-write logs `draft_dual_write_failed` whenever the DB write fails — operational signal that the two have drifted.
- `superseded_at` introduces a soft-delete pattern that needs a Phase 2 GC job; without it, draft tables grow unboundedly. Acceptable for Phase 1.

### Migration risk

- Migration 040 adds the table and unique constraint + index. Idempotent (`CREATE TABLE IF NOT EXISTS` per rule #15).
- No data migration — drafts only exist forward from Phase 1 cutover.
- Existing rows in CvAnalysis are not touched; the `base_cv_id` FK is nullable and only set on adapter-writes that have a `cv_analysis_id` in scope.

## Implementation status

- **Phase 1 / CV-318** (this ADR): schema, migration 040, repository CRUD, serialiser-completeness contract, ADR-008 ← landed in commit `84f82927` (schema + migration) + this commit (repo + ADR).
- **CV-319**: draft REST endpoints (Pydantic v2 models).
- **CV-320**: fabrication + drift check helpers on the new path.
- **CV-321**: adapter-write `/apply-single` + `/update-cv` (auto-promote new draft).
- **CV-322**: `<DraftTimeline>` component replaces DocumentCentre variant selector.
- **CV-323**: adapter-write `/alternative` + `/multi-generate` (sibling drafts under the same parent).
- **CV-324**: `<DiffView>` picker + base CV compare.
- **CV-325**: retrofit fabrication on the legacy path (`CV_FABRICATION_STRICT` env var, defaulted false).
- **CV-326**: 7-row test coverage matrix + E2E lifecycle spec.

## Related lessons

- **CV-228** (encryption pattern): `cv_text` on CvAnalysis is encrypted at rest. CvDraft inherits the same pattern.
- **CV-287** (serialiser-completeness contract, rule #56): every `_*_to_dict` helper must expose every model column or exempt it with a reason. `_cv_draft_to_dict` is registered in `HELPER_CONTRACTS`.
- **Rule #24** (user-scoped data): every `get_*` / `list_*` repo function on user-owned data MUST accept `user_id` and filter on it. `get_cv_draft`, `list_cv_drafts_for_job`, `promote_cv_draft` all do.
- **Rule #15** (idempotent DDL): migration 040 uses `CREATE TABLE IF NOT EXISTS`.
- **ADR-007** (`cv_snapshot` vs latest): the `base_cv_id` FK on CvDraft is the iterative-authoring counterpart of the `cv_analysis_id` FK on SearchProfile — both anchor user-visible derived state to a canonical CV snapshot.
