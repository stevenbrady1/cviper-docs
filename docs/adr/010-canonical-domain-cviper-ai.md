# ADR 010 — Canonical domain: cviper.ai (cviper.uk 301s to it)

**Status**: accepted
**Date**: 2026-07-27
**Source**: SEO audit 2026-07-27 (seo-quick-win-issues.md), Issues 1–4

## Context

`cviper.uk` and `cviper.ai` both returned HTTP 200 with identical content and no
redirect in either direction. `og:url`, `sitemap.xml`, and `robots.txt` already
pointed at cviper.ai (post-rebrand 2026-05, see `azure/container-apps.bicep`
header comment), but the served HTML carried no `<link rel="canonical">` and
nothing forced one domain to win. Consequences: duplicate-content risk, split
backlink equity, and Google free to pick either domain as canonical.

Every future SEO/content investment compounds on exactly one domain, so the
choice had to be made explicitly rather than left to crawler heuristics.

## Decision

**cviper.ai is the canonical domain.** Chosen over cviper.uk for global
headroom and AI association; it also matches the infra intent already recorded
in the Bicep (custom-domain comment, Plausible `data-domain`, `og:url`,
sitemap/robots) — the rebrand had de facto happened, this ADR makes it binding.

Concretely:

- All absolute self-referencing URLs in served output (`index.html` meta/OG,
  `sitemap.xml`, `robots.txt`) use `https://cviper.ai` only.
- `<link rel="canonical" href="https://cviper.ai/" />` ships in the served HTML.
  Future marketing routes must emit per-page canonicals, not the root.
- nginx (`frontend/nginx.azure.conf.template`) 301s `cviper.uk`,
  `www.cviper.uk`, and `www.cviper.ai` to `https://cviper.ai$request_uri`,
  preserving path and query. The apex `cviper.ai` block remains the default
  server so Container Apps health probes (internal Host) are unaffected.
- CI guard `scripts/check_canonical_domain.py` runs against the built
  `frontend/dist/` and fails on any `https://cviper.*` URL whose host is not
  exactly `cviper.ai` (forbid-list shape, LESSON-033; scans rendered output,
  LESSON-034).

## Consequences

- **Search Console**: verify both domains; submit the sitemap under cviper.ai;
  use Change of Address for cviper.uk if it has indexed pages. (Manual, owner
  action — cannot be done from the repo.)
- **cviper.uk stays registered and bound** in Container Apps (cert + CORS
  entries retained) so the 301 can be served; do not drop the binding.
- **Brand surfaces referencing cviper.uk** (marketing copy, personal
  CLAUDE.md "CViper (cviper.uk)", app-store listings, email footers) should be
  updated opportunistically — the 301 keeps old links working meanwhile.
- iOS/Android bundle IDs are already `ai.cviper.app` — no mobile impact.
- Prod rollout of the nginx redirect goes through the normal gated deploy;
  acceptance is checked on production (`curl -I https://cviper.uk/x?q=1` →
  `301` + `Location: https://cviper.ai/x?q=1`), not staging only.

## Alternatives considered

- **cviper.uk canonical**: UK trust signal matching the London-finance
  positioning; rejected — contradicts the 2026-05 rebrand already embedded in
  infra/meta, and narrows global headroom.
- **Keep both live, canonical tag only**: leaves 200-duplicates and split link
  equity; canonical tags are a hint, not a directive. Rejected.
