# ADR-006: TRACE Compliance Protocol for EU AI Act
**Date:** 2026-02-09 | **Status:** ACCEPTED
**Context:** EU AI Act requires transparency, explainability, and auditability for AI systems. Need a structured protocol embedded in every AI response.
**Decision:** TRACE (Transparency, Reasoning, Auditability, Compliance, Explainability) protocol:
1. **Transparency:** Every AI response includes model name, confidence score, and source attribution
2. **Reasoning:** 5-tier confidence system (GREEN/BLUE/ORANGE/RED/MAROON) with reasoning chain
3. **Auditability:** All requests/responses logged via eu_audit_trail_store Lambda, verifiable via eu_audit_trail_verify
4. **Compliance:** eu_compliance_engine checks GDPR + EU AI Act compliance on content
5. **Explainability:** eu_trace_explainer provides human-readable explanations, ChatTraceCard renders inline in UI
**Consequences:**
- (+) Full EU AI Act Article 50 compliance (transparency obligation)
- (+) Auditable trail for every AI interaction
- (+) Users can inspect confidence and reasoning for any response
- (-) Additional latency from compliance checks (~200ms per request)
- (-) Frontend complexity with TRACE card rendering
