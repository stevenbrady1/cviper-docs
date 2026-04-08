# ADR-002: Priority-Based AI Routing

**Status**: accepted (updated 2026-03-30 — replaced 3-tier model with priority-based routing)
**Date**: 2026-02 (original), 2026-03 (revised)
**Decision makers**: Project owner

## Context

CViper uses multiple AI providers (OpenAI, Claude, Gemini, Grok, Mistral, GitHub Models, OpenRouter, Ollama) plus Pluribus (local gateway only) for different tasks. Relying on a single provider creates vendor lock-in and single points of failure. The system needs automatic failover, user-configurable preferences, and graceful degradation.

## Decision

Implement **priority-based routing with automatic fallback**: the TaskRouter maintains a user-configurable ordered list of providers (DEFAULT_PRIORITY). For each AI call, it selects the highest-priority available provider. If that provider fails, the gateway tries the next in the chain. When all providers are exhausted, 22 keyword-based fallback methods provide basic results.

- **Priority chain**: pluribus → anthropic → google → openai → grok → openrouter → mistral → github → ollama (configurable per user)
- **Circuit breakers**: providers with recent failures are temporarily skipped
- **Sandbox isolation**: sandbox users route to dedicated sandbox_google → sandbox_openrouter → keyword (never production providers)
- **Keyword fallback**: 22 methods in FallbackService cover every AI operation with template-based results

## Consequences

**Positive**:
- No vendor lock-in: any provider can be first in the chain
- Automatic failover: provider outages are transparent to the user
- User control: drag-and-drop priority ordering in settings
- Graceful degradation: keyword fallback means no blank screens, ever

**Negative**:
- All providers see the same tasks (no cost optimization by task complexity)
- Priority ordering requires user judgement about provider quality
- Provider-specific quirks need per-provider handling in the gateway

**Implementation**:
- `ai/router.py` — `TaskRouter` with `DEFAULT_PRIORITY` list and `select_provider()` method
- `ai/gateway.py` — `AIGateway` dispatches calls with circuit breaker and `extract_json()` normalization
- `ai/fallbacks.py` — `FallbackService` with 22 keyword-based fallback methods

**Supersedes**: Original 3-tier routing (premium/standard/light) which mapped tasks to tiers. The tier infrastructure remains in code for backwards compatibility but is not actively used — routing defaults to flat priority order.
