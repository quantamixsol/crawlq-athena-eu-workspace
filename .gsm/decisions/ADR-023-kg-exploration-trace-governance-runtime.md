# ADR-023: KG Exploration UI + TRACE Governance Runtime

**Date:** 2026-02-12 | **Status:** ACCEPTED

---

## Context

CrawlQ Athena EU is a safety-and-governance-first runtime for high-consequence systems. The existing codebase has:

- **KnowledgeGraphPanelEU**: Full SVG-based KG viewer (14 entity types, zoom/pan, filter, search, stats, export) — currently **document-level only**
- **EUGraphBuilder Lambda**: Entity/relationship extraction from documents via LLM (18 entity types, confidence scoring, lineage tracking)
- **TRACE Protocol**: 5-dimensional scoring (T-R-A-C-E), 5-tier confidence (GREEN/BLUE/ORANGE/RED/MAROON), SHA-256 audit chain, human oversight routing, compliance metadata
- **ADR-022**: Smart chat mode rules (Web Search / TRACE / Combined / Plain chat)

### What's Missing

1. No per-response KG generation — only document-level
2. No whole-chat accumulated KG view
3. No decision lineage / context provenance visualization
4. No pre-execution governance gate (allow/refuse before commitment)
5. No fail-closed circuit breaker (rollback + closure semantics)
6. No cryptographic verifiability beyond simple SHA-256 chain

### Research Basis

| Source | Key Contribution | Applied To |
|--------|-----------------|-----------|
| Hippo (UIST 2025) | Interactive reasoning trees for chain-of-thought | Decision lineage DAG visualization |
| VCP v1.1 (VeritasChain Protocol) | Ed25519 + Merkle tree cryptographic audit trails | Enhanced audit chain (Phase 4C) |
| Microsoft GraphRAG | Global/local community-based graph summarization | Session KG accumulation strategy |
| GaaS (arXiv 2508.18765) | Governance-as-a-Service, policy-driven AI gates | Pre-execution governance gate |
| Trillian (RFC 6962) | Verifiable Merkle tree logs | Audit chain integrity proofs |
| Neo4j Bloom / Cytoscape.js | Graph visualization UX patterns | KG panel interaction design |

---

## Decision

Implement a 3-layer architecture with 5 phases:

### Architecture: Three Layers

```
+-----------------------------------------------------+
| Layer 1: Per-Response KG + Decision Lineage          |
|  - Extract KG from each response's context           |
|  - Show RAG chunks -> KG entities -> answer chain    |
|  - On-demand per response (button click)             |
+-----------------------------------------------------+
| Layer 2: Session KG Accumulator                      |
|  - Merge per-response KGs into session-wide graph    |
|  - Always-accessible from toolbar                    |
|  - Entities gain importance across responses          |
+-----------------------------------------------------+
| Layer 3: TRACE Governance Runtime                    |
|  - Pre-execution governance gate                     |
|  - Fail-closed circuit breaker                       |
|  - Cryptographic audit improvements (Merkle trees)   |
+-----------------------------------------------------+
```

---

## Sprint Plan: 5 Phases

### Phase 1: Per-Response KG Generation (Backend)

#### 1A. New Lambda: `eu_response_kg_extractor`

**Path:** `SemanticGraphEU/EUResponseKGExtractor/handler.py`
**Purpose:** On-demand KG extraction from a chat response + its RAG context. Lightweight — no Neo4j, returns KG data directly.

**Input contract:**
```json
{
  "answer": "The response text...",
  "question": "Original query",
  "rag_chunks": [{"content": "...", "source": "...", "score": 0.8}],
  "web_sources": [{"title": "...", "url": "...", "snippet": "..."}],
  "kg_entities": [{"name": "...", "type": "..."}],
  "trace_dimensions": {"transparency": 0.85, ...},
  "username": "user@example.com",
  "workspace": "default"
}
```

**Output contract:** `KGGraphDataEU`-compatible structure:
```json
{
  "nodes": [
    {"id": "...", "name": "EU AI Act", "type": "REGULATION", "confidence": 0.95, "sourceQuote": "..."},
    {"id": "query", "name": "User Query", "type": "DECISION", "importance": 10},
    {"id": "rag-1", "name": "RAG: document.pdf", "type": "PROCESS", "importance": 7},
    {"id": "answer", "name": "AI Response", "type": "OUTCOME", "importance": 10}
  ],
  "relationships": [
    {"sourceId": "query", "targetId": "rag-1", "type": "RETRIEVED_FROM", "confidence": 0.8},
    {"sourceId": "rag-1", "targetId": "entity-1", "type": "MENTIONS", "confidence": 0.9},
    {"sourceId": "entity-1", "targetId": "answer", "type": "GROUNDED_BY", "confidence": 0.85}
  ],
  "metadata": {
    "responseId": "assistant-123",
    "totalNodes": 12,
    "totalRelationships": 18,
    "extractionModel": "claude-opus-4-6"
  }
}
```

**Logic:**
1. Parse response text -> extract entities (adapted from EUGraphBuilder prompt)
2. Parse RAG chunks -> create source nodes with provenance
3. Parse web sources -> create web citation nodes
4. Build decision lineage: Query -> Sources -> Entities -> Answer
5. Score relationships by confidence (from TRACE dimensions)

**Config:** Memory 512MB, Timeout 60s, Python 3.10, Function URL auth-type NONE

#### 1B. Decision Lineage DAG

Inside the same Lambda, generate a decision lineage DAG:
```json
{
  "lineage": [
    {"stage": "query", "label": "User Query", "detail": "What is EU AI Act?", "timestamp": "..."},
    {"stage": "web_search", "label": "Web Search (Perplexity)", "detail": "5 sources retrieved", "score": 0.9},
    {"stage": "rag_retrieval", "label": "RAG Retrieval", "detail": "3 document chunks", "score": 0.8},
    {"stage": "kg_grounding", "label": "KG Grounding", "detail": "4 entities matched", "score": 0.85},
    {"stage": "generation", "label": "LLM Generation", "detail": "Claude Opus 4.6", "tokens": 1024},
    {"stage": "trace_scoring", "label": "TRACE Scoring", "detail": "Overall: 0.80", "dimensions": {}},
    {"stage": "governance", "label": "Governance Gate", "detail": "Auto-approved (BLUE tier)", "action": "allow"}
  ]
}
```

**Rationale:** Inspired by Hippo (UIST 2025) interactive reasoning trees. Each stage in the pipeline becomes a node in a horizontal DAG, providing full context provenance from query to governance decision.

---

### Phase 2: Per-Response KG (Frontend)

#### 2A. "Explore" Button on ChatMessageBubble

**File:** `src/components/chat-eu/ChatMessageBubble.tsx`

Add an "Explore" button (Network icon) in the footer actions row (next to Copy and Share). Only visible for assistant messages with content.

On click: Set expandedKG state -> invoke Lambda -> show loading skeleton -> render inline KG panel.

#### 2B. ResponseKGPanel Component (NEW)

**Path:** `src/components/chat-eu/ResponseKGPanel.tsx` (~350 lines)

**Two tabs:**

**Tab 1: Knowledge Graph**
- Reuses `KnowledgeGraphPanelEU` in compact mode (height: 400px, no sidebar by default)
- Entity types color-coded (same as document KG)
- Special node types: QUERY (purple), SOURCE (blue), ANSWER (green)
- Click node -> see source quote, confidence, lineage position

**Tab 2: Decision Lineage**
- Horizontal DAG (left-to-right flow): Query -> Web Search -> RAG -> KG Grounding -> Generation -> TRACE -> Governance
- Each node: label, score badge, detail text
- Color-coded: green (high confidence), amber (medium), red (low)
- Animated connection lines (framer-motion pathLength)
- Click any stage -> expands to show details

#### 2C. useResponseKG Hook (NEW)

**Path:** `src/queries/chat-eu/useResponseKG.ts`
- Lazy fetch (only on button click)
- Caches result in React Query by messageId
- Returns: `{ data: KGGraphDataEU, lineage: LineageStage[], isLoading, error }`

---

### Phase 3: Session-Wide KG Accumulator

#### 3A. Session KG Store

**Modify:** `src/store/chat-eu/useChatEUStore.ts`

Add sessionKG state:
- `sessionKG: KGGraphDataEU | null` — Accumulated graph
- `sessionKGMessageIds: string[]` — Which messages contributed
- `mergeResponseKG(messageId, kg)` — Merge nodes/relationships, increment importance for recurring entities
- `clearSessionKG()` — Reset on new conversation

**Merge logic:** Nodes appearing in multiple responses get importance incremented. Duplicate relationships merged with max confidence.

#### 3B. Session KG Button in Toolbar

**Modify:** `src/components/chat-eu/ChatToolbar.tsx`

4th button: "Session KG" (GitBranch icon). Always visible. Entity count badge. Opens KnowledgeGraphPanelEU overlay.

#### 3C. Auto-KG Extraction (Optional Enhancement)

After each response completes, automatically extract KG in background (fire-and-forget). Pre-populates session KG without user clicking "Explore" on each response.

---

### Phase 4: TRACE Governance Runtime (Backend)

#### 4A. Pre-Execution Governance Gate

**Modify:** `EUChatAthenaBot/handler.py`

Before Bedrock invocation, evaluate governance:
- Check 1: Query safety (PII detection, injection attempts)
- Check 2: Source quality (avg RAG confidence threshold)
- Check 3: KG entity relevance (are grounding entities available?)
- Check 4: Historical trust (past confidence scores for this user/workspace)

Returns: `{ decision: "allow"|"warn"|"deny", reason, obligations, trust_score }`

**Rationale:** Aligned with GaaS (arXiv 2508.18765) governance-as-a-service pattern. Pre-execution gates prevent low-quality or potentially harmful responses before committing compute resources.

#### 4B. Fail-Closed Circuit Breaker

**New:** `shared/circuit_breaker.py`

Three states:
- **CLOSED** (normal): Process all requests
- **OPEN** (circuit tripped): Return safe fallback response with explanation + human review link
- **HALF-OPEN** (recovery probe): Process one request to test recovery

Metrics tracked per user/workspace:
- Consecutive Bedrock failures
- Average confidence score (rolling window of last 10 responses)
- Policy violation count

**Rationale:** Governance-first systems must fail closed, not open. When the system detects degraded quality, it should stop producing potentially misleading outputs and route to human review.

#### 4C. Enhanced Merkle Tree Audit Chain

**Modify:** `shared/audit_trail.py`

Upgrades from simple SHA-256 chain to cryptographically verifiable Merkle tree:
- Every N audit entries (configurable, default 100), compute a Merkle root
- Store Merkle root as a separate entry for efficient verification
- Add `verify_audit_chain(start_id, end_id)` function

**Rationale:** Aligned with VCP v1.1 spec and RFC 6962. Simple hash chains are O(n) to verify; Merkle trees enable O(log n) inclusion proofs. Required for true auditability at scale.

---

### Phase 5: Frontend Governance UI

#### 5A. GovernanceGateBadge Component (NEW)

**Path:** `src/components/chat-eu/GovernanceGateBadge.tsx`

Badge in ChatTraceCard:
- Green shield: "Auto-approved"
- Amber shield: "Warning -- review recommended"
- Red shield: "Blocked -- human review required"

#### 5B. Enhanced ChatTraceCard

**Modify:** `src/components/chat-eu/ChatTraceCard.tsx`

Add governance gate status, decision lineage mini-preview (3-4 key stages as pills), "Explore Full KG" button.

---

## Implementation Order

```
Phase 1A -> 1B -> 2C -> 2A -> 2B -> 3A -> 3B  (KG Track)
Phase 4A -> 4B -> 4C -> 5A -> 5B               (Governance Track)
```

KG track and Governance track are **independent** and can be worked in parallel.

**Minimum viable delivery:** Phase 1A + 2A + 2B + 2C = Per-response KG with decision lineage.

---

## Files to Create

| File | Est. Lines | Purpose |
|------|-----------|---------|
| `EUResponseKGExtractor/handler.py` | ~250 | Lambda: extract KG from response |
| `EUResponseKGExtractor/requirements.txt` | ~3 | Dependencies |
| `src/components/chat-eu/ResponseKGPanel.tsx` | ~350 | Inline KG + lineage viewer |
| `src/queries/chat-eu/useResponseKG.ts` | ~80 | Lazy KG fetch hook |
| `shared/circuit_breaker.py` | ~120 | Fail-closed circuit breaker |
| `src/components/chat-eu/GovernanceGateBadge.tsx` | ~60 | Governance status badge |

## Files to Modify

| File | Change |
|------|--------|
| `ChatMessageBubble.tsx` | Add "Explore" button, expand/collapse ResponseKGPanel |
| `ChatTraceCard.tsx` | Add governance badge + lineage mini-preview |
| `useChatEUStore.ts` | Add sessionKG state + merge function |
| `ChatToolbar.tsx` | Add "Session KG" button |
| `page.tsx` | Wire Session KG overlay |
| `region-config.ts` | Add responseKGExtractor endpoint |
| `EUChatAthenaBot/handler.py` | Add governance gate before Bedrock |
| `shared/audit_trail.py` | Add Merkle tree root computation |
| `deploy_deep_research_lambdas.py` | Add eu_response_kg_extractor |

## Existing Code to Reuse

| Component | Source | Reuse For |
|-----------|--------|-----------|
| `KnowledgeGraphPanelEU` | `knowledge-graph-eu/` | Session KG + per-response KG display |
| `KGGraphDataEU` types | `types/kg-types-eu.ts` | Data contract for all KG operations |
| `kg-utils-eu.ts` | `knowledge-graph-eu/` | Filter, search, stats, export |
| Entity extraction prompt | `EUGraphBuilder/helpers.py:829-997` | Adapt for response-level extraction |
| `GraphVisualizationEU` | `knowledge-graph-eu/` | SVG rendering engine |
| `AnimatedConfidenceGauge` | `trace-intelligence/` | Governance gate confidence display |

---

## Consequences

### Positive

- **Full context provenance**: Users can trace any response back through RAG chunks -> KG entities -> source documents -> governance decision
- **Session-wide intelligence**: Accumulated KG reveals cross-response entity patterns and importance ranking
- **Governance-first**: Pre-execution gate prevents low-quality responses before they reach users
- **Fail-closed safety**: Circuit breaker protects against cascading failures and quality degradation
- **Cryptographic verifiability**: Merkle tree audit chain enables O(log n) inclusion proofs (vs O(n) for simple hash chain)
- **Research-grounded**: Every design decision backed by academic/industry research (Hippo, VCP, GaaS, GraphRAG)

### Negative

- **Additional Lambda cost**: `eu_response_kg_extractor` invokes Bedrock for entity extraction (~$0.02 per extraction)
- **Latency for KG extraction**: On-demand extraction adds 5-15s when user clicks "Explore" (mitigated by lazy loading + caching)
- **Complexity increase**: 6 new files + 8 modified files increases maintenance surface
- **Merkle tree overhead**: Computing Merkle roots adds ~10ms per batch of 100 audit entries (negligible)

### Trade-offs

- **On-demand vs auto-extraction**: Phase 3C (auto-KG) is optional — starts with user-initiated to minimize Lambda costs
- **Circuit breaker state**: Stored in DynamoDB (durable) vs in-memory (fast) — chose DynamoDB for cross-invocation persistence
- **Governance gate granularity**: Per-request (chosen) vs per-session — per-request is more precise but adds ~50ms latency

---

## Verification Criteria

1. **Per-Response KG:** Send chat message -> click "Explore" -> verify KG shows entities from response + decision lineage DAG
2. **Session KG:** Send 3 messages -> click "Session KG" -> verify accumulated graph with entities from all 3 responses
3. **Decision Lineage:** Verify each stage: query -> sources -> grounding -> generation -> scoring -> governance
4. **Governance Gate:** Low-confidence query -> verify "warn" badge in TRACE card
5. **Circuit Breaker:** 3 consecutive failures -> verify "open" state with safe fallback
6. **Audit Chain:** Verify Merkle root computation and `verify_audit_chain()` function
7. **Build:** `npx next build` -- zero errors
8. **Deploy:** boto3 ZIP deploy for new + updated Lambdas

---

## Cross-References

- **ADR-006**: TRACE Compliance Protocol (foundation for TRACE dimensions)
- **ADR-014**: boto3 Deployment (deployment pattern for new Lambda)
- **ADR-018**: Lambda Function URLs (auth-type NONE pattern)
- **ADR-019**: EU LLM Fallback Strategy (model chain for extraction)
- **ADR-022**: Chat Mode Rules (mode matrix this builds upon)
