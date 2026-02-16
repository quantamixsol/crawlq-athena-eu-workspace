# CrawlQ Athena EU — End-to-End System Inventory

**Date:** 2026-02-16 (Updated: 2026-02-16 15:00 UTC)
**Version:** 2.0
**Scope:** 36 Lambda functions, 25 API Gateway routes, 30 frontend endpoints, Amplify config, test infrastructure
**Region:** eu-central-1 | **Account:** 680341090470

---

## 1. LAMBDA FUNCTION INVENTORY (36 total)

| # | Lambda Name | Handler | Runtime | Timeout | Memory | Code KB | Function URL AuthType | Frontend Endpoint Key |
|---|-------------|---------|---------|---------|--------|---------|----------------------|----------------------|
| 1 | eu_audit_trail_store | handler.lambda_handler | py3.10 | 300s | 512MB | 40 | NONE | auditTrailStore |
| 2 | eu_audit_trail_verify | handler.lambda_handler | py3.10 | 300s | 512MB | 40 | NONE | auditTrailVerify |
| 3 | eu_chat_athena_bot | handler.lambda_handler | py3.10 | 120s | 1024MB | 77 | NONE | chatAthenaBot |
| 4 | eu_chat_job_queue | handler.lambda_handler | py3.10 | 30s | 512MB | 41 | NONE | chatJobQueue |
| 5 | eu_chat_job_status | handler.lambda_handler | py3.10 | 10s | 256MB | 40 | NONE | chatJobStatus |
| 6 | eu_chat_job_worker | handler.lambda_handler | py3.10 | 900s | 3008MB | 45 | NONE | (SQS trigger, no frontend) |
| 7 | eu_compliance_engine | handler.lambda_handler | py3.10 | 300s | 512MB | 64 | NONE | complianceEngine |
| 8 | eu_confirm_signup | lambda_function.lambda_handler | py3.10 | 30s | 256MB | 1 | NONE | confirmSignup |
| 9 | eu_consent_manager | handler.lambda_handler | py3.10 | 300s | 512MB | 63 | NONE | consentManager |
| 10 | eu_conversation_memory | handler.lambda_handler | py3.10 | 60s | 256MB | 41 | NONE | (async fire-and-forget) |
| 11 | eu_create_project | handler.lambda_handler | py3.10 | 30s | 256MB | 1 | NONE | createProject |
| 12 | eu_create_session | handler.handler | py3.9 | 10s | 256MB | 39 | NONE | createSession |
| 13 | eu_deep_graph_builder | handler.lambda_handler | py3.10 | 300s | 1024MB | 52 | NONE | (internal, invoked by other Lambdas) |
| 14 | eu_deep_research | handler.lambda_handler | py3.10 | 900s | 512MB | 43 | NONE | deepResearch |
| 15 | eu_deep_research_status | handler.lambda_handler | py3.10 | 10s | 256MB | 40 | NONE | (polled via deepResearch) |
| 16 | eu_friction_event_batcher | handler.handler | py3.11 | 60s | 512MB | 97 | NONE | (EventBridge cron) |
| 17 | eu_generate_deep_insights | handler.lambda_handler | py3.10 | 900s | 2048MB | 5254 | NONE | (async fire-and-forget) |
| 18 | eu_get_chat_history | handler.lambda_handler | py3.10 | 30s | 256MB | 64 | NONE | getChatHistory |
| 19 | eu_get_deep_documents | handler.lambda_handler | py3.10 | 300s | 512MB | 65 | NONE | getDeepDocuments |
| 20 | eu_get_deep_insights | handler.lambda_handler | py3.10 | 60s | 512MB | 298 | NONE | getDeepInsights |
| 21 | eu_get_document_insights | handler.lambda_handler | py3.10 | 300s | 512MB | 41 | NONE | getDocumentInsights |
| 22 | eu_get_user_archetype | handler.handler | py3.9 | 10s | 256MB | 39 | NONE | getArchetype |
| 23 | eu_list_campaigns | handler.lambda_handler | py3.10 | 15s | 128MB | 1 | NONE | campaigns |
| 24 | eu_list_sessions | handler.handler | py3.9 | 10s | 256MB | 40 | NONE | listSessions |
| 25 | eu_onboard_user | handler.lambda_handler | py3.10 | 60s | 256MB | 13765 | NONE | onboardUser |
| 26 | eu_process_deep_document | handler.lambda_handler | py3.10 | 300s | 512MB | 41 | NONE | (internal, invoked by upload) |
| 27 | eu_query_usage | handler.lambda_handler | py3.10 | 15s | 128MB | 1 | NONE | queryUsage |
| 28 | eu_reasoner | handler.lambda_handler | py3.10 | 300s | 1024MB | 1510 | NONE | reasoner |
| 29 | eu_register | lambda_function.lambda_handler | py3.10 | 30s | 256MB | 64 | NONE | register |
| 30 | eu_resend_code | lambda_function.lambda_handler | py3.10 | 30s | 256MB | 1 | NONE | resendCode |
| 31 | eu_response_kg_extractor | handler.lambda_handler | py3.10 | 60s | 512MB | 68 | NONE | responseKG |
| 32 | eu_save_user_archetype | handler.handler | py3.9 | 10s | 256MB | 40 | NONE | saveArchetype |
| 33 | eu_subscription | handler.lambda_handler | py3.12 | 15s | 128MB | 64 | NONE | subscription |
| 34 | eu_trace_explainer | handler.lambda_handler | py3.10 | 300s | 512MB | 49 | NONE | traceExplainer |
| 35 | eu_upload_deep_document | handler.lambda_handler | py3.10 | 60s | 512MB | 75 | NONE | uploadDeepDocument |
| 36 | eu_web_search | handler.lambda_handler | py3.10 | 30s | 256MB | 97 | NONE | webSearch |

---

## 2. FRONTEND ENDPOINT → LAMBDA MAPPING

| Frontend Key | Lambda Function URL | HTTP Method | Auth Pattern | Request Shape |
|-------------|-------------------|-------------|-------------|---------------|
| register | ndycr36k...on.aws | POST | None | {email, name, password} |
| confirmSignup | vwz52t6f...on.aws | POST | None | {email, code} |
| resendCode | weqo2wy3...on.aws | POST | None | {email} |
| onboardUser | nbye6mq4...on.aws | POST | **Required JWT (RS256)** | {sessionId} + Authorization header |
| chatAthenaBot | fuuyi3tn...on.aws | POST | Optional JWT | {question, username, workspace, ...} |
| chatJobQueue | msby2wga...on.aws | POST | Optional JWT | {query, user_id, session_id, options} |
| chatJobStatus | d3fjrowu...on.aws | GET | Optional JWT | ?job_id=xxx |
| getChatHistory | nsdsmanm...on.aws | POST | Required JWT | {username, workspace, page_num, session_id} |
| uploadDeepDocument | rpajxt2r...on.aws | POST | Optional JWT | Binary (base64), ?sessionId&filename |
| getDeepDocuments | a7ezbunp...on.aws | GET | Required JWT | Authorization header (JWT extraction) |
| getDeepInsights | yocplzdg...on.aws | GET | **Required JWT (RS256)** | ?name&query |
| getDocumentInsights | a3cr3h3f...on.aws | GET | **Required JWT (RS256)** | ?username&workspace&documentId |
| deepResearch | xcw7giwp...on.aws | POST | Optional JWT | {question, username, workspace, ...} |
| webSearch | szwe24pa...on.aws | POST | Internal | {query, max_results} |
| auditTrailStore | yhk3dw2d...on.aws | POST | **Required JWT (ext) / None (internal)** | {operation, lambda_name, user_id, ...} |
| auditTrailVerify | 5lixjvl6...on.aws | POST | **Required JWT (ext) / None (internal)** | {audit_id} OR {start_date, end_date} |
| consentManager | etpphotej...on.aws | POST | Required JWT | {action, user_id, consent_type, ...} |
| complianceEngine | odr7ton4...on.aws | POST | Required JWT | {action, params} |
| traceExplainer | fvkvuah3...on.aws | POST | Optional | {query, answer, rag_chunks, kg_data} |
| reasoner | fiz2ibjc...on.aws | POST | Required JWT | {username, workspace, document_ids} |
| responseKG | n6s2blnj...on.aws | POST | Optional JWT | {answer, question, rag_chunks, ...} |
| saveArchetype | 6joydzjf...on.aws | POST | Required JWT | {userId, ...archetype fields} |
| getArchetype | 6lj7orpu...on.aws | GET | Required JWT | ?userId=xxx |
| createSession | m3n757es...on.aws | POST | Required JWT | {workspaceKey, sessionName} |
| listSessions | xqzifgqi...on.aws | GET | Required JWT | ?workspaceKey=xxx |
| createProject | iinfihpz...on.aws | POST | Optional JWT | {name, fullName, site} |
| campaigns | 4me7u72k...on.aws | GET | Optional JWT | (no body) |
| workspaces | API Gateway /workspaces | GET | Optional JWT | (API Gateway only, no dedicated Lambda) |
| subscription | nx4pgkogt...on.aws | GET | Optional JWT | (no body, JWT extraction) |
| queryUsage | emw6oett...on.aws | GET/POST | Optional JWT | GET: ?username=xxx / POST: {username} |

---

## 3. API GATEWAY ROUTES (25 routes, API ID: 1v186le2ee)

### JWT-Protected Routes (5):
- POST /onboard → eu_onboard_user
- POST /create-project → eu_create_project
- POST /get-documents → eu_get_deep_documents
- GET /get-documents → eu_get_deep_documents
- POST /chat-history → eu_get_chat_history

### NONE Auth Routes (20):
- All remaining routes have AuthorizationType: NONE

### Potentially Broken:
- POST /chat → integration fk0y5gi NOT found in integrations list

### Orphaned Integrations:
- eu_audit_trail_verify (1bysa92) — no visible route
- eu_trace_explainer (29g00c2) — no visible route
- eu_get_deep_insights — 3 integrations, only 1 route
- eu_list_sessions — 2 integrations, no visible route
- eu_deep_research — 2 integrations, only 1 route

---

## 4. HANDLER PATTERNS

### Pattern A: normalize_event + build_function_url_response (20 handlers)
Standard pattern. Event parsed by shared/lambda_utils.py. Response formatted with CORS headers.

### Pattern B: Direct Response (8 handlers)
Legacy handlers (eu_create_session, eu_list_sessions, eu_get_user_archetype, eu_save_user_archetype, eu_chat_job_queue, eu_chat_job_status, eu_chat_job_worker, eu_register). Manual event parsing, manual response construction.

### Pattern C: Async Self-Invoke (eu_deep_research)
Creates DynamoDB job, self-invokes with __async_worker flag, returns 202.

### Pattern D: Fire-and-Forget (eu_conversation_memory, eu_response_kg_extractor)
Invoked with InvocationType="Event" by eu_chat_athena_bot.

### Pattern E: SQS Worker (eu_chat_job_worker)
Triggered by SQS queue, processes through 6-stage pipeline.

### Pattern F: EventBridge Cron (eu_friction_event_batcher)
Scheduled every 5 minutes, batches friction events.

---

## 5. AMPLIFY ENVIRONMENT VARIABLES

### Branch-Level (main): 15 vars
| Variable | Value | Used By Code? |
|----------|-------|:---:|
| NEXT_PUBLIC_REGION | eu | YES |
| NEXT_PUBLIC_EU_API_BASE | https://1v186le2ee... | YES |
| NEXT_PUBLIC_EU_USER_POOL_ID | eu-central-1_Z0rehiDtA | YES |
| NEXT_PUBLIC_EU_USER_POOL_CLIENT_ID | 7d4487490ur1tpai0fuh4qle0b | YES |
| DYNAMO_REGION | eu-central-1 | YES |
| DYNAMO_ACCESS_KEY_ID | ***MASKED*** | YES |
| DYNAMO_SECRET_ACCESS_KEY | ***MASKED*** | YES |
| CANVAS_MOCK_LLM | false | YES |
| CHAT_ATHENA_EU_URL | https://1v186le2ee.../chat | **NO (orphan)** |
| NEXT_PUBLIC_CHAT_ATHENA_EU_URL | https://1v186le2ee.../chat | **NO (orphan)** |
| NEXT_PUBLIC_ENABLE_CANVAS | true | **NO (orphan)** |
| NEXT_PUBLIC_ENABLE_CANVAS_3D | false | **NO (orphan)** |
| NEXT_PUBLIC_ENABLE_CANVAS_CHAINING | true | **NO (orphan)** |
| NEXT_PUBLIC_ENABLE_CANVAS_COLLAB | false | **NO (orphan)** |
| NEXT_PUBLIC_EU_UPLOAD_URL | rbbtum33... (STALE) | **NO (orphan)** |

### Missing from Amplify (code reads but not set):
- NEXT_PUBLIC_EU_CHAT_QUEUE_URL (falls back to hardcoded URL)
- NEXT_PUBLIC_EU_CHAT_STATUS_URL (falls back to hardcoded URL)
- AWS_ACCESS_KEY_ID (friction emit route fails)
- AWS_SECRET_ACCESS_KEY (friction emit route fails)

---

## 6. CROSS-CUTTING ISSUES

### Issue 1: Auth Token Format Inconsistency
- Most endpoints: `Authorization: token` (raw JWT)
- Session endpoints: `Authorization: Bearer ${token}` (Bearer prefix)
- Backend handlers handle BOTH formats via normalize_event JWT extraction

### Issue 2: Lambda Proxy Envelope
- Some handlers return `{statusCode, body: "stringified JSON"}`
- Others return `{statusCode, data: {...}}`
- Frontend unwraps with: `if (typeof data.body === "string") data = JSON.parse(data.body)`

### Issue 3: statusCode Type Mismatch
- Some handlers return statusCode as number (200)
- Some return as string ("200")
- Frontend checks both: `statusCode !== HTTP_STATUS[200]` and `statusCode !== HTTP_STATUS.OK`

### Issue 4: conftest.py Stale Env Vars
- `CREATE_PROJECT_FUNCTION=eu_create_project_proxy` → should be `eu_create_project`
- `ATHENA_TRAINING_FUNCTION=eu_test_semantic` → should be removed entirely

---

## 7. TEST INFRASTRUCTURE SUMMARY

| Layer | Framework | Files | Cases | Status |
|-------|-----------|-------|-------|--------|
| Frontend Unit | Jest | 1 | ~10 | INACTIVE (commented out) |
| Frontend E2E | Playwright | 2 | ~30 | ACTIVE |
| Visual Regression | Pallas | 6 suites | 75+ | ACTIVE (10 personas × 3 viewports) |
| Backend Unit | pytest | 26 | ~100+ | ACTIVE |
| Backend E2E | pytest | 4 | ~20 | ACTIVE (FrictionMelt) |
| Backend Compliance | pytest | 4 | ~30 | ACTIVE (GDPR, EU AI Act) |
| CI/CD | None | 0 | 0 | NOT CONFIGURED |

### How to Run:
```bash
# Frontend E2E (production)
E2E_LIVE=1 npx playwright test

# Visual regression
npm run visual-audit

# Backend all tests
cd crawlq-athena-eu-backend/SemanticGraphEU && pytest tests/ -v

# Backend compliance only
pytest tests/ -m compliance
```

---

## 8. LAMBDA INTER-INVOCATION MAP

```
eu_upload_deep_document
  └──(Event)→ eu_deep_graph_builder
  └──(Event)→ eu_generate_deep_insights

eu_chat_athena_bot
  └──(Sync)→ eu_web_search
  └──(Sync)→ eu_get_deep_insights (session KG)
  └──(Event)→ eu_conversation_memory
  └──(Event)→ eu_response_kg_extractor

eu_deep_research
  └──(Self-invoke)→ eu_deep_research (async worker)
  └──(Sync)→ eu_web_search
  └──(Sync)→ eu_deep_graph_builder

eu_onboard_user
  └──(Sync)→ eu_create_project
  └──(Sync)→ eu_deep_graph_builder

eu_chat_job_queue → SQS → eu_chat_job_worker
  └──(pipeline)→ eu_get_deep_insights, eu_deep_graph_builder, Bedrock

eu_process_deep_document
  └──(Sync)→ eu_deep_graph_builder
```

---

---

## 9. DEPLOYMENT LOG (2026-02-16)

### Lambda Deployments (via `deploy_eu_lambdas.py` — ADR-043)
- **Method:** Download-Overlay-Upload (preserves bundled deps)
- **18/19 deployed** to eu-central-1
- **1 missing:** `eu_deep_reasoner` — Lambda function not yet created in AWS
- **Script:** `crawlq-athena-eu-backend/SemanticGraphEU/deploy_eu_lambdas.py`

### Key Code Changes Deployed
| Component | Change | Commit |
|-----------|--------|--------|
| `shared/jwt_auth.py` | Centralized RS256 JWT verification (Cognito JWKS) | Phase 1 |
| `shared/document_extractor.py` | PyMuPDF + full DOCX parsing (headers, footnotes, lists) | 5cb078e |
| `shared/cryptographic_delete.py` | GDPR Art. 17 cascade (8 tables + S3 + Neo4j) | 8f38c90 |
| `shared/data_export.py` | GDPR Art. 20 portability (canvas_documents added) | 8f38c90 |
| `EUGetDeepInsights/handler.py` | `require_auth()` — returns 401 | Phase 1 |
| `EUGetDocumentInsights/handler.py` | `require_auth()` — returns 401 | Phase 1 |
| `EUOnboardUser/handler.py` | `require_auth()` — replaces crash | f14543c |
| `EUAuditTrailStore/handler.py` | JWT auth gate for Function URL calls | f14543c |
| `EUAuditTrailVerify/handler.py` | JWT auth gate for Function URL calls | f14543c |
| `EUChatAthenaBot/handler.py` | Pre-flight compliance + consent checks | Phase 4 |
| `EUResponseKGExtractor/handler.py` | Neo4j write wired | Phase 3 |
| `EUConversationMemory/handler.py` | session_id added | Phase 3 |
| Frontend payload fixes | `documentId`→`query`, `workspaceName`→`name` | 15f294c |
| Canvas feature gating | Server-side subscription + client-side gate | 56260fd |

### Frontend (Amplify)
- **App ID:** d45bl3mgpjnhy
- **Latest commit:** 15f294c (payload sync fixes)
- **Build status:** SUCCEED

---

## 10. SHARED MODULE INVENTORY

All 19 core Lambdas include these shared modules (updated via overlay deploy):

| Module | Purpose | Size |
|--------|---------|------|
| `shared/eu_config.py` | Region-specific configuration (DynamoDB tables, Neo4j, Bedrock models) | 4KB |
| `shared/lambda_utils.py` | `normalize_event()`, `build_function_url_response()` | 3KB |
| `shared/jwt_auth.py` | `require_auth()`, `optional_auth()`, RS256 JWKS verification | 6KB |
| `shared/audit_trail.py` | `log_request()`, `log_response()` | 3KB |
| `shared/compliance_metadata.py` | `enrich_response()` with compliance metadata | 2KB |
| `shared/document_extractor.py` | Multi-format extractor (PDF, DOCX, XLSX, PPTX, etc.) | 20KB |
| `shared/cryptographic_delete.py` | GDPR Art. 17 cascade delete | 5KB |
| `shared/data_export.py` | GDPR Art. 20 data portability export | 4KB |
| `_jwt_lib/` | Bundled PyJWT 2.11.0 with JWKS support | 21 files |

*This document is a living manifest. Update when Lambda deployments, env vars, or endpoint wiring changes.*
