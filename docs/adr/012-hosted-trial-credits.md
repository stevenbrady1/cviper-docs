# ADR-012 — Hosted trial credits replace the bring-your-own-key wall

**Status**: Accepted · 2026-09-05
**Relates to**: BRD assumption A-001 (retired by this decision), CV-1369

## Context

BRD assumption A-001 assumed every user can obtain an AI provider API key
before their first result. In practice the key wall is CViper's largest
first-session drop-off: competitor products are zero-setup, and our own
"New to AI" onboarding work (CV-1473/#1478) exists precisely because the
wall confuses the audience the product most wants to serve. The demo
(sandbox) path proves hosted inference is operable within budget: sandbox
users already route through operator-held keys with quotas.

## Decision

Give every Free account a small, one-time bundle of hosted trial credits:

- Config row `trial_credits` (global): `{enabled, operations:
  {analyze_cv: 1, score_job: 3, generate_documents: 1}, provider:
  "google", daily_cap_usd: 10}` — defaults layered under stored values
  (rule #52), admin-editable.
- A Free user with **no personal key** consuming one of those operations
  with credit remaining is allowed through `UsageLimitMiddleware`, served
  via the operator's admin key (the existing gateway fallback path), and
  the per-user counter (Config row `trial_usage`, Postgres — rule #41)
  is decremented on success.
- Exhausted credits return the existing soft-wall 429 shape plus two CTAs:
  "add your free key" (existing key guide) and "go Pro".
- Trial traffic is capped globally at `daily_cap_usd` (default 10 USD/day,
  measured from the gateway's `estimated_cost_usd` call meta, accumulated
  in a row-locked Config row — rule #55). At the cap the soft wall returns
  immediately and `trial_credits_daily_cap_hit` is logged.
- BYO-key users and Pro users are entirely unaffected (their existing
  bypasses run first).

## Consequences

- The operator's provider account processes trial users' CV content:
  DPIA Step 2 records the admin-key path; ToS gains a sentence on
  trial processing via the operator's account.
- A-001 becomes a decision, not an assumption: "Users can reach their
  first result on hosted trial credits; sustained use requires their own
  key (free tiers available) or Pro."
- Cost exposure is bounded by (credits × op cost) per user and the global
  daily cap; abuse via re-registration shares the registration rate
  limits and the global cap.
- The landing "New-to-AI" card can honestly say "Try it first — no key
  needed for your first analysis."

## Alternatives considered

- **Time-boxed full trial (all ops, 24h)** — unbounded cost per user,
  harder to explain than counted credits.
- **Demo-account-only trial** — already exists, but results are not
  portable to the user's real account; conversion data says users who
  invest their own CV convert better.
- **Keep the wall, improve key onboarding** — shipped (CV-1478); drop-off
  remained the top funnel issue.
