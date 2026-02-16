### COMMIT 40 — 2026-02-14T00:00:00Z
**Branch:** feature-eu-standalone-app
**Milestone:** Comprehensive API endpoint audit + fixes — 26/32 endpoints PASS (up from 12/29). Auth bug fixed, web search wired, PyJWT deployed, get-documents auth fixed.
**State:** WORKING

**Summary:**
- Fixed "Loading user profile..." auth bug by decoding JWT directly instead of fetchUserAttributes()
- Wired web search toggle end-to-end (frontend → backend → EUWebSearch Lambda)
- Fixed Deep Research hardcoded US endpoint → EU API Gateway
- Fixed URL mismatches (/insights → /get-insights, /doc-insights → /get-document-insights)
- Fixed EUGetDeepDocuments auth header extraction (legacy event.params.header → event.headers)
- Deployed eu_chat_athena_bot and eu_get_deep_documents with PyJWT dependency
- Comprehensive test: 26 PASS / 2 WARN / 4 FAIL (all 4 are expected/infra issues)

**Remaining:**
- Configure get-insights Lambda with Neo4j + OpenAI env vars
- Push backend fixes to repo
- Verify auth on live Amplify
- Phase 18: Production launch
