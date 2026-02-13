# ADR-018: Use Lambda Function URLs for Long-Running Operations
**Date:** 2026-02-12 | **Status:** ACCEPTED
**Context:** The `eu_generate_deep_insights` Lambda performs multiple Bedrock LLM calls (graph generation + insight generation) which takes 60-360 seconds depending on document size. API Gateway HTTP API has a 30-second integration timeout. Direct Lambda invoke from the upload Lambda also hits boto3 default read timeouts (60s). These timeouts cause insight generation to fail silently, returning null insights.
**Decision:** Use Lambda Function URLs (not API Gateway) for any Lambda that:
1. Takes longer than 25 seconds to respond
2. Makes multiple sequential LLM calls (Bedrock, Anthropic, OpenAI)
3. Processes large documents (PDF text extraction + analysis)

Specifically:
- `eu_generate_deep_insights` — Function URL with 900s timeout
- `eu_deep_graph_builder` — Function URL with 600s timeout
- `eu_test_semantic` (Athena training) — Function URL with 600s timeout

For the upload flow: the upload Lambda should invoke insights generation asynchronously (`InvocationType: Event`) and the frontend should poll for completion, rather than waiting synchronously.

**Consequences:**
- Positive: No API Gateway 30s limit, no boto3 read timeout issues, direct Lambda invocation with configurable timeouts
- Positive: Function URLs support response streaming, enabling future progress updates
- Negative: Function URLs need separate CORS configuration per Lambda
- Negative: Need to manage IAM auth or use `AWS_IAM` auth type (not JWT like API Gateway)
- Mitigation: Use `NONE` auth type for internal Lambda-to-Lambda calls via invoke, use Function URL only for direct frontend access if needed
