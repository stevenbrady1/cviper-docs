# ADR-003: Sandbox Abuse Prevention (5-Layer Strategy)

**Status**: accepted
**Date**: 2026-03 (updated 2026-03-29)
**Decision makers**: Project owner

## Context

CViper offers a public sandbox mode for unauthenticated users to try the app. Without controls, this exposes AI API calls to abuse (token farming, prompt injection, resource exhaustion).

## Decision

Implement a **5-layer abuse prevention strategy**:

1. **Browser fingerprint + IP session limits**: Track sandbox usage per device fingerprint (50 sessions/day) and per IP address (100 sessions/day) to prevent session recycling
2. **IP rate limiting**: Limit sandbox AI calls per IP address per time window
3. **Dedicated sandbox providers with daily quotas**: Sandbox users route exclusively to `sandbox_google` (Gemini 2.0 Flash, 1000 req/day) and `sandbox_openrouter` (GPT-4o-mini, 50 req/day) with separate API keys — never to production providers. Falls back to keyword matching when quotas exhaust
4. **Truncated outputs**: AI responses in sandbox mode are truncated (e.g., 3 sub-scores instead of all, 300-char rationales, 2 interview questions) to demonstrate value without giving away the full product
5. **1-hour session auto-expiry**: Sandbox sessions expire after 1 hour with background cleanup every 5 minutes, preventing long-running abuse

## Consequences

**Positive**:
- Cost is bounded by daily quotas on cheap models (Gemini Flash, GPT-4o-mini)
- Users get a genuine taste of the product without full access
- Multiple layers mean no single point of bypass
- Session expiry limits the abuse window

**Negative**:
- Sandbox experience is noticeably limited (intentional, but may reduce conversion)
- Fingerprint tracking raises privacy considerations (mitigated: no PII stored, session-scoped only)
- Requires dedicated sandbox API keys to be provisioned and monitored

**Implementation**:
- `backend/main.py` — `SandboxAbusePrevention` middleware with named truncation constants
- `backend/ai/router.py` — sandbox user detection routes to `sandbox_google`/`sandbox_openrouter` only
- `backend/provider_visibility.py` — blocks non-sandbox providers for sandbox users
- `backend/deps.py` — per-provider daily quota enforcement
- `backend/routes/auth.py` — fingerprint + IP session creation limits
- Frontend: `SandboxWelcomeModal.jsx` explains limitations upfront
