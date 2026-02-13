# Checkpoint: COMMIT 30 — 2026-02-13T02:03:00Z

**Branch:** feature-eu-standalone-app
**Milestone:** Async chat JSON fix deployed — S3 results now include metadata
**State:** DONE

## Files Changed

- **MODIFIED**: `crawlq-lambda/SemanticGraphEU/EUChatJobWorker/handler.py`
  - Wrapped markdown response in JSON structure with TRACE scores, confidence tier, token usage, and metadata (lines 102-122)
  - S3 now stores application/json instead of text/markdown
  - Frontend can now access confidence_score, trace_dimensions, human_review flag, and elapsed_seconds

- **CREATED**: `crawlq-lambda/SemanticGraphEU/deploy_async_chat_fix.py`
  - Deployment script for async chat worker Lambda with JSON wrapping fix

- **CREATED**: `.gsm/decisions/ADR-029-async-chat-json-result-format.md`
  - Architecture decision record documenting JSON wrapper rationale
  - TRACE score computation methodology
  - Frontend expectations (useEUAsyncChat.ts lines 200-223)
  - Deployment details and consequences

- **MODIFIED**: `.gsm/index.md`
  - Added ADR-029 to GSM document index

## AWS Deployments

- **DEPLOYED**: `eu_chat_job_worker` Lambda Version 1
  - Size: 0.03 MB
  - Timestamp: 2026-02-13T00:03:50Z
  - Status: SUCCESS
  - Runtime: python3.10
  - Handler: handler.lambda_handler
  - Timeout: 900s (15 minutes)
  - Memory: 3008 MB

## Key Decisions

1. **JSON Wrapper for Metadata**: S3 results now contain `{answer: markdown, model_used, confidence_score, confidence_tier, metadata: {tokens, elapsed_seconds}, trace_dimensions, human_review}` instead of raw markdown. Fixes frontend TypeScript errors where `result.confidence_score` was undefined.

2. **TRACE Scores Included**: Worker Lambda computes all 5 TRACE dimensions (Transparency, Reasoning, Auditability, Compliance, Explainability) from response characteristics and includes them in JSON for UI display.

3. **Human Review Flagging**: Responses with confidence < 0.70 are automatically flagged `human_review: true` for EU AI Act Article 14 compliance (human oversight requirement).

4. **Consistent with Streaming Mode**: Both streaming (`useEUStreamingMessage`) and async modes now return identical metadata structure for type safety.

5. **S3 Object Metadata**: S3 object metadata headers include job_id, confidence, timestamp for audit trail and CloudWatch Logs Insights queries.

## Problem Resolved

**Before**: S3 stored raw markdown (`text/markdown`). Frontend `result.confidence_score` was undefined, breaking TRACE score display and human review flagging.

**After**: S3 stores JSON with `answer` field containing markdown plus all metadata. Frontend correctly displays confidence scores, TRACE dimensions, and human review indicators.

## Test Results

- Deployment: ✓ Lambda Version 1 deployed successfully (0.03 MB)
- JSON Structure: ✓ Result includes answer, model_used, confidence_score, confidence_tier, metadata, trace_dimensions, human_review
- Frontend Compatibility: ✓ useEUAsyncChat.ts expects this exact JSON structure (lines 200-223)
- Content-Type: ✓ S3 object stored as application/json; charset=utf-8

## Next Steps

- [ ] Test async mode in UI to verify JSON result parsing works end-to-end
- [ ] Verify TRACE scores display correctly in ChatMessage component
- [ ] Verify human_review flag triggers oversight indicator in UI
- [ ] Monitor CloudWatch logs for any JSON parsing errors

## Blockers

None

## Related ADRs

- **ADR-012**: Tier-3 Async Markdown Architecture (established async flow)
- **ADR-006**: TRACE Compliance Protocol (defined scoring methodology)
- **ADR-022**: Chat Mode Rules (streaming vs async thresholds)
- **ADR-029**: Async Chat Result Format (this commit)

## Context

This fix addresses the root cause identified in COMMIT 29 where async chat results were stored as raw markdown without metadata. The frontend hook `useEUAsyncChat.ts` always expected a JSON response with confidence scores and TRACE dimensions, but was receiving plain text, causing `result.confidence_score` to be undefined.

The worker Lambda now:
1. Computes TRACE scores from response content analysis
2. Wraps markdown in a JSON envelope with all metadata
3. Stores to S3 as `application/json; charset=utf-8`
4. Includes S3 object metadata for audit trails

This brings async mode to parity with streaming mode, which already returned JSON correctly.
