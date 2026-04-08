# ADR-004: State-Based Tab Navigation (No React Router)

**Status**: accepted
**Date**: 2026-01
**Decision makers**: Project owner

## Context

CViper is a single-page application. Options for navigation:
1. React Router (URL-based routing)
2. State-based tab navigation (`useState` with `activeTab`)

## Decision

Use **state-based tab navigation** via `useState('search' | 'applications' | 'config' | 'companies' | 'logs')`. No React Router dependency.

## Consequences

**Positive**:
- Simpler mental model — one state variable controls the entire view
- No URL sync complexity, no route guards, no nested route configuration
- Smaller bundle (no react-router-dom dependency)
- Works naturally with the sidebar navigation pattern

**Negative**:
- No deep linking (users can't bookmark a specific tab)
- Browser back/forward buttons don't navigate between tabs
- All tab components mount/unmount on switch (no route-level code splitting)

**Mitigation**:
- Deep linking is not a priority for a personal productivity tool
- Tab state is lightweight; mount/unmount is fast with React 18
