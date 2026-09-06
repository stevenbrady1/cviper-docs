# ADR 012 — CViper Light: free, MIT-licensed, local-only distribution

**Status**: accepted
**Date**: 2026-09-06
**Source**: Owner decisions of 2026-09-06 on `ClaudeReports/summaries/2026-09-06-summary-cviper-light-free-rollout-plan.md` §7 (itself a follow-up to the 2026-09-05 CVAurum competitive analysis)
**Related**: [ADR 011](011-two-product-architecture.md) (two products, one brand), [ADR 002](002-ai-tier-routing.md) (AI routing — the hosted product's model, which Light deliberately does not inherit)

## Context

ADR 011 established CViper Light as a separate, local-first desktop product
(Tauri v2 + React/TypeScript + SQLite) that reuses domain logic copied from
this repository under the port-parity manifest. It left three questions open:
how Light is licensed, how it is distributed, and how it avoids becoming a
cost centre or a data-protection liability for a solo owner.

The 2026-09-05 competitive analysis showed that the nearest free competitor
(CVAurum, MIT, browser-only) is credible precisely because it holds no user
data and runs no paid inference. The owner's brief for Light is the same
three-part guarantee — no GDPR exposure for the owner, no ongoing AI cost to
the owner, free to use — and the rollout plan proposed five decisions to make
that concrete. The owner took all five on 2026-09-06.

## Decision

1. **Licence: MIT, public repository.** Light's source is published under the
   MIT licence in its own public repository. This repository (the hosted
   product) stays private and is never mirrored into Light — the moat is the
   hosted service, not the desktop code. Ported modules keep their
   `PORTED-TO:` markers and manifest entries (ADR 011).

2. **Platforms at launch: Windows and macOS signed builds at beta; Linux as an
   unsigned AppImage.** macOS builds are notarised under the owner's Apple
   Developer account; Windows builds are signed (Azure Trusted Signing or an
   equivalent certificate). Linux ships unsigned with a checksum on the
   release page. These are the only recurring costs Light creates.

3. **Local model runtime: require Ollama for v1.** Light detects Ollama on
   `localhost:11434` and guides the user to install it and pull a model. No
   inference runtime is bundled in v1; a sidecar (`llama.cpp`) is a candidate
   for a later minor release, not a launch requirement.

4. **Jobserve: manual-search card, not a scraper.** The hosted product's
   self-identifying Jobserve scraper is not ported. Jobserve appears in Light
   as a manual-search card alongside the boards that already block automation
   (LinkedIn, Indeed, Glassdoor, Totaljobs …), reusing the hosted
   `manual_search_boards` pattern.

5. **Update check: on by default, with a visible toggle.** The Tauri updater
   reads a static, signed `latest.json` from GitHub Releases once per day.
   The Light privacy notice names this as the one network call the app makes
   without a user action, states what it reveals (IP address and app version
   to the manifest host) and points at the toggle in Settings.

Three invariants follow, each enforced by a test in Light's CI rather than by
policy (LESSON-033 forbid-list shape):

| Invariant | Guard |
|---|---|
| No personal data reaches an owner-controlled server: no accounts, sync, telemetry, crash upload or analytics | Network-host allowlist test over source and Tauri capabilities; dependency forbid-list for analytics/crash SDKs |
| No inference is ever billed to the owner: no default key, no relay, no owner keys in the binary | Secret scan over the *built* artefact; constructor-level test that every provider client requires a keychain key or `localhost` |
| Every feature works without paying: no licence check, trial or paywall | Forbid test on entitlement modules and on "upgrade / unlock / Pro" UI strings |

## Consequences

- **Owner cost model**: Apple Developer Program (~$99/yr) and Windows signing
  (~$120–400/yr) are the only recurring costs. Inference, downloads, updates
  and job-board quota are £0 to the owner — users bring their own keys.
- **GDPR posture**: the owner is not a controller for in-app data. The
  remaining touchpoints are the static download page, the update-manifest
  host, GitHub Issues, and the existing `WaitlistSignup` rows in the hosted
  product, which are purged after the launch invitation
  (`docs/DATA_RETENTION_SCHEDULE.md` gains a line for this).
- **Public code, private moat**: publishing Light under MIT exposes the
  ported prompts, calibration anchors and keyword scorer. This is accepted:
  the hosted product's value is the server-side pipeline, outcome data and
  cross-device sync, none of which is in Light.
- **The hosted landing page's promise must change on launch day**:
  `ProductFamilySection.jsx` "Coming soon" + waitlist becomes download links
  plus a link to the Light privacy notice; the landing contract-test pins
  move in the same PR.
- **Already public, not yet licensed**: the Light repository is
  `github.com/stevenbrady1/cviper-light`, public, with CI and a release
  workflow — the manifest's "no git remote" note was stale and is corrected in
  the same commit as this ADR. The repository carries **no `LICENSE` file**, so
  the first act under decision 1 is adding the MIT licence text; until then a
  public repository with no licence grants nobody any rights.
- **Decision 5 changes a guarded behaviour.** Light today checks for updates
  only when the user presses the button — "nothing is checked at launch" is
  pinned by two tests (single updater import site; zero calls before a button
  press) and stated in its README. Turning the check on by default means
  editing those guards and that copy deliberately, and the privacy notice
  must say so. If the owner prefers the stronger existing behaviour, decision
  5 reverts to "off by default, manual button" with no other change.
- **Manual-card Jobserve** loses live results for that board in Light. Users
  who want aggregated Jobserve results are directed to the hosted product.
- **Requiring Ollama** raises the first-run bar for the local lane. Mitigated
  by the keyless lane (deterministic ATS checks, keyword matching, writing
  coach) being fully useful before any AI is configured, and by the
  bring-your-own-key lane with the Google AI Studio free tier as the default
  recommendation.

## Alternatives considered

- **Source-available licence** (protects ported prompts): rejected — the
  competitor's trust advantage is inspectable code, and the moat is elsewhere.
- **Windows-only launch** (cheapest signing): rejected — the waitlist cohort
  includes macOS users and an unsigned macOS build is effectively
  uninstallable for non-technical users.
- **Bundled `llama.cpp` sidecar at v1**: deferred — adds 30–100 MB and a
  second runtime to support before the product has any users.
- **Port the Jobserve scraper**: rejected — a scraper running from many home
  IPs against a third party's terms is a reputational risk the owner cannot
  mitigate from the repo.
- **Update check off by default**: rejected — leaves users on stale builds
  with known bugs; disclosure plus a toggle is the honest middle.
