# ADR-009: Structured CV content representation for iterative section-level editing

**Status**: proposed (draft for review)
**Date**: 2026-05-16
**Decision makers**: Project owner
**Related**: [CV-333](https://github.com/stevenbrady1/CViper/issues/565), [CV-334](https://github.com/stevenbrady1/CViper/issues/566), [CV-335](https://github.com/stevenbrady1/CViper/issues/568), [CV-336](https://github.com/stevenbrady1/CViper/issues/570), [CV-337](https://github.com/stevenbrady1/CViper/issues/572), [CV-338](https://github.com/stevenbrady1/CViper/issues/575), ADR-008 (CvDraft DAG), ADR-007 (cv_snapshot SoT)
**Implementation**: blocks CV-334, CV-335, CV-336, CV-337, CV-338. Settle this once; build for years.

## Context

ADR-008 settled the **versioning** of CVs (one row per generation, parent_draft_id DAG, encrypted text columns, fabrication checks). It deliberately did not settle the **representation** of the content within a draft, because Phase 1 only needed end-to-end text generation.

Phases 2 and 3 — the work tracked by CV-333 through CV-338 — all require **section-level operations**:

- **CV-334** (3 alternative bullet rewrites): needs to address one bullet inside one role inside `experience[]`.
- **CV-335** (section locking): needs to mark `summary` or `experience` as immutable on the next regeneration.
- **CV-336** (multi-stage wizard): builds the CV section-by-section, each section produced independently.
- **CV-337** (continuous update mode): appends an achievement to the *appropriate* role inside `experience[]`.
- **CV-338** (Quick Ask): targets edits at a named section.

All five share one prerequisite: the system must be able to read, mutate, and write **a single named section** of a CV without re-parsing the entire encrypted text blob on every operation, and without LLM-based parsing variance corrupting unrelated sections.

Today, `cv_drafts.cv_text` is a single encrypted text column. To edit one section, code must:

1. Decrypt the text.
2. LLM-parse it into structure (with non-determinism risk).
3. Mutate the named section.
4. LLM-render the structure back to text.
5. Re-encrypt and store.

This is expensive (two LLM calls per edit), non-deterministic (parsers vary), and risks silent corruption of locked sections that the user never asked to touch. The fabrication-check helper (`detect_fabrication()`) only protects against new content invented by the model — it does not protect against a parser misattributing existing content.

## Decision drivers

| Driver | Weight |
|---|---|
| Section-level edits must be deterministic — no LLM in the parse path | Critical |
| Section locking (CV-335) must be guaranteed, not best-effort | Critical |
| Existing CvDraft rows must continue to render correctly (no big-bang migration) | Critical |
| Encryption-at-rest pattern from CV-228 must extend to structured form | High |
| New writes from CV-333+ should produce structured form natively, not retro-parsed | High |
| Fabrication check must keep working — needs canonical base CV to compare against | High |
| The shape must accommodate cover letters (CvDraft already stores both) | Medium |
| Avoid two-source-of-truth drift (text vs structure disagree) | Medium |
| Migration must be idempotent and SQLite-compatible (rule #15) | Required |

## Decision (proposed)

Add a `cv_data` JSON column to `cv_drafts` as the **canonical structured form** for all new drafts created by the Phase-2 pipeline. The existing `cv_text` column becomes a **rendered cache** for legacy reads and export. Existing drafts are bootstrapped lazily: the first Phase-2 operation that touches a legacy draft parses its `cv_text` once, persists the result to `cv_data`, and from that point onward operations are JSON-typed.

### Canonical shape

```json
{
  "schema_version": 1,
  "summary": "string | null",
  "experience": [
    {
      "id": "uuid",
      "role": "string",
      "company": "string",
      "start_date": "string",
      "end_date": "string | 'present'",
      "bullets": ["string"]
    }
  ],
  "skills": ["string"],
  "projects": [
    {
      "id": "uuid",
      "name": "string",
      "description": "string",
      "bullets": ["string"]
    }
  ],
  "education": [
    {
      "id": "uuid",
      "institution": "string",
      "degree": "string",
      "year": "string"
    }
  ],
  "certifications": ["string"],
  "languages": ["string"]
}
```

Stable IDs on `experience[]` and `projects[]` items are essential — they survive renames, reorderings, and partial regeneration. The user's "lock my second role" instruction binds to an ID, not a list index.

`schema_version` is the only field that may not be omitted. Every other field is optional so partially-built CVs (CV-336 wizard mid-flow) are still valid.

### Encryption

`cv_data` is stored as a Fernet-encrypted JSON string in a `Text` column, mirroring the `cv_text` pattern from CV-228. Decryption happens in `_cv_draft_to_dict` only — the ORM column holds ciphertext. The encryption key (`CV_TEXT_ENCRYPTION_KEY`) is shared with `cv_text`; rotating the key rotates both. No per-field encryption.

### Source-of-truth direction: `cv_data` is canonical, `cv_text` is rendered

For drafts created by the Phase-2 pipeline:

- **Writes**: code mutates `cv_data` directly via typed helpers (`set_summary`, `replace_bullet`, `append_bullet_to_role(role_id)`, etc.). After every write, a deterministic renderer (`render_cv_text(cv_data) -> str`) regenerates `cv_text` and re-encrypts it.
- **Reads**: structured operations read `cv_data` directly. Legacy reads (exports, fabrication checks, the old text-based DiffView) continue to read `cv_text`.
- **Invariant**: `cv_text` is always derivable from `cv_data` via `render_cv_text`. CI guard asserts the round-trip.

The renderer is **deterministic Python**, not an LLM. Format is governed by an internal template; output is plain Markdown-flavoured text that the existing PDF/DOCX generators already consume.

### Legacy drafts: lazy bootstrap, never blocking

A draft created before Phase 2 has `cv_data = NULL`. The first Phase-2 operation on it does:

```python
if draft.cv_data is None:
    structured = parse_legacy_cv_text(decrypt(draft.cv_text))  # LLM call, ONE TIME
    draft.cv_data = encrypt(json.dumps(structured))
    session.flush()
```

The legacy parser is the only LLM call on the structured path. It runs **once per draft, ever**, and the result is persisted. If the parser fails, the operation falls back to the legacy text-based pipeline — no user-visible failure. Telemetry event `cv_legacy_bootstrap_failed` lets us measure the failure rate.

There is no batch migration of historical drafts. Drafts that are never re-edited stay text-only forever; that's fine, because nothing in Phase 2 touches them.

### Section locking lives on the draft, not on cv_data

The `locked_sections` array (CV-335) is a separate column on `cv_drafts`, not a field inside `cv_data`. Reason: the lock state is **metadata about the draft**, not part of its content. Putting it in `cv_data` would mean lock state appears in renders, exports, and diffs — none of which it should.

```sql
ALTER TABLE cv_drafts ADD COLUMN locked_sections JSON DEFAULT '[]'::json;
```

Allowed values: any subset of `['summary', 'experience', 'skills', 'projects', 'education', 'certifications', 'languages']`.

### Per-section helpers — one chokepoint per mutation type

To prevent ad-hoc mutations from drifting (rule #56 pattern), every section change goes through a typed helper in `backend/cv_data_ops.py`:

```python
def replace_summary(cv_data: dict, new_text: str) -> dict
def replace_bullet(cv_data: dict, role_id: str, bullet_idx: int, new_text: str) -> dict
def append_bullet(cv_data: dict, role_id: str, new_text: str) -> dict
def add_alternative_bullets(cv_data: dict, role_id: str, bullet_idx: int, alts: list[str]) -> dict  # CV-334
def lock_sections(draft: CvDraft, sections: list[str]) -> None  # operates on column, not cv_data
def merge_achievement(cv_data: dict, raw_text: str, kind: str) -> dict  # CV-337
```

The Quick Ask (CV-338) and chat (future) endpoints emit *intents* (`{"op": "replace_summary", "args": {...}}`); the helpers apply them. The LLM never directly writes JSON into the database — it emits ops, the server validates and applies.

This separation lets us add a "preview mutation" mode (return the result of an op without persisting) for chat UX without rewriting plumbing.

## Options considered

### Option A — `cv_data` is canonical, `cv_text` is rendered cache (chosen)

What this ADR proposes. Clean direction, deterministic rendering, structured operations, one-time legacy bootstrap. **Selected** — best fit for decision drivers.

### Option B — `cv_text` stays canonical, structure is derived on read

No schema change. Every edit re-parses text via LLM, applies the change, re-renders.

- Pro: zero migration; existing pipeline untouched.
- Con: LLM in the hot path of every edit (2 calls per Quick Ask, not 1). Non-deterministic. Section locking is enforced via post-hoc text comparison — brittle. CV-334's "3 alternative bullets" needs ID-level addressing the text doesn't provide.

**Rejected** — section locking and bullet-level addressing are not achievable with confidence.

### Option C — Hybrid: `cv_data` is a materialised cache, `cv_text` is canonical

Add `cv_data` but flag it derived. Re-parse on every legacy write to keep it fresh.

- Pro: legacy writes don't break.
- Con: cache invalidation on every legacy path; parser determinism problem returns; two writers (legacy + structured) compete to update the same cache.

**Rejected** — adds complexity without solving the determinism issue.

### Option D — Generic `documents` table with typed bodies

Replace `cv_drafts` with a `documents` table where `cv_data` is one body shape among many (cover_letter_data, portfolio_data).

- Pro: generalises for future document types.
- Con: deferred in ADR-008 (Option D there too). YAGNI; bullet-level addressing is CV-specific.

**Deferred** — revisit if a second document type ships with similar editing requirements.

### Option E — Document model in a dedicated content store (e.g., MongoDB, Postgres JSONB-only)

Move structured CV data out of Postgres entirely.

- Pro: native JSON queries.
- Con: introduces a new datastore for one feature; backup/encryption/observability all need rebuilding; conflicts with ADR-001's dual-DB strategy.

**Rejected** — operational cost dwarfs the benefit.

## Consequences

### Positive

- Section-level operations are deterministic and don't touch the LLM.
- Section locking is enforceable in code, not by post-hoc text comparison.
- Bullet alternatives (CV-334), continuous append (CV-337), and the wizard (CV-336) all read from a single canonical shape.
- The Quick Ask (CV-338) and future chat editor emit typed ops; the LLM is constrained to a known surface, reducing prompt-injection blast radius.
- Render-from-structure makes PDF/DOCX/Markdown exports trivially consistent.
- Fabrication checks gain a structural mode (compare base `cv_data` to new `cv_data`) on top of the existing text mode.

### Negative / costs

- One-time legacy parse per re-edited draft (~1 LLM call). Cost amortised over the draft's remaining edit lifetime.
- New column + new helper file + new renderer = ~400 lines of code and tests added.
- `cv_data` and `cv_text` can drift if a future bug skips the renderer step — CI guard required (round-trip test: parse → render → parse must be idempotent).
- The renderer's output format becomes a stability contract. Changes to render style affect every draft's `cv_text` on next write. Versioning the renderer (`render_v1`, `render_v2`) is the escape hatch.
- DPO surface widens: structured CV is more analysable than text. Ensure the data-export endpoint includes `cv_data` (decrypted) and the erasure flow purges both columns.

### Migration risk

- New migration `041_cv_data_column.py`: adds `cv_data Text NULL` and `locked_sections JSON DEFAULT '[]'` to `cv_drafts`. Idempotent (`ADD COLUMN IF NOT EXISTS` via dialect branch — rule #15). `batch_alter_table()` for SQLite parity.
- No data backfill. Legacy drafts stay `cv_data = NULL` until their first Phase-2 edit.
- Adapter writes (ADR-008 Option A) are unaffected — they continue to populate `cv_text` directly and skip `cv_data`.

## Decision gates before implementation

Before any code lands against this ADR, verify:

1. **Render determinism**: prototype `render_cv_text(cv_data) -> str` against 20 sample structured CVs. Confirm output is stable across runs and matches a deterministic snapshot test.
2. **Legacy parser failure rate**: estimate from a sample. If > 5% of legacy drafts cannot be parsed, the bootstrap fallback design needs revision (e.g., user-facing "we need to re-parse your CV, please confirm" UI).
3. **Cost model**: confirm the per-edit token saving (1 LLM call instead of 2) covers the one-time bootstrap LLM call within 3 edits per draft. If most drafts get fewer edits, reconsider.

These gates should be a half-day prototype in a scratch branch before opening any PR.

## Implementation status

- **Phase 1**: this ADR (proposed).
- **Decision required**: project owner approval moves status to `accepted`.
- **First implementation card**: TBD — likely a new `CV-NNN` for the column + helpers + renderer, blocking CV-334/335/336/337/338.

## Related lessons / rules

- **ADR-008**: parent ADR establishing the CvDraft DAG. This ADR extends it; does not replace it.
- **ADR-001**: dual-database strategy. `cv_data` stays in the primary Postgres / SQLite DB; no new datastore.
- **Rule #15** (idempotent DDL): migration 041 must use `ADD COLUMN IF NOT EXISTS` and `batch_alter_table()`.
- **Rule #21** (no naive datetimes): the renderer emits ISO strings for dates; no `datetime.now()` in the render path.
- **Rule #24** (user-scoped data): `cv_data` is user-owned; all repo accessors must continue to pass `user_id`.
- **Rule #56** (serialiser-completeness contract): `_cv_draft_to_dict` already registered in `HELPER_CONTRACTS` — must be updated to expose `cv_data` (decrypted) and `locked_sections`.
- **CV-228** (encryption pattern): reused unchanged for `cv_data`.
