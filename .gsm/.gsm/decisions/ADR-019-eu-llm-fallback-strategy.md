# ADR-019: EU LLM Fallback Strategy — Bedrock Primary, No Non-EU SDKs
**Date:** 2026-02-12 | **Status:** ACCEPTED

**Context:** The `eu_generate_deep_insights` Lambda needs LLM access for graph generation, text summarization, and insight generation. The original code tried three SDKs in sequence:
1. Anthropic SDK (direct API — needs API key, data leaves EU)
2. Google Gemini SDK (direct API — needs GCP credentials, data leaves EU)
3. OpenAI SDK (direct API — needs API key, data leaves EU)

None were configured, and more critically, all three send data to US-hosted API endpoints, violating GDPR Art. 44-49 (data transfer restrictions) and the EU AI Act Art. 13 (transparency about where data is processed).

**Decision:**
- **Primary**: Amazon Bedrock in `eu-central-1` (Frankfurt) — all data stays in EU. Uses IAM role auth, no API keys needed.
  - **Opus 4.6** (`eu.anthropic.claude-opus-4-6-v1`) for insight generation (highest quality)
  - **Sonnet 4.5** (`eu.anthropic.claude-sonnet-4-5-20250929-v1:0`) for graph generation and summarization (3-5x faster, sufficient quality for extraction tasks)
- **No fallback to OpenAI or Gemini** until either:
  1. OpenAI/Google offer EU-resident inference endpoints with GDPR-compliant DPAs, OR
  2. We implement a consent mechanism where users explicitly opt in to non-EU processing (GDPR Art. 49(1)(a))
- The frontend should display which model was used (primary vs fallback), per EU AI Act Art. 50 transparency requirements.

**Consequences:**
- Positive: Full GDPR compliance — all data processed in eu-central-1
- Positive: No API key management — uses IAM role auth via Bedrock
- Positive: Single billing through AWS (no separate OpenAI/Google billing)
- Negative: No multi-provider redundancy — if Bedrock has an outage, insights generation fails
- Negative: Limited to models available on Bedrock EU (currently Claude family only)
- Mitigation: We can add Azure OpenAI (EU West) as a GDPR-compliant fallback in the future, since Azure offers EU-resident endpoints
- Future: When Google Vertex AI is available in eu-central-1, Gemini can be added as a EU-compliant fallback
