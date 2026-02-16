# ADR-037: COMMIT 9 — Subscription Fix, KG/RAG Panel, Audit Trail, UX Clarity

**Date:** 2026-02-14 | **Status:** ACCEPTED

## Context

After COMMIT 8 (63 PASS / 0 FAIL on Pallas E2E), several critical gaps remained:

1. **All users showed as Explorer** — Subscription data flow had a type mismatch; EU Lambda returns correct tier data but frontend `PREMIUM_PLAN_TYPES` typing caused fallthrough to Explorer.
2. **Per-response KG + RAG panel missing** — `useResponseKGQuery` hook existed but was never called; no RAG chunk display existed.
3. **Session KG crash** — `KnowledgeGraphPanelEU` required `graphData` as non-optional prop; crashed when undefined.
4. **Mermaid syntax errors in UI** — 3-level sanitization wasn't catching all edge cases.
5. **Search vs TRACE unclear** — No per-response indicator showing which mode produced a given response.
6. **Document vs Query KG filtering** — No way to distinguish document-sourced vs query-sourced entities.
7. **Server-side query counter** — localStorage-only counter was bypassable.
8. **Audit trail viewer** — Hooks existed but no UI.

## Decision

Implement all 8 phases as a single cohesive commit:

### Phase 1: Subscription Data Flow Fix
- Widen types in `IPlanDetails` and `IUserPlanAddons` to accept EU response shapes
- Add direct match on exact EU tier identifiers (`eu-enterprise`, `eu-business`, etc.) in `resolveEUTier()`
- Normalize EU Lambda response in `useUserPlan.ts` (unwrap envelope, fill missing fields)

### Phase 2: Session KG Stability
- Make `graphData` optional in `KnowledgeGraphPanelEU` with defensive `useMemo` default
- Create `ErrorBoundaryEU` React class component
- Wrap KG in error boundary on page.tsx

### Phase 3: Per-Response KG + RAG Sources
- Create `ResponseKGPanel` — inline KG per response with entity badges, fullscreen, feature-gated
- Create `ResponseSourcesPanel` — RAG chunks with relevance scores, expandable text
- Wire into `ChatMessageBubble` with "Explore KG" button and auto-display of RAG chunks
- Add `rag_chunks` and `source_documents` to `IEUMessage.metadata`

### Phase 4: Mermaid Error Handling
- Add Level 4 "nuclear" sanitization (unclosed subgraphs, classDef removal, colon escaping)
- Improve fallback UI from raw code block to user-friendly "Diagram couldn't render" with toggle

### Phase 5: Search/TRACE Per-Response Indicators
- Add `mode`, `web_search_used`, `trace_enabled` to message metadata
- Store active mode when streaming starts
- Display Globe/Network/FlaskConical badges per response

### Phase 6: KG Source Type Filtering
- Add `sourceType` field to `KGNodeEU` (document | query | inferred)
- Tag nodes in `transformGraphData()` based on `sourceDocument` presence
- Add "Source Type" filter section in `GraphFilterEU` with checkboxes
- Visual distinction in `GraphVisualizationEU`: solid (doc), dashed (query), dotted (inferred) borders
- Source type legend in graph stats overlay

### Phase 7: Server-Side Query Counter
- Add `queryUsage` endpoint to EU region config
- Create `useQueryUsageQuery` hook (React Query, fire-and-forget POST)
- Hybrid model in `useQueryCounter`: localStorage for instant UX, server for authority
- Sync on page load using `Math.max(local, server)` to prevent bypass

### Phase 8: Audit Trail Viewer
- Create `AuditTrailPanel` — full-screen viewer with date range picker, chain verification, stats cards, hash chain visualization
- Add "Audit Trail" toggle to `ChatToolbar` (gated to Business+ via `canUseAuditTrail`)
- Wire into page.tsx with upgrade modal fallback for lower tiers
- Add audit trail trigger message in `UpgradeModal`

## Consequences

**Positive:**
- All 4 plan tiers now resolve correctly from EU Lambda responses
- Users see which mode (Search/TRACE/Deep Research) produced each response
- Per-response KG and RAG chunks provide full transparency into AI reasoning
- KG source type filtering enables transparent provenance tracking (EU AI Act Art. 13)
- Audit trail with Merkle chain verification enables tamper-evident logging (GDPR Art. 15)
- Server-side query counter prevents localStorage bypass

**Negative:**
- Chat page bundle size increased (~666kB with all new panels)
- Server-side query counter requires DynamoDB table + Lambda deployment (infrastructure pending)
- Audit trail verification depends on backend Merkle chain implementation

**Trade-offs:**
- Chose hybrid local+server counter model over pure server-side to avoid latency on every query
- Chose full-screen overlay for audit trail (vs inline panel) to give room for hash chain visualization
- Chose entity badge display for response KG (vs D3 graph) to keep it compact inline
