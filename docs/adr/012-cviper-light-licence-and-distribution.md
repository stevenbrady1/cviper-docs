# ADR 012 — CViper Light: free, MIT-licensed, local-only distribution

**Status**: accepted (amended 2026-09-12 — Windows code-signing route corrected)
**Date**: 2026-09-06 (original), 2026-09-12 (amended)
**Source**: Owner decisions of 2026-09-06 on `ClaudeReports/summaries/2026-09-06-summary-cviper-light-free-rollout-plan.md` §7 (itself a follow-up to the 2026-09-05 CVAurum competitive analysis)
**Related**: [ADR 011](011-two-product-architecture.md) (two products, one brand), [ADR 002](002-ai-tier-routing.md) (AI routing — the hosted product's model, which Light deliberately does not inherit)

> **Amendment 2026-09-12**: decision 2 and the first Consequences bullet named
> Azure Trusted Signing and priced Windows signing at ~$120–400/yr. Both are
> wrong for this owner. The service was renamed Azure Artifact Signing and is
> closed to individual developers outside the USA and Canada; and since 2024 no
> certificate of any grade removes the SmartScreen warning by itself. The
> licence, the platform set and the other four decisions are unchanged. See
> [Amendment (2026-09-12)](#amendment-2026-09-12--windows-code-signing) below.

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
   unsigned AppImage.** *(Original text, retained for the record — superseded
   in part by the [2026-09-12 amendment](#amendment-2026-09-12--windows-code-signing):
   the Windows route named below is not available to this owner, and the cost
   figure is wrong.)* macOS builds are notarised under the owner's Apple
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

- **Owner cost model** *(original text, retained for the record — superseded in
  part by the [2026-09-12 amendment](#amendment-2026-09-12--windows-code-signing);
  the corrected figure is $99/yr, already paid)*: Apple Developer Program
  (~$99/yr) and Windows signing (~$120–400/yr) are the only recurring costs.
  Inference, downloads, updates and job-board quota are £0 to the owner — users
  bring their own keys.
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

## Amendment (2026-09-12) — Windows code signing

Decision 2 and the first Consequences bullet were checked against primary
sources on 2026-09-11/12. Three claims did not survive. The decision to ship
Windows and macOS at beta stands; the Windows *mechanism* and the cost model
are corrected here.

### 1. Azure Trusted Signing cannot be bought by this owner

The service was renamed **Azure Artifact Signing**. Microsoft Learn, *"Code
signing options for Windows app developers"* (`ms.date` 2026-08-29), states:

> "Geographic limitation: Azure Artifact Signing is available to organizations
> in the USA, Canada, the European Union, and the United Kingdom. Individual
> developers are currently limited to the USA and Canada. If you are an
> individual developer outside those regions, see OV certificates below."

The owner is a UK **individual**, not a registered company, so this route is
closed today. A UK limited company *would* be eligible — incorporation costs
~£50 and would unlock Azure Artifact Signing at ~$120/yr. The older
"organisation must be 3+ years old" rule appears to have been removed (a
Microsoft employee stated there are "no minimum org age restrictions" on
2026-08-17), but that is moot here: the blocker is geography plus individual
status, not age.

### 2. Buying a certificate does not remove the SmartScreen warning

Microsoft Learn, *"SmartScreen reputation for Windows app developers"*
(`ms.date` 2026-05-04, updated 2026-08-17), states:

> "EV certificates no longer bypass SmartScreen. Years ago, signing files with
> an Extended Validation (EV) code signing certificate would result in positive
> SmartScreen reputation by default, but this behavior no longer exists. […]
> Paying a premium for EV solely to avoid SmartScreen warnings is no longer
> justified."

That behaviour was removed in 2024. A signed OV or IV build still warns until
reputation accrues organically — Microsoft describes this as several weeks and
hundreds of clean installs, with no mechanism to request expedited review. The
implicit premise of the original decision (buy a certificate, get a non-scary
install) is therefore false for Windows.

### 3. The free route that does work is the Microsoft Store

Same source (`ms.date` 2026-08-29):

> "If you publish your app as an MSIX package through the Microsoft Store, code
> signing is free and handled for you automatically […] users never see a
> SmartScreen warning."

The individual developer account is now free — the former $19 registration fee
was waived; registration is via `storedeveloper.microsoft.com` and requires ID
and selfie verification. Two caveats against Light's Tauri v2 stack (ADR 011):
Tauri cannot emit MSIX natively and needs Microsoft's `winapp` CLI (public
preview at the time of writing); and a Store build cannot use the in-app
updater, because the Store handles its own updates — which bears directly on
decision 5.

### 4. macOS — unchanged in substance, but name the certificate

macOS notarisation remains correct as decided. The certificate is a **Developer
ID Application** certificate, which is a *different* certificate from the
**Apple Distribution** one already used for iOS/TestFlight. It is included in
the existing $99/yr Apple Developer Program membership at no extra cost, is
available to Individual (not only Organization) memberships, and notarisation
itself is free.

### Corrected cost model

| Platform | Route | Recurring cost |
|---|---|---|
| macOS | Developer ID Application certificate + notarisation, under the existing membership | **$99/yr — already paid** |
| Windows | Microsoft Store MSIX; Microsoft signs it, and no SmartScreen warning is shown | **£0 / $0** |
| Linux | unsigned AppImage + checksum (unchanged) | £0 |

The realistic recurring cost of Light's distribution is therefore **$99/yr,
which the owner already pays** — not the $219–499/yr the original bullet
implied.

If a Windows certificate is still wanted for the direct download (which the
Store route does not cover), the cheapest eligible options are:

- **Certum Open Source Code Signing** — ~€49–69/yr. Light is MIT-licensed, so
  it qualifies on the open-source test. *(UNCONFIRMED: Certum's published terms
  on commercial use of their open-source certificate could not be verified
  first-hand. Check before purchase.)*
- **SSL.com IV (Individual Validation)** — ~$129/yr, plus roughly $180/yr more
  for CI/automated signing.
- **Incorporate a UK Ltd company** (~£50) — unlocks Azure Artifact Signing at
  ~$120/yr.

None of these removes the SmartScreen warning on day one (see §2). They buy
attribution and the start of a reputation clock, not a clean install.

### Recorded as unconfirmed

- Whether an **individual** Microsoft Partner Center account can complete the
  Microsoft Entra tenant association required for automated/CI submission to
  the Store — **unconfirmed**. If it cannot, Store releases may have to be
  uploaded by hand.
- Certum's terms on commercial use of their open-source certificate, as above —
  **unconfirmed**.

**Sources:** Microsoft Learn, *"Code signing options for Windows app
developers"* (`ms.date` 2026-08-29); Microsoft Learn, *"SmartScreen reputation
for Windows app developers"* (`ms.date` 2026-05-04, updated 2026-08-17);
Microsoft employee statement on organisation age, 2026-08-17. All verified
2026-09-11/12.

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
