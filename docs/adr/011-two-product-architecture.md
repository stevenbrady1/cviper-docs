# ADR 011 — Two products, one brand: CViper (cloud) and CViper Light (desktop)

**Status**: accepted
**Date**: 2026-08-20
**Source**: Architecture audit 2026-08-20 (`ClaudeReports/audits/2026-08-20-audit-full-architecture-refactor-cviper-and-light.md`), finding F1/F2
**Related**: [ADR 001](001-dual-database-strategy.md) (dual database), [ADR 002](002-ai-tier-routing.md) (AI routing)

## Context

Since August 2026 there have been **two** CViper products, in two repositories,
in two languages:

| | CViper | CViper Light |
|---|---|---|
| Repository | `github.com/stevenbrady1/CViper` | `C:\Dev\cviper` (no remote yet) |
| Stack | FastAPI + React 18/Vite + PostgreSQL | Tauri v2 + React + TypeScript + SQLite |
| Model | Hosted, accounts, server-side AI keys | Local-first, no accounts, BYO key or local Ollama |
| Size | ~146k lines Python, ~86k lines JS | ~59k lines TS + Rust |

Light is not a fork. It is a separate product that deliberately reuses logic
which was expensive to get right here — the salary wording rules, the dedupe
fingerprint, Reed's salary and contract handling, the advert-extraction prompt
and schema, the JSON repair, the prompt calibration anchors, and the keyword
scorer. Twelve CViper sources have TypeScript counterparts in Light.

Two problems followed, and neither was anybody's mistake — they are what happens
when a second product appears faster than the architecture record does:

1. **This repository had no idea Light existed.** An audit search across all 85
   tracked docs, the README, `APPLICATION_SPEC.md`, `PROJECT_CONTEXT.md`,
   `DEVELOPER_GUIDE.md` and all ten prior ADRs found exactly one mention of a
   local-first product, in a March competitor-comparison table written before
   Light was conceived. No diagram showed it. No ADR explained it.

2. **Nothing detected upstream drift.** Light's ports are deliberately
   divergent in places and each documents its own reasoning — but that reasoning
   is stated against the Python *as it was on the day of the port*. `salary_utils.py`
   changed on 2026-08-11 for New Zealand support, eight days before the module
   that depends on it was written, and New Zealand work is ongoing. Nothing on
   either side would have noticed the next change.

The audit confirmed no drift had actually occurred yet: every symbol Light cited
was still present and correct. The window was open and nothing had fallen
through it.

## Decision

**CViper and CViper Light are two products sharing a brand and a body of
domain logic, not one system with two front ends.** They stay in separate
repositories, ship separately, and are free to diverge in behaviour where the
product context differs.

Shared logic is **copied, not extracted into a common library.** A shared
package would have to be published, versioned, and consumed across a Python
service and a Rust/TypeScript desktop app — a build and release burden out of
all proportion to twelve files. Copies are accepted; **undetected** copies are
not.

Every ported source is therefore governed by three rules:

1. **`docs/port-parity-manifest.yaml` is the register.** Every ported CViper
   source appears there with its content hash, the downstream TypeScript that
   depends on it, the symbols that port relies on, and whether the relationship
   is `fidelity` (must behave identically) or `divergent` (deliberately differs,
   and the difference is the point).

2. **`scripts/check_port_parity.py` is the guard.** It fails when a pinned
   source changes, naming the downstream file and what it relies on. Its
   push-time twin is `backend/tests/infrastructure/test_port_parity_guard.py`,
   which includes a test that deliberately breaks the pin to prove the guard
   fires — this repository's most-repeated failure is a check that looks green
   because nothing ever asked it a question.

3. **Every ported source carries a `PORTED-TO:` marker.** The manifest is for
   machines; the marker is what a developer opening the file actually sees. The
   tests enforce both directions: a manifest entry without a marker, or a marker
   without a manifest entry, fails.

**Citations name symbols, never line numbers.** The first draft of the manifest
cited line numbers copied from Light's headers; adding the `PORTED-TO` markers
shifted every one of them and the manifest was stale before it was committed.
Line numbers are a citation format that decays on contact with any edit.

**Divergence is legitimate and must be recorded.** `salary-wording.ts` narrows
patterns that are safe on a five-word salary field but not on a whole pasted
advert, adds pro-rata handling CViper does not have, and returns null for day
rates rather than annualising them. That is better than the original in its
context. What the manifest protects is not sameness — it is that somebody is
told to re-check the reasoning when the ground under it moves.

## Consequences

**Good**

- Editing a ported source now produces a visible warning in the file and a red
  check in CI, naming the downstream file and what depends on it.
- The two-product shape is recorded once, in the place people look for
  architecture decisions.
- The divergence register doubles as documentation of why Light behaves
  differently — previously only discoverable by reading TypeScript comments in
  another repository.

**Costs and risks**

- **The guard proves notification, not correctness.** It cannot check that the
  TypeScript is still right — that code is in another repository, in another
  language, and Light is destined to be publicly mirrored with this
  repository's source deliberately excluded. A human still has to look.
- **Re-pinning can become a reflex.** `--repin` makes a red check green in one
  command. The manifest header, the guard's own failure output and this ADR all
  say the same thing: do not re-pin without opening the downstream file. If
  re-pinning without review becomes habit, this guard degrades into a formality
  — the exact failure recorded in LESSON-091 and auto-correction rule #71.
- **Measured parity covers the `fidelity` ports; the `divergent` ones are
  covered by their own behavioural tests, not by parity.** Three modules pin
  expectations produced by *running* the Python:
  `packages/keyword-scoring/src/parity.test.ts` (term matching),
  `packages/job-apis/src/fingerprint.test.ts` (four exact 64-bit values, all
  re-verified against today's Python on 2026-08-20), and
  `packages/ai-providers/src/prompt/constants.test.ts` (the calibration text,
  added with this ADR). The remaining ports are `divergent` by design, where an
  equality assertion would be wrong; each carries its own behavioural suite in
  Light, and this manifest is what tells somebody to revisit the divergence
  reasoning when the upstream moves.
- **Manual hash maintenance.** Every legitimate change to a ported source costs
  a re-pin commit.

## Follow-up

- Re-run the measured parity checks whenever the guard fires. They are the only
  thing that answers "is the port still *correct*", as opposed to "did the
  upstream change".
- Represent Light in the architecture diagrams: a Light architecture diagram and
  a two-product system context. (Audit Phase 2.)
- Add Light to the Documentation Registry with an owner and refresh trigger.
- Revisit this ADR if a third consumer appears. Two copies is a manifest;
  three is an argument for extracting a real shared package.
