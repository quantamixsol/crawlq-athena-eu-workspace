### COMMIT 16 — 2026-02-14T02:30:00Z (Canvas Session Part 3)
**Milestone:** DynamoDB credentials FIXED — All 5 API endpoints working on deployed Amplify app, full CRUD verified with real Cognito auth

**State:** DONE

**Root Cause:** Amplify WEB_COMPUTE SSR Lambda does not inherit the Amplify service role credentials. Server-side env vars also not available at SSR runtime.

**Fix:** Created IAM user `canvas-dynamodb-service` with scoped DynamoDB access, shared DynamoDB client (`src/lib/dynamodb.ts`) using `DYNAMO_*` env vars, updated `amplify.yml` to write `.env.production` during build.

**Verification:** All 5 API endpoints passing on https://main.d1tnt2fg41rrrv.amplifyapp.com with real Cognito auth (support@quantamixsolutions.com).

**Next:** Browser testing, connect real LLM, Sprint 3 features
