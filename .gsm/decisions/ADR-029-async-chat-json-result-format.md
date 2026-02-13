# ADR-029: Async Chat Result Format - JSON Wrapper for Markdown + Metadata

**Date:** 2026-02-13 | **Status:** ACCEPTED

## Context

The async chat worker Lambda (`EUChatJobWorker`) processes long-running chat requests (TRACE protocol, deep research) that exceed API Gateway's 30-second timeout. Previously, the Lambda was storing raw markdown responses directly to S3, but the frontend (`useEUAsyncChat.ts`) expects a structured JSON response containing not just the markdown answer, but also critical metadata including:

- **TRACE scores**: Transparency, Reasoning, Auditability, Compliance, Explainability dimensions
- **Confidence metrics**: Overall confidence score and confidence tier (HIGH/MEDIUM/LOW)
- **Token usage**: Input/output tokens for cost tracking and billing
- **Processing metadata**: Elapsed time, job ID, timestamps
- **Human review flags**: Whether the response requires human oversight (when confidence < 70%)

The mismatch caused the frontend to display raw markdown without access to TRACE scores or confidence indicators, breaking the governance and explainability features that are core to the EU compliance strategy.

## Problem

When the worker Lambda saved only markdown to S3:
```python
# OLD (incorrect):
s3.put_object(
    Bucket=bucket,
    Key=s3_key,
    Body=markdown.encode('utf-8'),  # Just raw markdown
    ContentType='text/markdown'
)
```

The frontend polling code expected:
```typescript
const result = await resultResponse.json();  // Expects JSON!

const finalMessage: IEUMessage = {
  content: result.answer,  // ✗ Fails - result is a string, not an object
  metadata: {
    confidence_score: result.confidence_score,  // ✗ Undefined
    trace_dimensions: result.trace_dimensions,   // ✗ Undefined
    // ...
  }
};
```

This caused:
1. **No TRACE scores displayed** - Users couldn't see confidence breakdown
2. **No human review flags** - Low-confidence responses weren't flagged for oversight
3. **No token usage tracking** - Cost monitoring broken
4. **Frontend errors** - TypeScript expecting object properties on a string

## Decision

**Wrap the markdown answer in a JSON structure that matches frontend expectations.**

The worker Lambda now builds a complete result object:

```python
# File: crawlq-lambda/SemanticGraphEU/EUChatJobWorker/handler.py (lines 102-122)

result_json = {
    'answer': markdown,  # The structured markdown content
    'model_used': 'claude-opus-4-6',
    'confidence_score': trace_scores.get('overall', 0.85),
    'confidence_tier': {
        'tier': 'HIGH' if trace_scores.get('overall', 0) >= 0.85 else 'MEDIUM',
        'threshold': 0.85
    },
    'metadata': {
        'input_tokens': options.get('input_tokens', 0),
        'output_tokens': len(raw_response.split()) * 1.3,  # Rough estimate
        'elapsed_seconds': round(processing_time, 1),
        'job_id': job_id,
        'user_id': job_data.get('user_id', 'guest'),
        'session_id': job_data.get('session_id', ''),
        'timestamp': datetime.utcnow().isoformat() + 'Z',
    },
    'trace_dimensions': trace_scores,  # Full T-R-A-C-E breakdown
    'human_review': trace_scores.get('overall', 0) < 0.70,
}

# Save as JSON
s3.put_object(
    Bucket=S3_BUCKET,
    Key=s3_key,
    Body=json.dumps(result_data, indent=2, ensure_ascii=False).encode('utf-8'),
    ContentType='application/json; charset=utf-8',
    Metadata={
        'job_id': job_id,
        'timestamp': datetime.utcnow().isoformat() + 'Z',
        'model': result_data.get('model_used', 'unknown'),
        'confidence': str(result_data.get('confidence_score', 0)),
    }
)
```

## TRACE Scores Computation

The Lambda computes TRACE dimensions by analyzing response characteristics (lines 327-391):

```python
def _compute_trace_scores(self, response: str, job_data: Dict) -> Dict:
    """
    T - Transparency: Source citations, data references (baseline: 0.40)
    R - Reasoning: Logical structure, because/therefore, numbered steps (baseline: 0.35)
    A - Auditability: Timestamps, IDs, tables, deterministic markers (baseline: 0.50)
    C - Compliance: GDPR/EU AI Act references, consent patterns (baseline: 0.25)
    E - Explainability: Plain language, examples, readability (baseline: 0.40)
    """
    # ... pattern matching logic ...

    overall = (
        0.25 * transparency +
        0.20 * reasoning +
        0.20 * auditability +
        0.15 * compliance +
        0.20 * explainability
    )

    return {
        'transparency': round(transparency, 3),
        'reasoning': round(reasoning, 3),
        'auditability': round(auditability, 3),
        'compliance': round(compliance, 3),
        'explainability': round(explainability, 3),
        'overall': round(overall, 3),
    }
```

## Frontend Consumption

The frontend (`useEUAsyncChat.ts`, lines 200-223) now correctly parses the JSON:

```typescript
const result = await resultResponse.json();

const finalMessage: IEUMessage = {
  id: assistantId,
  role: "assistant",
  content: result.answer,  // ✓ Markdown from JSON.answer
  timestamp: new Date().toISOString(),
  isStreaming: false,
  metadata: {
    model: result.model_used,  // ✓ 'claude-opus-4-6'
    confidence_score: result.confidence_score,  // ✓ 0.85
    confidence_tier: result.confidence_tier?.tier,  // ✓ 'HIGH'
    input_tokens: result.metadata?.input_tokens,
    output_tokens: result.metadata?.output_tokens,
    elapsed_seconds: result.metadata?.elapsed_seconds,
    human_review: result.human_review,  // ✓ true if confidence < 0.70
    trace_dimensions: result.trace_dimensions,  // ✓ Full T-R-A-C-E object
    job_id: jobId,
  },
};
```

## Consequences

### Positive

1. **TRACE scores now visible** - UI can display confidence breakdown, enabling users to assess response reliability
2. **Human review flagging works** - Low-confidence responses (< 70%) are automatically flagged for oversight (EU AI Act Article 14 compliance)
3. **Token usage tracked** - Cost monitoring restored for async requests
4. **Type safety** - Frontend TypeScript types now match actual response structure
5. **Consistent with streaming mode** - Both streaming (`useEUStreamingMessage`) and async modes now return the same metadata structure
6. **Auditability** - S3 object metadata includes job_id, confidence, timestamp for audit logs
7. **No breaking changes** - Frontend code already expected this format; we fixed the backend to match

### Negative

1. **Slightly larger S3 objects** - JSON wrapper adds ~500 bytes overhead vs raw markdown (negligible)
2. **Token estimation** - Output tokens are estimated (`len(response.split()) * 1.3`) until we capture actual Bedrock usage stats (tracked in backlog)

## Implementation Files

- **Lambda**: `crawlq-lambda/SemanticGraphEU/EUChatJobWorker/handler.py` (lines 102-122, 327-433)
- **Frontend**: `crawlq-ui/src/queries/chat-eu/useEUAsyncChat.ts` (lines 200-223)
- **Deployment**: `crawlq-lambda/SemanticGraphEU/deploy_async_chat_fix.py`

## Deployment

Deployed: 2026-02-13 02:03 UTC
Lambda: `eu_chat_job_worker` (Version 1)
Region: eu-central-1

## Related ADRs

- **ADR-012**: Tier-3 Async Markdown Architecture (established async flow)
- **ADR-006**: TRACE Compliance Protocol (defined scoring methodology)
- **ADR-022**: Chat Mode Rules (streaming vs async thresholds)

## Notes

- This fix resolves the root cause of "TRACE scores missing" in async mode
- Streaming mode already returned JSON correctly; this brings async mode to parity
- Future enhancement: Capture actual token usage from Bedrock response instead of estimation
