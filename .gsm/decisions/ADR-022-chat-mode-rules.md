# ADR-022: Chat Response Mode Rules (Web Search / TRACE / Combined)
**Date:** 2026-02-12 | **Status:** ACCEPTED

## Context
The EU Athena chat supports multiple response modes: Web Search (Perplexity AI), TRACE protocol analysis (Bedrock + confidence scoring), and Deep Research (6-stage pipeline). Users need individual toggles for each mode, with clear rules for how modes interact. The previous implementation always routed through Bedrock even when only web search was requested, causing unnecessary latency and Bedrock availability errors.

## Decision

### Mode Matrix

| Web Search | TRACE | Deep Research | Backend Behavior |
|-----------|-------|---------------|------------------|
| OFF | ON | OFF | Standard Bedrock + TRACE scoring (`mode: "trace"`) |
| ON | OFF | OFF | Perplexity only, **skip Bedrock** (`mode: "web_search"`) |
| ON | ON | OFF | Perplexity + Bedrock + TRACE (`mode: "combined"`) |
| OFF | OFF | OFF | Basic Bedrock chat (`mode: "chat"`) |
| ON | ON | ON | Full 6-stage pipeline via EUDeepResearch |

### Toggle Interaction Rules
1. **Deep Research ON** → auto-enables Web Search + TRACE
2. **Web Search OFF** while Deep Research ON → auto-disables Deep Research
3. **TRACE OFF** while Deep Research ON → auto-disables Deep Research
4. **Deep Research OFF** → only disables Deep Research; Web Search + TRACE remain as-is

### Backend Protocol
- `EUChatAthenaBot` reads `web_search` and `trace_enabled` flags from request body
- When `mode == "web_search"`: returns Perplexity synthesized answer directly, skips Bedrock invocation entirely
- When `trace_enabled == false`: skips confidence scoring + TRACE dimensions computation
- Response includes `mode` field so frontend knows how to render

### Frontend Protocol
- TRACE card only renders when `trace_dimensions` exists in response metadata
- Thinking text adapts to mode: "Searching the web...", "Analyzing with TRACE protocol...", "Searching web + building TRACE analysis..."
- Auto-scroll triggers on streaming content changes and when streaming completes (cards appear)

## Consequences
- **Positive:** Web-search-only queries are 3-5x faster (no Bedrock round-trip)
- **Positive:** Bedrock unavailability doesn't block web search results
- **Positive:** Smart toggle UX prevents impossible states (e.g., Deep Research without its dependencies)
- **Positive:** `mode` field enables future mode-specific UI rendering
- **Negative:** Two code paths in EUChatAthenaBot (web-search-only vs Bedrock) — slightly more complexity
- **Trade-off:** TRACE defaults to ON for backward compatibility; new users see full TRACE immediately
