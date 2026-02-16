# CrawlQ Athena EU — End-to-End Workflow Audit Report

**Date:** 2026-02-16
**Scope:** 36 Lambda functions, 4 repos, full pipeline audit
**Status:** SYSTEM NOT PRODUCTION-READY — 25 critical/high issues across 4 workflows

---

## EXECUTIVE SUMMARY

Four parallel audits were conducted across the entire CrawlQ Athena EU system:

| Audit | Status | Critical | High | Medium |
|-------|--------|----------|------|--------|
| 1. Document Upload → KG → RAG | 60% ready | 6 | 3 | 9 |
| 2. Chat → Memory → KG Feedback | 70% ready | 4 | 3 | 5 |
| 3. Canvas System & Feature Gating | 80% ready | 2 | 3 | 3 |
| 4. TRACE Compliance & Audit Trail | 60% ready | 5 | 8 | 12 |
| **TOTAL** | | **17** | **17** | **29** |

### Top 5 Blocking Issues (Must Fix Before Launch)

1. **ZERO AUTH on document retrieval endpoints** — Any attacker can read any user's insights
2. **Hardcoded Neo4j credentials in source code** — `CrawlQ-EU-2026!` exposed
3. **KG feedback loop broken** — Extracted knowledge never persisted back to Neo4j
4. **TRACE compliance not enforced** — Checks exist but are never called pre-flight
5. **Canvas feature gating not enforced** — Any user bypasses subscription limits

---

## CONSOLIDATED FINDINGS BY SEVERITY

### CRITICAL (17 issues — Must fix before ANY user access)

#### Security (6)
| ID | Finding | Component | Impact |
|----|---------|-----------|--------|
| SEC-1 | JWT signature NOT verified (`verify_signature: False`) | EUUploadDeepDocument | User impersonation |
| SEC-2 | ZERO auth on GET_DEEP_INSIGHTS endpoint | EUGetDeepInsights | Full information disclosure |
| SEC-3 | ZERO auth on GET_DOCUMENT_INSIGHTS endpoint | EUGetDocumentInsights | Document enumeration |
| SEC-4 | Hardcoded Neo4j credentials in source (`CrawlQ-EU-2026!`) | shared/eu_config.py | DB compromise |
| SEC-5 | Canvas execute-llm endpoint has NO auth | /api/canvas/execute-llm | Cost exposure, DoS |
| SEC-6 | DynamoDB credentials in `.env.local` | Frontend repo | Credential leak |

#### Data Loss (4)
| ID | Finding | Component | Impact |
|----|---------|-----------|--------|
| DL-1 | Fire-and-forget async invocations — no DLQ, no retry | EUUploadDeepDocument | Silent doc processing failure |
| DL-2 | DynamoDB write-then-read race condition | Upload → GraphBuilder | Intermittent processing failure |
| DL-3 | KG extraction not persisted to Neo4j | EUResponseKGExtractor | Feedback loop broken |
| DL-4 | Conversation memory not isolated by session | EUConversationMemory | Context contamination |

#### Compliance (5)
| ID | Finding | Component | Impact |
|----|---------|-----------|--------|
| CMP-1 | Compliance checks exist but NEVER called | EUComplianceEngine | Art. 9 risk not assessed |
| CMP-2 | Consent recorded but NEVER checked before ops | EUConsentManager | GDPR Art. 7 violation |
| CMP-3 | Human review routing is "soft" — doesn't block | EUChatAthenaBot | Art. 14 not enforced |
| CMP-4 | Audit trail verification — NO API route (orphaned) | EUAuditTrailVerify | Art. 51 unverifiable |
| CMP-5 | TRACE explainer — NO API route (orphaned) | EUTraceExplainer | Art. 13 unreachable |

#### Feature Gating (2)
| ID | Finding | Component | Impact |
|----|---------|-----------|--------|
| GATE-1 | Canvas feature gates defined but NEVER enforced | Canvas routes | Subscription bypass |
| GATE-2 | Canvas API routes have NO subscription validation | /api/canvas/* | Unlimited usage |

### HIGH (17 issues — Must fix before first real users)

| ID | Finding | Category |
|----|---------|----------|
| H-1 | GET_DEEP_DOCUMENTS handler missing (Lambda not found) | Infrastructure |
| H-2 | Anthropic API key init failure not handled (silent fallback) | Reliability |
| H-3 | Async job worker uses placeholder RAG/KG (TODO comments) | Functionality |
| H-4 | No retry logic for fire-and-forget Lambda invocations | Reliability |
| H-5 | Missing audit trail integration in document pipeline | Compliance |
| H-6 | Non-deterministic hash retrieval (scan without sort) | Data Integrity |
| H-7 | Risk levels are advisory, not enforceable | Compliance |
| H-8 | Compliance passport always returns "COMPLIANT" | Compliance |
| H-9 | Implicit consent model (opt-out, should be opt-in) | GDPR |
| H-10 | Art. 17 erasure incomplete (no cascade delete) | GDPR |
| H-11 | TRACE scores computed but not persisted | Auditability |
| H-12 | Compliance score always 0.80-1.0 (hardcoded factors) | Compliance |
| H-13 | ChatTraceCard no error boundary for missing data | Frontend |
| H-14 | Chat→Canvas bridge incomplete (navigation only, no data) | Feature |
| H-15 | execute-llm has no auth + no rate limiting | Security |
| H-16 | Model aliasing misleads users (GPT-4o → actually Claude) | UX |
| H-17 | Original Canvas repo status unclear (archive or maintain?) | DevOps |

### MEDIUM (29 issues — Fix within 1 month)

Key items:
- N+1 Bedrock API calls per workspace query (4-10s latency, $12K/mo at scale)
- Unsupported file formats fail silently (XLS, PPT, DOC)
- Tesseract OCR not in Lambda runtime (images fail silently)
- Guardrails are warnings, not blockers
- No consent expiry/renewal mechanism
- No scheduled audit verification
- No TRACE score anomaly detection
- Web search results not validated against document sources
- Offset pagination slow on large chat histories
- Orphaned Amplify environment variables (6 stale vars)
- Frontend polling mechanism missing for async insight generation

---

## PRIORITIZED ACTION PLAN

### PHASE 1: SECURITY HARDENING (Days 1-3)

**Goal:** Close all CRITICAL security vulnerabilities

| # | Action | Owner | Hours | Blocks |
|---|--------|-------|-------|--------|
| 1.1 | Add JWT verification to GET_DEEP_INSIGHTS + GET_DOCUMENT_INSIGHTS | Backend | 4h | SEC-2, SEC-3 |
| 1.2 | Fix JWT signature verification in upload handler | Backend | 2h | SEC-1 |
| 1.3 | Move Neo4j credentials to AWS Secrets Manager | Backend | 2h | SEC-4 |
| 1.4 | Add requireAuth() to Canvas execute-llm route | Frontend | 2h | SEC-5 |
| 1.5 | Rotate DynamoDB credentials, remove from .env.local | DevOps | 1h | SEC-6 |
| 1.6 | Rotate Neo4j password after migration | DevOps | 1h | SEC-4 |

**Total: ~12 hours**

### PHASE 2: DATA INTEGRITY (Days 3-7)

**Goal:** Fix data loss risks and broken feedback loops

| # | Action | Owner | Hours | Blocks |
|---|--------|-------|-------|--------|
| 2.1 | Add SQS DLQ for graph builder + insights generator async invocations | Backend | 4h | DL-1 |
| 2.2 | Add DynamoDB ConsistentRead=True in graph builder | Backend | 2h | DL-2 |
| 2.3 | Persist KG extraction to Neo4j (close feedback loop) | Backend | 24h | DL-3 |
| 2.4 | Add session_id to memory DynamoDB key schema | Backend | 12h | DL-4 |
| 2.5 | Add retry logic via SQS for memory + KG extraction | Backend | 16h | H-4 |
| 2.6 | Complete async job worker — replace placeholder RAG/KG | Backend | 16h | H-3 |

**Total: ~74 hours (2 weeks with parallel work)**

### PHASE 3: COMPLIANCE ENFORCEMENT (Days 7-10)

**Goal:** Make compliance checks enforceable, not advisory

| # | Action | Owner | Hours | Blocks |
|---|--------|-------|-------|--------|
| 3.1 | Add pre-flight compliance check in EUChatAthenaBot | Backend | 8h | CMP-1 |
| 3.2 | Add consent check before all data processing operations | Backend | 8h | CMP-2 |
| 3.3 | Block low-confidence responses (< 0.50) pending review | Backend | 6h | CMP-3 |
| 3.4 | Create API Gateway route for EUAuditTrailVerify | Backend | 4h | CMP-4 |
| 3.5 | Create API Gateway route for EUTraceExplainer | Backend | 4h | CMP-5 |
| 3.6 | Fix hash chain determinism (Query instead of Scan) | Backend | 4h | H-6 |
| 3.7 | Fix compliance passport to reflect actual status | Backend | 4h | H-8 |
| 3.8 | Switch consent model from opt-out to opt-in | Backend | 6h | H-9 |

**Total: ~44 hours**

### PHASE 4: FEATURE GATING (Days 10-12)

**Goal:** Enforce subscription tier limits on Canvas

| # | Action | Owner | Hours | Blocks |
|---|--------|-------|-------|--------|
| 4.1 | Add useEUFeatureGate() checks to Canvas layout + routes | Frontend | 4h | GATE-1 |
| 4.2 | Add subscription validation to Canvas save/execute API routes | Frontend | 6h | GATE-2 |
| 4.3 | Implement canvas count + daily run counter in DynamoDB | Backend | 8h | GATE-2 |
| 4.4 | Add Canvas limits to subscription Lambda response | Backend | 4h | H-16 |
| 4.5 | Remove or fix model aliasing (GPT-4o → Claude fallback) | Frontend | 2h | H-16 |

**Total: ~24 hours**

### PHASE 5: QUALITY & COMPLETENESS (Days 12-20)

**Goal:** Fix high-priority gaps, improve reliability

| # | Action | Owner | Hours | Blocks |
|---|--------|-------|-------|--------|
| 5.1 | Deploy missing GET_DEEP_DOCUMENTS Lambda | Backend | 4h | H-1 |
| 5.2 | Persist TRACE scores in DynamoDB | Backend | 6h | H-11 |
| 5.3 | Fix compliance score computation (dynamic, not hardcoded) | Backend | 4h | H-12 |
| 5.4 | Implement cascade delete for GDPR Art. 17 | Backend | 8h | H-10 |
| 5.5 | Add Chat→Canvas data bridge (parse sourceDocumentId) | Frontend | 8h | H-14 |
| 5.6 | Implement scheduled audit verification (nightly EventBridge) | DevOps | 6h | Medium |
| 5.7 | Add consent-audit linking | Backend | 4h | Medium |
| 5.8 | Add error boundaries to TRACE frontend components | Frontend | 4h | H-13 |
| 5.9 | Archive original crawlq-athena-eu-canvas repo | DevOps | 2h | H-17 |
| 5.10 | Clean up 6 orphaned Amplify environment variables | DevOps | 1h | Medium |

**Total: ~47 hours**

### PHASE 6: PERFORMANCE & POLISH (Days 20-30)

**Goal:** Optimize for production scale

| # | Action | Owner | Hours |
|---|--------|-------|-------|
| 6.1 | Cache entity extraction in GET_DEEP_INSIGHTS (reduce 4 Bedrock calls to 2) | Backend | 8h |
| 6.2 | Add support for XLS/XLSX/PPT/PPTX extraction | Backend | 8h |
| 6.3 | Add Lambda Layer with Tesseract for OCR | Backend | 6h |
| 6.4 | Change guardrails from warnings to blockers | Backend | 4h |
| 6.5 | Add consent expiry + renewal mechanism | Backend | 4h |
| 6.6 | Add frontend polling for async insight generation | Frontend | 6h |
| 6.7 | Replace offset pagination with cursor-based | Backend | 6h |
| 6.8 | Add TRACE score anomaly detection | Backend | 8h |
| 6.9 | Build compliance dashboard for auditors | Frontend | 16h |
| 6.10 | Build human review queue dashboard | Frontend | 16h |

**Total: ~82 hours**

---

## TIMELINE SUMMARY

| Phase | Focus | Days | Est. Hours |
|-------|-------|------|------------|
| Phase 1 | Security Hardening | 1-3 | 12h |
| Phase 2 | Data Integrity | 3-7 | 74h |
| Phase 3 | Compliance Enforcement | 7-10 | 44h |
| Phase 4 | Feature Gating | 10-12 | 24h |
| Phase 5 | Quality & Completeness | 12-20 | 47h |
| Phase 6 | Performance & Polish | 20-30 | 82h |
| **TOTAL** | | **30 days** | **~283 hours** |

**MVP Launch Gate (Phases 1-4):** ~154 hours / ~12 working days
**Full Production Ready (All Phases):** ~283 hours / ~30 working days

---

## CROSS-CUTTING ARCHITECTURE ISSUES

### 1. Auth Token Format Inconsistency
- Some endpoints: `Authorization: token` (raw JWT)
- Some endpoints: `Authorization: Bearer ${token}` (Bearer prefix)
- Backend handles BOTH via normalize_event but should standardize

### 2. Lambda Response Envelope Inconsistency
- Some: `{statusCode, body: "stringified JSON"}`
- Some: `{statusCode, data: {...}}`
- Frontend unwraps with: `if (typeof data.body === "string") data = JSON.parse(data.body)`

### 3. statusCode Type Mismatch
- Some handlers return statusCode as number (200)
- Some return as string ("200")

### 4. conftest.py Stale Env Vars
- `CREATE_PROJECT_FUNCTION=eu_create_project_proxy` → should be `eu_create_project`
- `ATHENA_TRAINING_FUNCTION=eu_test_semantic` → should be removed

### 5. API Gateway Orphaned Routes
- POST /chat → integration fk0y5gi NOT found
- eu_audit_trail_verify — no visible route
- eu_trace_explainer — no visible route
- eu_list_sessions — 2 integrations, no visible route

---

## DEPLOYMENT VERIFICATION CHECKLIST

Before declaring ANY fix deployed:
1. Amplify build SUCCEED status verified
2. Wait 5 min for CDN propagation
3. Incognito browser verification
4. Lambda smoke test (200 OK)
5. Run Crucible/Pallas tests
6. Verify no `x-cache: Error from cloudfront`

---

*This document combines the EU-System-E2E-Inventory with 4 parallel workflow audits.
Update as fixes are deployed.*
