# ADR-011: Lambda Function URLs with Buffered Response for Timeout-Sensitive Endpoints
**Date:** 2026-02-11 | **Status:** ACCEPTED
**Supersedes:** ADR-010 (partially — chat endpoints only)
**Updated:** 2026-02-11 — Changed from RESPONSE_STREAM to BUFFERED (simpler implementation)

**Context:**
API Gateway HTTP API has a hard 30-second timeout limit that cannot be increased. Long AI responses (complex GDPR questions, multi-document analysis) regularly take 30-60+ seconds, causing 503 Service Unavailable errors. Current "streaming" is simulated — Lambda buffers full response in memory, API Gateway buffers entire Lambda response, then frontend progressively renders pre-collected chunks. Real-world user testing shows long answers timing out despite streaming=true.

**Decision:**
Use **Lambda Function URLs with Response Streaming (InvokeMode: RESPONSE_STREAM)** for timeout-sensitive endpoints. Keep API Gateway for fast endpoints.

**Architecture:**
```
Timeout-Sensitive (Function URL direct):
  Browser → Lambda Function URL (AuthType: NONE, InvokeMode: RESPONSE_STREAM) → Bedrock streaming

Fast Endpoints (API Gateway):
  Browser → API Gateway → Lambda (AuthType: AWS_IAM)
```

**Lambda Classification:**

| Lambda | Route | Timeout Risk | Solution |
|--------|-------|--------------|----------|
| `eu_chat_athena_bot` | — | **HIGH (30-300s)** | **Function URL (RESPONSE_STREAM)** |
| `eu_deep_graph_builder` | — | HIGH (60s+) | **Function URL** |
| `eu_generate_deep_insights` | — | HIGH (60s+) | **Function URL** |
| `eu_reasoner` | — | MEDIUM (30-60s) | **Function URL** |
| `eu_process_deep_document` | — | MEDIUM (30-60s) | **Function URL** |
| `eu_register` | POST /register | LOW (<2s) | API Gateway |
| `eu_confirm_signup` | POST /confirm | LOW (<2s) | API Gateway |
| `eu_resend_code` | POST /resend-code | LOW (<2s) | API Gateway |
| `eu_upload_deep_document` | POST /upload | LOW (<2s) | API Gateway |
| `eu_get_chat_history` | POST /chat-history | LOW (<2s) | API Gateway |
| `eu_get_deep_documents` | POST /get-documents | LOW (<2s) | API Gateway |
| `eu_onboard_user` | POST /onboard | LOW (<2s) | API Gateway |
| `eu_audit_trail_store` | — | LOW (<2s) | Internal only |
| `eu_audit_trail_verify` | — | LOW (<2s) | Internal only |
| `eu_compliance_engine` | — | LOW (<5s) | Internal only |
| `eu_consent_manager` | — | LOW (<2s) | Internal only |
| `eu_trace_explainer` | — | LOW (<5s) | Internal only |

**Implementation:**
1. Update `eu_chat_athena_bot` Lambda Function URL: AuthType → NONE, InvokeMode → RESPONSE_STREAM
2. Update handler.py to use awslambdaric response streaming API (write chunks to pipe, not buffered JSON)
3. Update frontend `region-config.ts` to use Function URL directly for chat (bypass API Gateway)
4. Update `useEUStreamingMessage.ts` to consume real NDJSON stream (not simulated progressive rendering)
5. Keep API Gateway routes for all other endpoints (no changes needed)

**Consequences:**
- (+) No timeout limit for chat (Lambda max = 15 min vs API Gateway 30s hard limit)
- (+) True real-time streaming (chunks arrive as Bedrock generates them, not buffered)
- (+) Cost reduction (no API Gateway charges for high-volume chat endpoint)
- (+) Memory efficiency (Lambda streams chunks instead of buffering entire response)
- (+) Better UX (users see tokens streaming in real-time, not simulated)
- (+) Consistent principle: Use right tool for right job (Function URL for long-running, API Gateway for fast operations)
- (-) JWT validation in Lambda adds ~20ms latency per chat request (acceptable trade-off)
- (-) Slightly more complex frontend streaming client (ReadableStream vs fetch().json())
- (-) Chat endpoint loses API Gateway throttling/WAF (can add Lambda reserved concurrency if needed)

**Why Hybrid Approach:**
- **Consistency where it matters:** All timeout-sensitive endpoints use Function URLs, all fast endpoints use API Gateway
- **Best of both worlds:** API Gateway provides throttling, logging, and JWT validation for fast routes; Function URLs provide unlimited execution time for AI operations
- **Cost optimization:** Only chat endpoint generates high volume; keeping auth/query endpoints on API Gateway maintains centralized logging and rate limiting
