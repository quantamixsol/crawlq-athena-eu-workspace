# ADR-020: Rolling Context Window for Document Analysis
**Date:** 2026-02-12 | **Status:** ACCEPTED

## Context
Large documents (100K-600K+ chars) overwhelm single LLM calls due to:
1. Context window limits (200K tokens for Claude, but practical quality degrades at extremes)
2. Output quality drops with very large inputs (attention dilution)
3. Token costs scale linearly with redundant re-processing

Current approach: chunk text into fixed 50K blocks, process each independently, merge at the end. This loses cross-chunk context and produces fragmented insights.

## Decision: Rolling Context Window with Memory Compaction

Inspired by Claude Code's context management (auto-compaction when nearing limits), we implement a **rolling window** that:

### Architecture

```
Document (N chars)
  |
  v
[Token Estimator] — estimates tokens from char count (chars / 3.5 ≈ tokens)
  |
  v
[Window Planner] — calculates optimal window sizes
  |          Window = 80% of model's max input tokens
  |          Overlap = 20% of window (rolling carry-forward)
  |
  v
[Pass 1: Window 1]  ──→  [Compact] ──→  accumulated_context_1
  |                                              |
[Pass 2: Window 2 + accumulated_context_1] ──→ [Compact] ──→ accumulated_context_2
  |                                              |
[Pass N: Final Window + accumulated_context_N-1] ──→ [Final Synthesis]
  |
  v
[Merged Insights + Graph + Summary]
```

### Key Concepts

1. **Token Estimation**: `tokens ≈ len(text) / 3.5` (empirically calibrated for English text)

2. **Window Planning**:
   - Model max context: 200K tokens (Claude Opus 4.6/Sonnet 4.5)
   - Usable input window: 80% = 160K tokens ≈ 560K chars
   - Reserved for output: 20% = 40K tokens (enough for detailed JSON)
   - For each window pass: input = new_text + accumulated_context
   - accumulated_context target: 10K chars (compacted from previous passes)

3. **Memory Compaction** (per-pass):
   - After each window, the LLM extracts structured memory:
     ```json
     {
       "key_entities": [...],
       "key_relationships": [...],
       "running_insights": [...],
       "compliance_flags": [...],
       "cross_references": [...]
     }
     ```
   - This compacted memory is carried forward to the next window
   - Similar to Claude Code's context auto-compaction: preserve essential information, discard verbatim text

4. **Overlap Zone**:
   - Last 20% of each window is re-included in the next window
   - Prevents losing insights at chunk boundaries
   - The overlap chars are NOT counted toward the "new text" budget

5. **Final Synthesis**:
   - After all windows processed, a final synthesis pass combines:
     - All accumulated compacted memories
     - The last window's raw text (most recent context)
     - Merged graph data from all passes
   - Produces the final insights, executive summary, and compliance analysis

### Processing Pipeline

```
estimate_tokens(document_text)
  |
  if tokens <= SINGLE_PASS_THRESHOLD (100K tokens ≈ 350K chars):
    → Single-pass analysis (current approach, fast)
  |
  else:
    → Rolling window analysis:
      1. Plan windows (with overlap)
      2. For each window:
         a. Combine: new_text + accumulated_context
         b. Call Sonnet for graph extraction (fast)
         c. Call Sonnet for memory compaction (fast)
         d. Update accumulated_context
      3. Final synthesis pass with Opus (high quality)
      4. Merge all graph data
      5. Store results
```

### Model Strategy Per Stage

| Stage | Model | Why |
|-------|-------|-----|
| Graph extraction | Sonnet 4.5 | Fast, good at structured extraction |
| Memory compaction | Sonnet 4.5 | Fast, good at summarization |
| Window insights | Sonnet 4.5 | Process each window quickly |
| Final synthesis | Opus 4.6 | Highest quality for final analysis |

### Performance Projections

| Document Size | Current Approach | Rolling Window |
|--------------|-----------------|----------------|
| 50K chars (small) | 1 pass, ~110s | 1 pass, ~110s (no change) |
| 200K chars (medium) | 4 chunks, ~200s | 1 pass, ~120s |
| 600K chars (large) | 12 chunks, ~400s | 2 passes + synthesis, ~200s |
| 2M chars (huge) | 40 chunks, ~900s+ | 6 passes + synthesis, ~400s |

### Edge Cases Handled

1. **Very small documents** (<10K chars): Skip windowing, single prompt
2. **Scanned PDFs with OCR errors**: Window overlap catches partial sentences
3. **Multi-language documents**: Token estimator adjusts for non-Latin scripts (÷3.0 instead of ÷3.5)
4. **Documents with tables/data**: Graph extraction prioritizes structured data in windows
5. **Memory compaction failure**: Fall back to truncation (last N chars) if LLM fails
6. **Document with repetitive sections**: Compaction deduplicates recurring entities/themes

## Consequences

- **Positive**: 2-4x faster for large documents (fewer total LLM calls)
- **Positive**: Higher quality insights (cross-chunk context preserved via memory)
- **Positive**: No lost insights at chunk boundaries (overlap zone)
- **Positive**: Predictable cost scaling (proportional to document size, not quadratic)
- **Positive**: Novel approach — not seen in competing document analysis tools
- **Negative**: More complex code (rolling state management)
- **Negative**: Memory compaction adds small overhead per window
- **Mitigation**: Single-pass shortcut for documents under 350K chars (most documents)
