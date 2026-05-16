# Architecture Decision Records (ADRs)

This folder records significant architecture decisions for CViper.

Each ADR captures the context, decision, and consequences so future contributors understand *why* the system is built this way — not just *how*.

## Format

Each ADR follows this structure:
- **Status**: proposed | accepted | superseded | deprecated
- **Context**: What problem or constraint prompted the decision
- **Decision**: What we chose and why
- **Consequences**: Trade-offs, risks, and follow-up actions

## Index

| ADR | Title | Status | Date |
|-----|-------|--------|------|
| [001](001-dual-database-strategy.md) | Dual database strategy (SQLite + PostgreSQL) | accepted | 2026-02 |
| [002](002-ai-tier-routing.md) | Priority-based AI routing with fallback chain | accepted | 2026-03 |
| [003](003-sandbox-abuse-prevention.md) | Sandbox abuse prevention (5-layer strategy) | accepted | 2026-03 |
| [004](004-state-based-navigation.md) | State-based tab navigation (no React Router) | accepted | 2026-01 |
| [005](005-fatal-startup-guard-rules.md) | Fatal startup guard rules | accepted | 2026-03 |
| [006](006-async-task-queue.md) | Async task queue for AI operations | proposed | 2026-04 |
| [007](007-cv-snapshot-vs-latest-analysis.md) | cv_snapshot vs latest CV analysis (Job Search drift class) | accepted | 2026-05 |
