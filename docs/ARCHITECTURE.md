# CViper — Architecture

**One description covering both products.** Written 2026-08-21 (CV-1322).

Before this document existed there was no single place that described the system
as it actually is. `APPLICATION_SPEC.md` describes the hosted web app as though
it were the whole thing; CViper Light's own docs describe the desktop app and
know nothing of this repository's conventions. Neither was wrong. Together they
left a gap exactly where the interesting decisions live.

Read this first, then the diagrams, then the ADRs.

---

## 1. There are two products

| | **CViper** | **CViper Light** |
|---|---|---|
| What it is | Hosted web application | Local-first desktop application |
| Repository | `github.com/stevenbrady1/CViper` | `C:\Dev\cviper` (no remote yet) |
| Stack | FastAPI + React 18 / Vite 5 + PostgreSQL | Tauri v2 + React 19 / Vite 8 + Rust + SQLite |
| Identity | Accounts, JWT, refresh cookies | None. Nothing to log into |
| Data | PostgreSQL on Azure, encrypted fields | One SQLite file on the user's disk |
| AI keys | Admin-managed and per-user, server-side | The user's own, in the OS credential store |
| Deployment | Azure Container Apps, manual deploy gate | Signed desktop installers (unsigned today) |
| Telemetry | Opt-in analytics | None, and a test enforces it |

They are **not** a front end and a back end. They are not a fork. They are two
products that share a brand and a body of hard-won domain logic.

**Diagram:** `two-product-context.svg`

### Why two

CViper is for someone who wants their job hunt on any device, with the AI
already paid for. CViper Light is for someone who will not put their CV on
somebody else's server. Those are different people, and one product cannot be
both without lying to one of them.

The decision, its costs, and the rules that follow are recorded in
[ADR-011](adr/011-two-product-architecture.md).

---

## 2. CViper — the hosted product

**Diagrams:** `system-architecture.svg`, `container-dev.svg`, `auth-rbac-flow.svg`,
`data-model.svg`, `ai-routing.svg`, `cicd-pipeline.svg`

### Request path

```
Browser
  → Cloudflare (DNS, TLS, WAF, CDN)
    → Azure Container Apps
        cviper-frontend   nginx + React SPA
        cviper-backend    Uvicorn + FastAPI
          → PostgreSQL (Azure Flexible Server, VNet private endpoint)
          → AI providers (admin keys ∪ the user's own personal keys)
```

Live counts are in [`STATS.md`](STATS.md) and are generated, never hand-written.

### Backend layering — and an honest note about it

The declared layers are `routes/` → `services/` → `repositories/`, with `ai/` as
a parallel domain package.

**The names do not currently match reality.** Measured 2026-08-20:

| Layer | Files | Lines |
|---|---|---|
| `routes/` | 34 | 21,174 |
| `ai/` | 38 | 17,729 |
| `helpers/` | 41 | **12,768** |
| `repositories/` | 8 | 4,411 |
| `services/` | 6 | **1,026** |

`helpers/` is twelve times the size of `services/` and holds real domain logic —
search, ATS scoring, geography, document generation, parse reporting. A new
developer told "business logic lives in services" will look in the wrong place
for almost all of it.

This is recorded rather than quietly tidied because fixing it is a large,
behaviour-preserving refactor that has not happened yet. See the audit report
`ClaudeReports/audits/2026-08-20-audit-full-architecture-refactor-cviper-and-light.md`,
Phase 4. Until then: **the layer names are aspirational, `helpers/` is where the
domain lives.**

### The AI gateway

Every AI call goes through `AIGateway.call()`. Nothing constructs a second
gateway. It provides, in order:

1. Per-task provider selection, which considers the user's own keys and not only
   the admin registry
2. Retry with backoff on rate limits; immediate failure on permanent quota errors
3. **One** truncation retry at double the token budget, then a typed error
4. Provider fallback down a chain
5. Response validation against a Pydantic schema — which returns the raw dict on
   mismatch rather than raising, so a schema change never blocks a user

Sanitisation happens at the schema layer (`ai/schemas.py`), so every call site
using `validate_response` gets HTML stripping and length caps for free.

---

## 3. CViper Light — the desktop product

**Diagram:** `cviper-light.svg`

### The one idea that shapes everything

**Network calls originate in Rust, never in the web page.**

The WebView renders screens and holds no key, opens no socket, and cannot reach
the network. Everything crosses a typed Tauri IPC boundary into the Rust core,
which owns the outbound requests. An API key therefore cannot enter the
renderer, even if the renderer is compromised.

This is why there is no official Tauri keyring plugin in use: Light writes ~30
lines of Rust over the `keyring` crate directly, because the specified plugin
does not exist.

### What it stores

- **SQLite**, one file on the user's disk, via `tauri-plugin-sql` (which needs
  `features = ["sqlite"]` — the plugin is inert without it)
- **OS credential store** for API keys, via the `keyring` crate. No command reads
  a key back out; a forgotten key is replaced, not recovered

### What leaves the machine

Only what the user starts: a job search, a CV analysis against a provider they
chose, an update check they press, and a single job advert they ask to fetch.
Nothing reaches a server of ours, because there is not one.

The fetch path is deliberately narrow: one page, no cookies, no key, no link
following, and private or loopback addresses refused before the request and
again on redirect.

### Workspace layout

```
apps/light             Tauri v2 desktop app
apps/cloud             Empty stub — declared in the workspace, contains nothing
packages/core-types    Shared domain types
packages/ai-providers  BYO-key and local Ollama adapters
packages/job-apis      Adzuna / Reed clients, keyless search links
packages/cv-parsing    CV ingestion and extraction
packages/keyword-scoring  The no-AI scorer
packages/ui            One file, 7 lines — a placeholder
```

Packages are **source-only and never built**: no `build` script, no `dist`. Vite
compiles them from source, `tsc --noEmit` typechecks them, Vitest runs them
directly.

`apps/cloud` and `packages/ui` are honest stubs, named as such in Light's own
`FEATURE-MATRIX.md`. They are workspace members that resolve to nothing.

---

## 4. What the two products share

Twelve modules were copied from CViper into CViper Light:

| Domain | CViper | CViper Light |
|---|---|---|
| Salary wording | `backend/salary_utils.py` | `packages/ai-providers/src/salary-wording.ts` |
| Dedupe fingerprint | `backend/helpers/dedupe_fingerprint.py` | `packages/job-apis/src/fingerprint.ts` |
| Reed salary + contract | `backend/job_sites_api.py` | `packages/job-apis/src/reed-{salary,contract}.ts` |
| Keyword scorer | `backend/ai/keywords.py` | `packages/keyword-scoring/` |
| Prompt calibration | `backend/ai/prompts/constants.py` | `packages/ai-providers/src/prompt/constants.ts` |
| Extraction prompt | `backend/ai/prompts/search_helpers.py` | `.../prompt/build-extraction-prompt.ts` |
| Extraction schema | `backend/ai/schemas.py` | `packages/core-types/src/job-extraction.ts` |
| Extraction pipeline | `backend/ai/services/search_helpers.py` | `packages/ai-providers/src/extract-job.ts` |
| JSON repair + sanitise | `backend/ai/gateway.py` | `.../extract-json.ts`, `packages/cv-parsing/src/sanitize.ts` |
| Awkward-advert fixtures | two test files | `packages/ai-providers/src/test/advert-fixtures.ts` |

### Copied, not extracted — and why

A shared package would have to be published and versioned across a Python
service and a Rust/TypeScript desktop app. That is a build and release burden
out of all proportion to twelve files.

**Copies are accepted. Undetected copies are not.**

- [`port-parity-manifest.yaml`](port-parity-manifest.yaml) pins each source's
  content hash, names the downstream file, and records whether the relationship
  is `fidelity` (must behave identically) or `divergent` (deliberately differs)
- `scripts/check_port_parity.py` fails when a pin drifts, naming what depends on it
- Each ported source carries a `PORTED-TO:` marker, so a developer sees the
  warning in the file itself
- Three modules carry **measured parity tests** — expectations produced by
  running the Python, not by predicting it

The guard proves *notification*, not correctness. It cannot check that the
TypeScript is still right; it checks that somebody was told to look.

### Where they deliberately diverge

Divergence is legitimate. `salary-wording.ts` narrows patterns that are safe on
a five-word salary field but not on a whole pasted advert — unanchored,
`excellent` matches "excellent communication skills" and `\bdoe\b` matches "John
Doe" in a signature block. It also **adds pro-rata handling that CViper does not
have at all**, and returns null for day rates rather than annualising them,
because Light's schema has no `salary_period` field and a wrong number is worse
than an empty one.

The full divergence list is in the manifest, and in the table on
`two-product-context.svg`.

---

## 5. The ingestion flow — the clearest shared path

**Diagram:** `job-ingestion-flow.svg`

Both products turn a pasted advert into a tracked job through the same six
stages:

1. **Input** — paste, upload, or (Light only) fetch one page from a link
2. **Guards** — extension, magic bytes, size cap, malware scan. All of it
   **before any AI token is spent**, so an attacker cannot spend the user's
   budget. Oversize pastes are truncated at 50,000 characters, never rejected
3. **AI extraction** — the prompt rule is *null, never guess*; the gateway
   handles retry, truncation retry and fallback; the response is schema-validated
4. **Normalise** — empty string and missing collapse into one null, so the form
   renders "the advert didn't say" identically every time
5. **Human review** — every field editable, nothing saved yet. This is the point
   of the feature
6. **Save** — now treated as ordinary *user* input, because the user may have
   edited it. Salary normalised, dedupe fingerprint computed, row written

Two invariants worth stating plainly:

- **Fail-open.** Every failure mode — no AI configured, no provider, any gateway
  exception, timeout, truncation, JSON decode error — resolves to an empty
  reviewable form and an honest banner. Never a 500. A manual save still works.
- **The pasted text is never logged.** It is third-party personal data and often
  client-confidential. Length and outcome only, in every path including failures.

---

## 6. Known gaps

Stated here so nobody has to rediscover them.

| Gap | Where |
|---|---|
| `helpers/` holds the domain logic that `services/` is named for | §2 above; audit Phase 4 |
| `App.jsx` is 5,191 lines in one function with 87 hooks | audit Phase 5 |
| `repositories/__init__.py` holds 60% of its own layer | audit Phase 4 |
| Pro-rata pay is unhandled in CViper (Light has it) | manifest, `salary_utils.py` |
| Hybrid/remote wording is captured but never parsed | `job-ingestion-flow.svg` |
| `.mmd` diagram companions are unverified and some are months stale | [`DIAGRAM_SCOPE.md`](DIAGRAM_SCOPE.md) |
| Light's `apps/cloud` and `packages/ui` are empty workspace members | §3 above |
| Light's updater is wired and tested but the signing key is a placeholder | Light's `RELEASE-SIGNING.md` |
| Nine stale agent worktrees hold 85,243 files and break local source-scanning tests | audit Phase 3 hygiene |

---

## 7. Where to look next

| Question | File |
|---|---|
| Why two products? | [`adr/011-two-product-architecture.md`](adr/011-two-product-architecture.md) |
| What is shared, and is it still in sync? | [`port-parity-manifest.yaml`](port-parity-manifest.yaml) |
| What do the diagrams have to show? | [`DIAGRAM_SCOPE.md`](DIAGRAM_SCOPE.md) |
| Every other architecture decision | [`adr/README.md`](adr/README.md) |
| Live counts | [`STATS.md`](STATS.md) |
| What CViper Light does and does not do | Light's `docs/FEATURE-MATRIX.md` |
| The full structural audit | `ClaudeReports/audits/2026-08-20-audit-full-architecture-refactor-cviper-and-light.md` |
