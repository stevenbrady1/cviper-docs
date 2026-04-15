# ADR-006: Async Task Queue for AI Operations

**Status**: proposed
**Date**: 2026-04-12
**Decision makers**: Project owner
**Backlog**: CV-161 (#228), CV-162 (#229)

## Context

CViper's AI operations (CV analysis, document generation, ATS scoring, multi-provider comparison) block FastAPI workers for 30-120 seconds per request. With `maxReplicas: 3` on Azure Container Apps, 3 concurrent AI requests exhaust all workers, causing other endpoints (search, config, health checks) to queue behind them.

Current mitigations (`asyncio.to_thread()`) prevent event loop blocking but still hold a thread pool worker for the full duration. `BackgroundTasks` is used for fire-and-forget jobs (link tracking) but doesn't support status polling, cancellation, or result retrieval.

**10 endpoints are affected**, with the heaviest being `multi-generate` (multiple parallel AI calls), `analyze-cv-multi` (multi-provider chain), and `documents/{id}/alternative` (CV + cover letter generation).

## Options Considered

### Option A: In-process task registry (dict + asyncio)
- Store tasks in an in-memory `dict[task_id, TaskState]`
- Run AI work in `asyncio.create_task()` or thread pool
- Expose `GET /api/tasks/{id}` for polling, `DELETE` for cancellation
- **Pros**: Zero new infrastructure, fast to implement, no Redis/broker dependency
- **Cons**: Tasks lost on container restart/scale, no cross-replica visibility, limited concurrency control

### Option B: ARQ (Async Redis Queue)
- Python-native async Redis queue, lightweight (~500 LOC core)
- Separate worker process reads from Redis, runs AI tasks
- Results stored in Redis with TTL
- **Pros**: Persistent across restarts, cross-replica task visibility, built-in retries/timeouts, async-native
- **Cons**: Requires Redis (Azure Cache for Redis ~$15/month), separate worker container, more infra complexity

### Option C: Celery + Redis/RabbitMQ
- Industry standard distributed task queue
- **Pros**: Battle-tested, rich ecosystem, priority queues, rate limiting
- **Cons**: Heavy dependency, sync-first design (awkward with async FastAPI), complex configuration, overkill for current scale

### Option D: Azure Queue Storage + Functions
- Cloud-native: Azure Queue Storage triggers Azure Functions
- **Pros**: Serverless scaling, no worker management, pay-per-execution
- **Cons**: Cold starts (5-30s), vendor lock-in, separate codebase for workers, debugging harder, doesn't fit the existing FastAPI structure

## Decision

**Phase 1 (this cycle): Option A — In-process task registry**
**Phase 2 (when load testing reveals need): Migrate to Option B — ARQ**

### Rationale
- Current scale (1-3 replicas, single user sessions) doesn't justify Redis infrastructure
- In-process registry solves the UX problem (non-blocking operations, progress polling) immediately
- The task registry interface (`submit_task`, `get_task`, `cancel_task`) is designed to be backend-agnostic — swapping the in-memory dict for ARQ Redis later requires changing only the storage layer, not the API contract
- Phase 1 delivers 90% of the user-facing value (responsive UI) at 10% of the infrastructure cost

### Phase 1 Design

```
POST /api/tasks/submit
  body: { "operation": "analyze-cv", "params": {...} }
  returns: 202 { "task_id": "uuid", "status": "pending" }

GET /api/tasks/{task_id}
  returns: { "task_id": "uuid", "status": "running|complete|failed", 
             "progress": 0.6, "result": {...} | null, "error": "..." | null }

DELETE /api/tasks/{task_id}
  returns: 204 (cancellation requested)
```

**Architecture**:
```
┌─────────────┐     ┌──────────────────┐     ┌────────────────┐
│ FastAPI      │────▶│ TaskRegistry     │────▶│ asyncio.Task   │
│ Route Handler│     │ (in-memory dict) │     │ (thread pool)  │
└─────────────┘     └──────────────────┘     └────────────────┘
       │                     │
       │ 202 Accepted        │ status updates
       ▼                     ▼
   Client polls          TaskState enum
   GET /tasks/{id}       pending → running → complete|failed
```

**Concurrency control**: Max 5 concurrent AI tasks per replica (configurable). Additional submissions queued in-memory. This prevents worker exhaustion while allowing parallel AI operations.

**TTL**: Completed/failed task results expire after 30 minutes. Cleanup loop runs every 5 minutes.

**Cancellation**: Sets a cancellation flag checked between AI pipeline steps. Long-running single-model calls cannot be interrupted mid-flight but multi-step chains (e.g., 3-step CV pipeline) check between steps.

### Phase 2 Migration Path (when needed)
- Add Azure Cache for Redis (~$15/month Basic C0)
- Replace in-memory dict with ARQ task storage
- Split worker into separate container in Bicep
- Same API contract — frontend unchanged

## Consequences

**Positive**:
- AI operations no longer block FastAPI workers — other endpoints remain responsive
- Users see progress indicators and can cancel long-running operations
- Foundation for multi-step AI pipelines with intermediate progress
- Clean migration path to distributed queue when scale demands it

**Negative**:
- In-memory tasks lost on container restart (acceptable — tasks are short-lived, 30-120s)
- No cross-replica task visibility in Phase 1 (acceptable — Azure sticky sessions route user to same replica)
- Additional complexity in error handling (task failure vs HTTP error)

**Risks**:
- Memory pressure from too many queued tasks — mitigated by max queue depth + TTL cleanup
- Race conditions in task state transitions — mitigated by asyncio single-threaded event loop (no locks needed for dict access)
