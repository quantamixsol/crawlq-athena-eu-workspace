# ADR-011: API Gateway 30s Timeout Handling Strategy

**Date:** 2026-02-11
**Status:** ACCEPTED
**Context:** EU Chat Athena 503 errors on complex queries

---

## Problem

API Gateway HTTP API has a **30-second hard timeout** that cannot be increased. Complex queries to Claude Opus 4.6 can take 30-60 seconds, especially when:
- Processing long context (GDPR compliance documents, multi-part questions)
- Generating detailed responses with examples
- Building knowledge graphs alongside responses

**User Experience Impact:**
1. User sends complex question
2. API Gateway times out after 30s → returns 503
3. Lambda continues processing and saves result to DynamoDB
4. User must refresh page to see the completed response
5. This creates confusion and appears as a system failure

---

## Current State

**Backend:**
- Lambda: `eu_chat_athena_bot` (no timeout, can run up to 15 minutes)
- API Gateway: `/chat` route → 30s timeout (cannot be changed)
- Response time: 18.8s (GDPR Article 22 query) to 40s+ (complex multi-part queries)

**Frontend:**
- `useEUStreamingMessage` throws error on 503
- Error message: "Your question is being processed..."
- User must manually refresh to see result

---

## Decision

Implement a **3-tier progressive enhancement strategy**:

### Tier 1: Immediate Fix (Implemented in COMMIT 9)
- Improve error message to guide users
- Auto-invalidate chat history after 15s on 503
- Display "Processing..." indicator
- Update to "Refresh manually" if still no response

### Tier 2: Near-term Fix (Next Sprint)
**Use Lambda Function URLs with Response Streaming**

AWS Lambda now supports [response streaming](https://docs.aws.amazon.com/lambda/latest/dg/configuration-response-streaming.html) via Function URLs:
- No API Gateway involvement
- No 30s timeout (Lambda timeout only: 15 min)
- True streaming: chunks delivered progressively
- HTTP/1.1 chunked transfer encoding

**Implementation:**
```python
# Lambda handler with streaming
def lambda_handler(event, context):
    return awslambdaric.http.Response(
        status_code=200,
        headers={"Content-Type": "text/plain"},
        body=stream_generator()  # Generator yields chunks
    )
```

**Frontend changes:**
```typescript
// Read stream progressively
const response = await fetch(lambdaFunctionURL);
const reader = response.body.getReader();
while (true) {
    const {done, value} = await reader.read();
    if (done) break;
    // Update UI with chunk
}
```

### Tier 3: Long-term Architecture (Sprint 2-3)
**Hybrid: Fast Path + Long Path**

**Fast Path (<15s):**
- Use current API Gateway + Lambda
- Serve 80% of queries instantly

**Long Path (>15s):**
- Return immediately with `job_id`
- Lambda processes async, saves to DynamoDB
- Frontend polls `/chat-status/{job_id}` every 3s
- Display progress: "Processing... (estimated 30s)"
- Fetch result when `status: completed`

**Database schema:**
```yaml
DynamoDB: eu_chat_jobs
PK: job_id
SK: timestamp
Attributes:
  - status: pending | processing | completed | failed
  - progress: 0-100
  - result: {answer, metadata, trace}
  - ttl: 1 hour
```

---

## Consequences

### Tier 1 (Current)
✅ Minimal code changes
✅ Users understand what's happening
⚠️ Still requires manual refresh for slow queries
⚠️ Suboptimal UX

### Tier 2 (Recommended Next)
✅ True streaming, no timeout
✅ Progressive response rendering
✅ Better UX than polling
⚠️ Requires Lambda Function URL (not API Gateway)
⚠️ Loses API Gateway features (JWT authorizer, rate limiting, WAF)
**Mitigation:** Handle JWT validation in Lambda, add rate limiting via DynamoDB

### Tier 3 (Enterprise-grade)
✅ Optimal UX with progress indicators
✅ Scales to any response time
✅ Can show intermediate steps (RAG retrieval, graph building, reasoning)
⚠️ More complex architecture
⚠️ Requires job queue + status table
**Best fit:** When adding FrictionMelt integration (async decision pipeline)

---

## Related ADRs
- ADR-010: API Gateway HTTP API + Cognito JWT Authorizer
- ADR-008: Claude Opus 4.6 as Primary EU Bedrock Model

---

## References
- [AWS Lambda Response Streaming](https://docs.aws.amazon.com/lambda/latest/dg/configuration-response-streaming.html)
- [API Gateway Limits](https://docs.aws.amazon.com/apigateway/latest/developerguide/limits.html) — 30s timeout (HTTP API), 29s (REST API)
- Test: GDPR query took 28.5s (non-streaming), 18.8s (streaming), still approaching limit
