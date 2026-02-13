# ADR-008: Claude Opus 4.6 as Primary EU Bedrock Model
**Date:** 2026-02-10 | **Status:** ACCEPTED
**Context:** EU Chat Athena was defaulting to `eu.anthropic.claude-sonnet-4-20250514-v1:0` (Sonnet 4) for all Bedrock calls. User requires Opus 4.6 for higher quality responses.
**Decision:** Changed default Bedrock model to `eu.anthropic.claude-opus-4-6-v1` (Opus 4.6) across all 17 EU Lambdas via:
1. Updated default in `shared/eu_config.py` line 32
2. Set `EU_BEDROCK_MODEL_ID` and `EU_BEDROCK_STREAMING_MODEL_ID` env vars on all 17 Lambdas
3. Redeployed 7 Lambdas that directly use the model
**Consequences:**
- (+) Higher quality AI responses (Opus 4.6 is the most capable model)
- (+) Consistent model across chat, insights, graph building, and reasoning
- (-) Higher cost per invocation vs Sonnet
- (-) Potentially higher latency (Opus is larger than Sonnet)
- Future: May use Sonnet for summarization tasks (EU_BEDROCK_SUMMARIZER_MODEL_ID) for cost savings
