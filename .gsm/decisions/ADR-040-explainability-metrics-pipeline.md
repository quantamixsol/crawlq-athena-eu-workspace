# ADR-040: Explainability Metrics Pipeline — 6 Sub-Metrics in Chat Response

**Date:** 2026-02-15 | **Status:** ACCEPTED

## Context

The frontend MetricsGridEU component was built to display 6 explainability sub-metrics (fidelity, interpretability, completeness, consistency, bias, stability) but received no data — the backend only computed 5 TRACE dimensions (T/R/A/C/E).

The EUTraceExplainer Lambda had 5 sub-metrics (fidelity, completeness, interpretability, bias, consistency) but these were only available in the TraceExplainer response, not in the main chat response.

## Decision

Add `compute_explainability_metrics()` to `shared/trace_scoring.py` that computes all 6 metrics (0-100 scale) from structural pipeline data:

1. **Fidelity** (RAG relevance 40% + KG confidence 30% + evidence presence 20% + base 10%)
2. **Interpretability** (structure: headings/lists/tables/mermaid + readability + base)
3. **Completeness** (coverage ratio of available factors 60% + length bonus + base)
4. **Consistency** (source convergence 50% + KG consistency + base)
5. **Bias** (Shannon entropy-based source balance 60% + KG independence 20% + base)
6. **Stability** (confidence 40% + evidence depth 30% + source diversity + base)

These are computed alongside TRACE dimensions in both handler.py and streaming_handler.py, included in the response as `explainability_metrics`.

Frontend wiring:
- `useChatEUStore.ts`: Added `explainability_metrics` to `IEUMessage.metadata`
- `useEUStreamingMessage.ts`: Extracts from response, passes through
- `ChatTraceCard.tsx`: Displays 6 metrics in a 3-column grid when expanded
- `ChatMessageBubble.tsx`: Passes `explainability_metrics` to ChatTraceCard

## Consequences

- (+) MetricsGridEU now receives actual computed data
- (+) All 6 metrics computed from structural evidence (same philosophy as TRACE v2.0)
- (+) Visible in every chat message's TRACE card (surface-level for all users)
- (-) Additional computation per response (~5ms overhead, negligible)
