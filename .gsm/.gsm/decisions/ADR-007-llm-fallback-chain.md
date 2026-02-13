# ADR-007: LLM Fallback Chain (Anthropic -> Gemini -> OpenAI)
**Date:** 2026-02-09 | **Status:** ACCEPTED
**Context:** EU Lambda functions need reliable AI generation for chat, insights, and graph building. Single LLM dependency creates availability risk.
**Decision:** Three-tier fallback chain:
1. **Primary:** Anthropic Claude (via Bedrock for chat, direct API for insights) — EU data residency via Bedrock eu-central-1
2. **Secondary:** Google Gemini (via Vertex AI) — currently optional due to google-genai deployment constraint (ADR-002)
3. **Tertiary:** OpenAI GPT — global API, used as last resort
Each Lambda's helpers.py tries Anthropic first, catches exception, tries Gemini, catches, tries OpenAI.
**Consequences:**
- (+) High availability — if one provider is down, others pick up
- (+) Anthropic primary ensures EU data residency via Bedrock
- (-) Gemini currently unavailable in ZIP deploys (requires Docker for google-genai)
- (-) OpenAI data may leave EU (acceptable as last-resort fallback)
