# ADR-038: TRACE Scoring v2.0 — Structural Evidence-Based Scoring

**Date:** 2026-02-15 | **Status:** ACCEPTED

## Context

TRACE scoring v1.0 used keyword pattern matching to compute dimension scores:
- Transparency: counted words like "source:", "according to", "cited" (+0.06 each)
- Reasoning: counted "because", "therefore", "since" (+0.06 each)
- Compliance: counted "gdpr", "eu ai act", "article" (+0.07 each)

This produced absurd results: a world-class response with mermaid diagrams, tables, and source attribution scored 35% transparency and 34% compliance — because the AI chose slightly different vocabulary than the keyword list.

Additionally, the LLM was generating its own TRACE score tables in responses (self-assessing 90%+), creating confusing dual scores that contradicted the backend-computed scores.

## Decision

Replace keyword matching with structural evidence-based scoring:

1. **Transparency** = source_coverage_ratio * 0.35 + source_quality * 0.25 + web_sources * 0.15 + model_disclosure * 0.15 + base * 0.10
2. **Reasoning** = kg_grounding_ratio * 0.30 + structure_score (headings, lists, tables, mermaid) + evidence_chain + base * 0.15
3. **Auditability** = audit_logged * 0.30 + traceability * 0.25 + structured_output + kg_entities + base * 0.05
4. **Compliance** = data_residency * 0.20 + art50_disclosure * 0.20 + art14_oversight * 0.20 + review_compliance * 0.20 + gdpr_consent * 0.20
5. **Explainability** = interpretability * 0.30 + completeness * 0.25 + diversity * 0.20 + readability * 0.25

Each factor is computed from actual pipeline data (RAG chunk counts, relevance scores, KG entity counts, audit trail status), NOT from scanning the answer text for keywords.

System prompt now explicitly prohibits LLM from generating TRACE score tables.

## Consequences

- (+) Scores reflect actual response quality, not vocabulary choices
- (+) Compliance dimension correctly scores high (our pipeline structurally guarantees GDPR, Art. 14, Art. 50)
- (+) No more confusing dual scores (LLM vs backend)
- (+) Scores improve with more evidence (upload documents, enable web search, build KG)
- (-) Scores will be lower for responses without RAG/KG grounding (by design — lower evidence = lower confidence)
- (-) Requires pipeline_metrics parameter for full scoring (backward-compatible default)
