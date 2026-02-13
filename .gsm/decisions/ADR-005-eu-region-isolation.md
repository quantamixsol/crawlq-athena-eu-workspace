# ADR-005: EU Region Isolation Strategy
**Date:** 2026-02-09 | **Status:** ACCEPTED
**Context:** EU Chat Athena must operate entirely in eu-central-1 without impacting existing US flows. Need clear separation at infrastructure, code, and routing levels.
**Decision:** Multi-layer isolation:
1. **Infrastructure:** All Lambda functions prefixed `eu_`, DynamoDB tables prefixed `eu-`, S3 bucket `eu-deep-document-analysis-bucket`, separate Cognito user pool
2. **Frontend:** `IS_EU` flag from `NEXT_PUBLIC_REGION=eu` in region-config.ts, all EU endpoints in separate config block
3. **Routing:** Middleware redirects EU users to `/chat-athena-eu`, separate page.tsx from US `/chat-athena`
4. **Code:** EU Lambda source in `SemanticGraphEU/` directory, shared modules imported from `shared/`
5. **CI/CD:** Separate GitHub Actions workflow `deploy-eu-lambdas.yml`
**Consequences:**
- (+) Zero impact on US flows — completely separate infrastructure
- (+) GDPR compliance — data never leaves eu-central-1
- (+) Independent deployment cycles
- (-) Code duplication between US and EU Lambda handlers (acceptable for regulatory isolation)
