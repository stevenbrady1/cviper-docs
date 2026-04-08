# ADR-005: Fatal Startup Guard Rules

**Status**: accepted
**Date**: 2026-03
**Decision makers**: Project owner (post-incident)

## Context

In March 2026, a `sys.exit()` guard was added to the backend to check for a required environment variable. The variable was added to the Python code but **not** to the Azure Bicep template, deploy.sh, or CI Docker smoke test. This caused the container to crash on every deploy for 4 days because the variable was never set in production.

## Decision

Establish **Fatal Startup Guard Rules** as a non-negotiable policy:

Any code that adds `SystemExit`, `sys.exit()`, or a fatal startup check **MUST** include in the **same commit or PR**:
1. The env var added to `azure/container-apps.bicep` backend env array
2. The env var added to `azure/deploy.sh` (require_secret or parameter passing)
3. The CI Docker smoke test updated to include the variable
4. The `test_deploy_config.sh` parity check updated if needed

## Consequences

**Positive**:
- Impossible to deploy a container that crashes on startup due to missing env vars
- Parity check script (`test_deploy_config.sh`) catches drift between Bicep, deploy.sh, and CI
- Documented as LESSON-001 in the lessons library for cross-conversation enforcement

**Negative**:
- Adds friction to any startup-guard change (4 files minimum)
- Requires discipline to update all files atomically

**Enforcement**:
- CLAUDE.md auto-correction rule #3: "New env var without Deployment Checklist → Stop"
- `test_deploy_config.sh` runs in CI and locally
- Lessons library records the original incident for recurrence checking
