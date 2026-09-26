# ADR 013 — What serves cviper.ai during the mothball

**Status**: accepted
**Date**: 2026-09-26
**Source**: cycle-plan M2 (`ClaudeReports/summaries/2026-09-26-summary-cycle-plan.md`), verified against the live `stevenbrady1/cviper-landing` repository (HEAD a7acef9, 2026-09-25)
**Related**: [ADR 011](011-two-product-architecture.md) (two products, one brand), [ADR 012](012-cviper-light-licence-and-distribution.md) (Light distribution), `docs/MOTHBALL-2026-09.md` (CV-1400)

## Context

The mothball (CV-1400, 2026-09-10) deleted every Azure resource, including the
Container Apps that served this repository's frontend and backend at
cviper.ai. Feature work on this repository's `frontend/` continued after that
date — the Light download panel (CV-1389, #1584) and the `/import` account
gateway (CV-1395, #1582) both merged post-mothball — which raised the
question this ADR answers: **what does a visitor to cviper.ai actually get,
and where does landing work belong?**

## Findings (verified 2026-09-26)

1. **cviper.ai is served by the `stevenbrady1/cviper-landing` GitHub Pages
   repository** — its `CNAME` file names `cviper.ai`, and its README states
   its purpose: "the static holding page served at https://cviper.ai while
   the CViper hosted application is paused." No build step, no external
   requests, no JavaScript dependencies for core content.
2. The live site consists of: the home page (whose download buttons link to
   `github.com/stevenbrady1/cviper-light/releases/latest`), `/privacy`
   (a transcription of the policy Light *generates* from its outbound-host
   registry, with a drift checker at `tools/check_privacy_drift.py` and a
   fresh 0.2.0 regeneration merged 2026-09-25), and two redirects:
   `/light → /` and `/light/privacy → /privacy` (the URLs the app and its
   store listings publish).
3. **Nothing in this repository's `frontend/` is deployed anywhere.** The
   post-mothball merges CV-1389 and CV-1395 are *reinstate-ready* work: the
   live site contains zero references to `/import`, so the feared "live page
   promising account creation against a deleted backend" does **not** exist.
   The dormant code is correct-by-construction — it simply is not served.

## Decision

- **Changes intended to be LIVE on cviper.ai go to `cviper-landing`**, not to
  this repository's `frontend/`. That includes copy, download links, and the
  privacy transcription refresh (whose trigger is any change to Light's
  `apps/light/src/lib/outbound-hosts.ts` / `dataLocations.ts`).
- **Changes to this repository's `frontend/` are for the reinstated product
  only** and must not be described as shipped in user-facing terms until the
  reinstate happens. PR/issue text for such work should say "reinstate-ready".
- The visual safety net planned as cycle item M4 targets **`cviper-landing`'s
  pages** (the surface with visitors), not this repository's frontend.

## Consequences

- The M2 risk flagged in the 2026-09-26 cycle plan ("`/import` may be a live
  trust bug") is resolved as a non-issue; the plan's risk register is updated
  in place.
- The privacy page's only freshness link between the two repositories is a
  human habit plus `check_privacy_drift.py` inside `cviper-landing`; any
  change to Light's outbound-host registry must be followed by a regeneration
  there (already exercised once, for 0.2.0).
- Housekeeping noted, not fixed here: two ADRs currently share the number 012
  (`012-cviper-light-licence-and-distribution.md` and
  `012-hosted-trial-credits.md`); the next free number after this file is 014,
  and one of the two 012s should be renumbered when next touched.
