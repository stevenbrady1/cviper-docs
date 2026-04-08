# ADR-001: Dual Database Strategy (SQLite + PostgreSQL)

**Status**: accepted
**Date**: 2026-02
**Decision makers**: Project owner

## Context

CViper needs a database for local development, CI testing, and production. Options:
1. PostgreSQL everywhere (requires local install + CI service container)
2. SQLite everywhere (limited for production)
3. Dual: SQLite for local dev/CI, PostgreSQL for production

## Decision

Use **dual database strategy**: SQLite (local dev, CI, Docker smoke tests) and PostgreSQL (production via Azure Flexible Server). SQLAlchemy ORM abstracts the dialect differences.

## Consequences

**Positive**:
- Zero-dependency local setup (no PostgreSQL install needed)
- Fast CI (in-memory SQLite with `StaticPool`)
- Production-grade PostgreSQL with managed backups

**Negative**:
- Dialect mismatches are a constant risk (e.g., `batch_alter_table()` required for SQLite constraint operations)
- Must test migrations against both dialects
- Developers must be vigilant about PostgreSQL-only features (partial indexes, JSON operators)

**Mitigations**:
- CLAUDE.md rule: all dev personas must flag SQLite/PostgreSQL mismatches
- Memory file `feedback_dual_db_risk.md` reminds Claude across conversations
- CI runs tests on SQLite; production uses PostgreSQL
- LESSON-003 in lessons library documents a real incident from this trade-off
