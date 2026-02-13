# ADR-025: Production Testing Strategy — Athena EU
**Date:** 2026-02-12 | **Status:** ACCEPTED

**Context:** Athena EU has 24 Lambda functions, 7 DynamoDB tables, an API Gateway, and a Next.js 14 frontend with 45+ components. We need a repeatable, cost-aware testing strategy that provides production confidence without burning excessive Bedrock/API credits.

**Decision:** Implement a 5-layer testing pyramid executed via Python scripts (no Playwright overhead for API tests). Each layer has a pass/fail gate.

---

## Layer 1: Infrastructure Health (Cost: $0)
**Tool:** AWS CLI / boto3 | **Time:** ~30s | **Run:** Every deploy

| Test | Command | Pass Criteria |
|------|---------|---------------|
| Lambda inventory | `aws lambda list-functions --region eu-central-1` | 24 functions Active |
| DynamoDB tables | `aws dynamodb list-tables --region eu-central-1` | 7 tables ACTIVE |
| API Gateway routes | `aws apigatewayv2 get-routes` | All routes configured |
| Cognito pool | `aws cognito-idp describe-user-pool` | Pool Active |

**Script:** `scripts/test-infrastructure.py`

---

## Layer 2: Backend Chain Tests (Cost: ~$0.02 per run)
**Tool:** Python requests + boto3 | **Time:** ~60s | **Run:** Every backend deploy

| Test ID | Endpoint | Method | Payload | Pass Criteria |
|---------|----------|--------|---------|---------------|
| CHAT-01 | `/chat` | POST | `{mode: "chat", question: "ping"}` | 200, `answer` present |
| CHAT-02 | `/chat` | POST | `{mode: "trace", question: "GDPR test"}` | 200, `trace_dimensions` present |
| CHAT-03 | `/chat` | POST | `{mode: "web_search", question: "EU AI Act"}` | 200, `web_sources` array |
| CHAT-04 | `/chat` | POST | `{mode: "combined"}` | 200 or 503 (async fallback) |
| HIST-01 | `/chat-history` | GET | `?username=test&workspace=default` | 200, `pairs` array |
| REG-01 | `/register` | POST | `{email: "test@invalid"}` | 400 (expected rejection) |
| DOC-01 | `/upload` | POST | multipart file | 200, `session_id` |
| DOC-02 | `/get-documents` | GET | `?username=test` | 200, array response |
| KG-01 | KG Lambda invoke | boto3 | `{answer: "test", question: "q"}` | 200, `nodes` + `relationships` |
| GOV-01 | Circuit breaker | DynamoDB | write/read/delete | CRUD success |

**Script:** `scripts/test-backend-chain.py`

---

## Layer 3: Frontend Build + Type Safety (Cost: $0)
**Tool:** `npx next build` + `npx tsc --noEmit` | **Time:** ~45s | **Run:** Every frontend deploy

| Test | Command | Pass Criteria |
|------|---------|---------------|
| Production build | `npx next build` | Exit 0, zero errors |
| Type checking | `npx tsc --noEmit` | Exit 0 (if tsconfig strict) |
| Page count | Check build output | 7 pages compiled |
| Bundle size | Check `chat-athena-eu` size | < 500 kB first load JS |

**Script:** `scripts/test-frontend-build.py`

---

## Layer 4: E2E Functional Tests (Cost: ~$0.05 per run)
**Tool:** Python requests (API) + Playwright (UI) | **Time:** ~120s | **Run:** Pre-release

| Test ID | Flow | Steps | Pass Criteria |
|---------|------|-------|---------------|
| E2E-01 | Chat → TRACE | Send question → verify TRACE card | Response has `trace_dimensions` + `governance` |
| E2E-02 | Chat → Export | Send question → export as MD | File downloaded, contains TRACE metadata |
| E2E-03 | Chat → KG | Send question → click Explore → verify nodes | KG returns `nodes` + `relationships` |
| E2E-04 | Chat → Web Search | Send web_search question → verify sources | Response has `web_sources` array |
| E2E-05 | Guest → Upload → Insights | Upload PDF as guest → poll insights | Document processed, insights returned |
| E2E-06 | Auth → Chat History | Login → send message → fetch history | History contains new pair |
| E2E-07 | Governance | Send borderline query → verify gate | Governance returns `allow`/`warn`/`deny` |
| E2E-08 | Circuit breaker | Simulate failures → verify state | Breaker transitions CLOSED→OPEN→HALF_OPEN |

**Script:** `scripts/test-e2e-functional.py`

---

## Layer 5: Visual UI Audit (Cost: ~$0.50 per run — Bedrock Vision)
**Tool:** Playwright + Claude Vision (via `scripts/visual-audit.mjs`) | **Time:** ~90s | **Run:** Pre-release

| Viewport | Resolution | Checks |
|----------|-----------|--------|
| Mobile | 390x844 | Touch targets ≥44px, no horizontal overflow, sidebar collapsed |
| Tablet | 768x1024 | Sidebar visible, panels stack correctly |
| Desktop | 1440x900 | Full layout, artifact panel opens, command palette renders |

**Script:** `node scripts/visual-audit.mjs` (existing GCC tool)

---

## Test Execution Order

```
Deploy → Layer 1 (Infra) → Layer 2 (Backend) → Layer 3 (Build) → Layer 4 (E2E) → Layer 5 (Visual)
         ~30s                ~60s                ~45s              ~120s             ~90s
         $0                  ~$0.02              $0                ~$0.05            ~$0.50
```

**Total cost per full test run:** ~$0.57
**Total time:** ~6 minutes

## Cost Controls
- Layer 2 uses minimal payloads ("ping", "GDPR test") — avoids long Bedrock responses
- KG test uses boto3 invoke (bypass Function URL 403 issue from local)
- Combined mode test accepts 503 as valid (async fallback)
- Visual audit uses `--skip-vision` flag for metrics-only runs ($0 cost)
- Never test with production user data — always use `test@crawlq.eu` + `default` workspace

## Pass/Fail Gates
- **Layer 1:** ALL must pass → proceed
- **Layer 2:** ≥9/10 pass → proceed (combined 503 is acceptable)
- **Layer 3:** ALL must pass → proceed
- **Layer 4:** ≥7/8 pass → proceed (1 allowed degraded)
- **Layer 5:** No critical issues at any viewport → ship

## When to Run
- **Full suite (L1-L5):** Before every Amplify deployment, after major feature branches merge
- **Quick suite (L1-L3):** After any code push
- **Regression (L2+L4):** After backend Lambda updates
