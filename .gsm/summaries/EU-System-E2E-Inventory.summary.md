---
source: .gsm/external/2026-02-16_EU-System-E2E-Inventory.md
added: 2026-02-16
type: architecture
tags: [lambda, api-gateway, amplify, inventory, endpoints, testing, e2e]
---

## Key Points
1. **36 EU Lambda functions** in eu-central-1, all with Function URL AuthType=NONE. 30 use py3.10, 4 use py3.9, 1 py3.11, 1 py3.12. All bypass API Gateway 30s timeout.
2. **30 frontend endpoints** mapped in region-config.ts — 29 use direct Lambda Function URLs, 1 (workspaces) still on API Gateway. Frontend uses React Query with smart cache invalidation.
3. **25 API Gateway routes** remain as fallback — 5 JWT-protected, 20 NONE auth. POST /chat has a broken integration (fk0y5gi missing).
4. **8 orphan Amplify env vars** set but never read by code. 2 missing env vars cause friction emit route to fail. conftest.py has 2 stale env vars.

## Requirements Extracted
- [ ] Remove 8 orphan Amplify env vars to reduce config noise
- [ ] Set AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY in Amplify for friction emit route (or refactor to use DYNAMO_* keys)
- [ ] Fix conftest.py: change eu_create_project_proxy→eu_create_project, remove ATHENA_TRAINING_FUNCTION
- [ ] Fix broken API Gateway POST /chat integration (fk0y5gi)
- [ ] Clean up orphaned API Gateway integrations (5 integrations with no routes)
- [ ] Consider making Lambda Function URLs configurable via env vars instead of hardcoded
- [ ] Set up CI/CD pipeline (no GitHub Actions currently)

## Numbers That Matter
- 36 Lambda functions, 13.4MB largest (eu_onboard_user)
- 30 frontend endpoints, 75+ Pallas visual test scenarios
- 360+ backend test cases across 89 test files
- 8 orphan Amplify vars, 2 missing critical vars
- 0% frontend unit test coverage (all commented out)

## Cross-References
- Related to: ADR-001 (Function URLs), ADR-010 (API Gateway), ADR-021 (Deployment), ADR-025 (Testing), ADR-035 (Pallas)
- Living document: update when Lambda deployments or endpoint wiring changes
