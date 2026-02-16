# ADR-042: EU System End-to-End Inventory & Configuration Audit

**Date:** 2026-02-16 | **Status:** ACCEPTED (Living Document)

## Context

After completing full Lambda Function URL migration (COMMIT 15) and removing EU_ATHENA_TRAINING_FUNCTION references, we need a single-source-of-truth for the entire EU system wiring: which Lambda handles which endpoint, what payload each expects, how the frontend calls them, and whether Amplify env vars match.

Without this inventory, configuration drift causes silent failures (e.g., missing env vars, stale function references, broken integrations).

## Decision

Maintain a living E2E System Inventory at `.gsm/external/2026-02-16_EU-System-E2E-Inventory.md` that documents:

1. **All 36 Lambda functions** — handler, runtime, timeout, memory, Function URL, auth type
2. **Frontend-to-Lambda mapping** — which region-config.ts key maps to which Lambda
3. **API Gateway routes** — preserved as fallback, 5 JWT-protected
4. **Handler patterns** — 6 invocation patterns (normalize_event, direct, async, fire-and-forget, SQS, cron)
5. **Amplify env vars** — cross-referenced against code `process.env` usage
6. **Lambda inter-invocation map** — which Lambda calls which
7. **Test infrastructure** — all test frameworks, files, coverage gaps

### Findings Requiring Action

| # | Finding | Severity | Action |
|---|---------|----------|--------|
| 1 | conftest.py has stale `CREATE_PROJECT_FUNCTION=eu_create_project_proxy` | HIGH | Fix to `eu_create_project` |
| 2 | conftest.py has removed `ATHENA_TRAINING_FUNCTION=eu_test_semantic` | HIGH | Remove from fixture |
| 3 | Friction emit route missing AWS credentials in Amplify | HIGH | Add env vars or use DYNAMO_* keys |
| 4 | POST /chat API Gateway integration fk0y5gi broken | MEDIUM | Investigate or remove stale route |
| 5 | 8 orphan Amplify env vars | LOW | Clean up |
| 6 | .env.eu.example has stale Function URLs | LOW | Update or remove |
| 7 | No CI/CD pipeline | MEDIUM | Set up GitHub Actions |
| 8 | Frontend unit tests all inactive | MEDIUM | Activate or remove jest config |

### Update Protocol

This inventory MUST be updated when:
- A Lambda is deployed/redeployed (new Function URL possible)
- A new endpoint is added to region-config.ts
- An Amplify env var is added/removed
- A handler's input/output contract changes

## Consequences

**Positive:**
- Single source of truth for all system wiring
- Drift detection becomes trivial (grep this file)
- New team members can understand the full system in <10 minutes
- Cross-referencing finds bugs (e.g., stale conftest.py values found immediately)

**Negative:**
- Requires discipline to update after every deployment
- Document can become stale if not maintained
- Large document (~500 lines) adds to GSM overhead

**Mitigation:**
- GCC COMMIT protocol includes "check inventory for changes"
- Summary file at ~200 tokens for quick reference
- Living document status means it's expected to change
