# ADR-026: Interactive Canvas Technology Selection

**Date:** 2026-02-12
**Status:** PROPOSED
**Decision Makers:** Claude (Research), Product Team (Approval Pending)
**Related:** ADR-027 (Visualization Strategy)

---

## Context

CrawlQ's TRACE system currently provides explainability through:
- Static dashboards (5-panel TRACE UI)
- Knowledge graphs (Neo4j NVL - limited customization)
- Document analysis results (cards, tables, charts)

**Problem:** Users cannot:
1. Visually explore relationships between insights interactively
2. Build custom research workflows (multi-step AI pipelines)
3. Annotate and collaborate on findings
4. Create knowledge maps that connect documents, insights, and TRACE scores

**Opportunity:** Add an interactive canvas workspace (Miro-like) that enables:
- Visual workflow builder for research pipelines
- Freeform whiteboarding for ideation
- 3D knowledge graph exploration
- Markdown documents rendered on canvas with interlinkages
- Unique value proposition for PhD research, B2B analysts, and compliance teams

**Research:** Evaluated 8+ canvas libraries across 3 categories:
1. **Node-based workflow builders:** React Flow, Cytoscape.js
2. **Freeform whiteboards:** Tldraw, Excalidraw
3. **Low-level canvas:** Konva.js, Fabric.js

Full research: [.gcc/branches/research-interactive-canvas/RESEARCH_REPORT.md](../../.gcc/branches/research-interactive-canvas/RESEARCH_REPORT.md)

---

## Decision

We will implement a **hybrid multi-mode canvas system** using:

### Mode 1: Workflow Builder — React Flow (@xyflow/react)
**For:** Structured research pipelines, multi-model orchestration

**Rationale:**
- ✅ **Production-proven** — used by Stripe, Typeform, Instagram
- ✅ **TypeScript-native** with excellent DX
- ✅ **React-first** — nodes are React components (easy to customize)
- ✅ **Rich plugin ecosystem** — minimap, controls, background
- ✅ **Built-in features** — multi-select, undo/redo, keyboard shortcuts
- ✅ **Best for workflows** — purpose-built for node-edge paradigm

**Trade-offs:**
- ❌ DOM-based (less performant than canvas for 10K+ nodes)
- ❌ Not suited for freeform drawing

**Use cases:**
- Multi-model workflows (GPT-4 → Claude → Validator → Report)
- Research pipelines (Search → Extract → Analyze → Export)
- Data lineage visualization (Sources → Processing → Outputs)
- Tool orchestration (drag tools from library, connect with logic)

---

### Mode 2: Freeform Canvas — Tldraw

**For:** Brainstorming, mind maps, document annotation

**Rationale:**
- ✅ **Best-in-class UX** — infinite canvas, smooth pan/zoom
- ✅ **Freeform tools** — pen, shapes, arrows, text, sticky notes
- ✅ **Real-time collaboration ready** — live cursors, viewport following
- ✅ **OpenGL minimap** — performance optimized
- ✅ **Extensible** — custom shapes and tools
- ✅ **44K GitHub stars** — active community

**Trade-offs:**
- ❌ **Watermark required** ("Made with tldraw") unless commercial license
- ❌ Not optimized for structured workflows

**Use cases:**
- Freeform research ideation (mind maps, concept sketches)
- Visual document annotation (mark up PDFs)
- Collaborative whiteboarding sessions
- Sticky note style insight exploration

**License:** Will require commercial license (~$1000-5000/year estimated) to remove watermark for enterprise use.

---

### Mode 3: Knowledge Graph — Reagraph (3D WebGL)

**For:** Large-scale knowledge graph visualization

**Rationale:**
- ✅ **3D visualization** — unique differentiator (no competitor offers this)
- ✅ **WebGL-powered** — handles 10K+ nodes smoothly
- ✅ **Interactive** — zoom, pan, rotate, node click
- ✅ **React-native** — integrates seamlessly
- ✅ **Customizable nodes** — render custom React components
- ✅ **Force-directed layout** — auto-positioning

**Trade-offs:**
- ❌ Requires GPU (fallback to 2D needed)
- ❌ Smaller community than Cytoscape.js

**Fallback:** Cytoscape.js for 2D graph mode (100+ layout algorithms, graph analysis)

**Use cases:**
- Citation network exploration (papers → references)
- Entity relationship visualization (10K+ nodes)
- TRACE knowledge graphs (insights → entities → sources)
- Compliance impact mapping (requirements → processes → systems)

---

### Shared Components Across All Modes

**1. Markdown Overlay System**
- Render markdown documents as HTML overlays on canvas
- Positioned using absolute positioning (not canvas text rendering)
- Supports wikilinks (`[[Document Name]]`) with auto-connectors
- Tag-based filtering (`#compliance`, `#high-risk`)
- Bi-directional links panel

**2. TRACE Integration**
- Every node/card displays confidence score badge
- Click → expand TRACE detail panel (5 dimensions)
- Lineage visualization (how this insight was generated)
- Audit trail export (PDF/DOCX with cryptographic hash)

**3. Export Engine**
- **PDF:** jsPDF + html2canvas (render canvas to image)
- **DOCX:** docx.js (embed images + markdown text)
- **Markdown:** Export as .md with embedded images
- **JSON:** Export canvas state (import/export workspaces)

---

## Implementation Architecture

```
┌─────────────────────────────────────────────────────────┐
│  TRACE Canvas Workspace (Next.js page)                  │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌─────────────────────┐  ┌─────────────────────────┐  │
│  │ Mode Selector       │  │ Panel Toggles           │  │
│  │ ○ Workflow Builder  │  │ □ Tools Library         │  │
│  │ ● Freeform Canvas   │  │ ☑ Lineage Panel         │  │
│  │ ○ Knowledge Graph   │  │ □ Results Panel         │  │
│  └─────────────────────┘  └─────────────────────────┘  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐ │
│  │  Canvas Container (mode-dependent renderer)        │ │
│  │                                                    │ │
│  │  Mode 1: <ReactFlowProvider>                      │ │
│  │            <ReactFlow nodes={} edges={} />        │ │
│  │          </ReactFlowProvider>                     │ │
│  │                                                    │ │
│  │  Mode 2: <Tldraw />                               │ │
│  │                                                    │ │
│  │  Mode 3: <GraphCanvas>                            │ │
│  │            <Reagraph graph={} />                  │ │
│  │          </GraphCanvas>                           │ │
│  │                                                    │ │
│  │  Overlay: <MarkdownCardOverlay position={} />    │ │
│  │                                                    │ │
│  └────────────────────────────────────────────────────┘ │
│                                                          │
│  ┌────────────────────────────────────────────────────┐ │
│  │ Bottom Panel: TRACE Details / Export / Settings   │ │
│  └────────────────────────────────────────────────────┘ │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

**Code splitting:** Each mode lazy-loaded to reduce initial bundle size.

```tsx
// Lazy load modes
const ReactFlowCanvas = lazy(() => import('@/components/canvas/ReactFlowCanvas'))
const TldrawCanvas = lazy(() => import('@/components/canvas/TldrawCanvas'))
const GraphCanvas = lazy(() => import('@/components/canvas/GraphCanvas'))

// Mode switcher
{mode === 'workflow' && <Suspense><ReactFlowCanvas /></Suspense>}
{mode === 'freeform' && <Suspense><TldrawCanvas /></Suspense>}
{mode === 'graph' && <Suspense><GraphCanvas /></Suspense>}
```

---

## Data Model

```typescript
// Canvas document schema (DynamoDB/MongoDB)
interface ICanvasDocument {
  id: string
  userId: string
  title: string
  mode: 'workflow' | 'freeform' | 'knowledge-graph'
  createdAt: string
  updatedAt: string

  // Mode-specific data (only one populated at a time)
  workflowData?: {
    nodes: IReactFlowNode[]
    edges: IReactFlowEdge[]
  }

  freeformData?: {
    tldrawDocument: ITldrawDocument
  }

  graphData?: {
    nodes: IGraphNode[]
    relationships: IGraphRelationship[]
    layout: 'force' | 'hierarchical' | 'circular'
  }

  // Shared across all modes
  markdownCards: IMarkdownCard[]
  traceMetadata: {
    auditHash: string
    lineage: ILineageStep[]
    confidenceScores: Record<string, number>
  }

  // Collaboration (Phase 6)
  collaborators?: string[]
  permissions: 'private' | 'shared' | 'public'
}

// Markdown card (overlays on canvas)
interface IMarkdownCard {
  id: string
  position: { x: number; y: number }
  size: { width: number; height: number }
  content: string // markdown
  wikilinks: string[] // extracted [[links]]
  tags: string[] // extracted #tags
  traceScore?: number
  sourceDocumentId?: string
}

// Workflow node (React Flow)
interface IReactFlowNode {
  id: string
  type: 'input' | 'llm' | 'tool' | 'condition' | 'output' | 'markdown'
  position: { x: number; y: number }
  data: {
    label: string
    config: Record<string, any>
    traceScore?: number
  }
}
```

---

## Consequences

### Positive

1. **Unique market position** — only tool combining canvas + workflows + AI + TRACE explainability
2. **Flexibility** — users choose mode based on task (structured vs freeform)
3. **Best-in-class UX** — each mode uses best library for that use case
4. **Future-proof** — modular architecture allows adding modes (e.g., timeline mode, Gantt mode)
5. **React ecosystem** — all libraries are React-first, easy to integrate
6. **TypeScript safety** — strong typing across all modes

### Negative

1. **Bundle size** — 3 canvas libraries increases bundle (~500KB combined, mitigated by code splitting)
2. **Complexity** — more code to maintain (3 canvas systems + integration layer)
3. **License costs** — Tldraw requires commercial license (~$5K/year estimated)
4. **Learning curve** — users must learn 3 different canvas UIs
5. **Data migration** — converting between modes (workflow → freeform) is complex

### Neutral

1. **No real-time collaboration in Phase 1** — deferred to Phase 6 (Liveblocks or Yjs)
2. **2D-first, 3D optional** — 3D graph requires GPU (auto-fallback to 2D)
3. **HTML overlays, not canvas text** — markdown rendered as HTML, not drawn on canvas

---

## Risks & Mitigation

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| Users confused by multiple modes | Medium | High | Provide mode templates, wizard onboarding |
| Tldraw license cost too high | Low | Medium | Negotiate pricing, or fallback to Excalidraw (MIT) |
| Performance issues with large workflows | Medium | Medium | Implement virtualization, node pagination |
| 3D graph doesn't work on low-end devices | Medium | Low | Auto-detect GPU, fallback to 2D (Cytoscape.js) |
| React Flow too slow for 10K+ nodes | Low | Medium | Consider Konva.js fallback for massive graphs |

---

## Alternatives Considered

### Alternative 1: Single Library (React Flow only)
- **Pro:** Simpler architecture, smaller bundle
- **Con:** No freeform drawing, less flexible
- **Rejected:** Too limiting for diverse use cases

### Alternative 2: Single Library (Tldraw only)
- **Pro:** Best UX, infinite canvas, extensible
- **Con:** Not optimized for structured workflows
- **Rejected:** Workflow builder is core requirement

### Alternative 3: Fabric.js + Custom Workflow Builder
- **Pro:** Full control, no library constraints
- **Con:** 3-6 months development time, high risk
- **Rejected:** Time-to-market too slow

### Alternative 4: No Canvas (Dashboard only)
- **Pro:** Zero development cost
- **Con:** No visual exploration, no differentiation
- **Rejected:** Market gap is too large to ignore

---

## Dependencies

**NPM Packages:**
- `@xyflow/react` (React Flow) — ~130KB
- `tldraw` — ~250KB
- `reagraph` — ~80KB
- `cytoscape` (fallback 2D graph) — ~60KB
- `react-markdown` + `remark-gfm` — ~40KB
- `zustand` (state management) — ~3KB

**Total estimated:** ~560KB (mitigated by code splitting to ~180KB initial load)

**Infrastructure:**
- DynamoDB table: `trace-canvas-documents` (partition key: userId, sort key: documentId)
- S3 bucket: `trace-canvas-exports` (PDF/DOCX/PNG exports)
- CloudFront: Cache static canvas assets

**Future (Phase 6 - Collaboration):**
- Liveblocks or Yjs (CRDT sync engine)
- WebSocket API Gateway (real-time updates)
- ElastiCache Redis (presence/cursors)

---

## Success Metrics

**Adoption (Month 1):**
- 50+ canvas documents created
- 10+ users creating workflows weekly
- 30% of document analyses exported to canvas

**Engagement:**
- Avg 15+ minutes per canvas session
- 3+ canvases per active user
- 20+ nodes per workflow

**Value Delivery:**
- 80% of workflows execute successfully
- 90% of users rate canvas "useful" or "very useful"
- 5+ users migrate from Miro/Obsidian/Litmaps

---

## Implementation Roadmap

**Phase 1: MVP Workflow Builder** (2-3 weeks)
- React Flow integration
- Basic nodes: Input, LLM (Claude/GPT-4/Gemini), Tool, Output
- Execute workflow → display results
- Export to JSON

**Phase 2: Markdown Overlays** (1 week)
- Render markdown cards on canvas
- Wikilink parsing + auto-connectors
- Tag-based filtering
- Markdown export

**Phase 3: Freeform Canvas** (1-2 weeks)
- Tldraw integration
- Mode switcher UI
- Annotation tools
- Image/PDF export

**Phase 4: Knowledge Graph** (1-2 weeks)
- Reagraph 3D integration
- Cytoscape.js 2D fallback
- Node click → TRACE detail panel
- Graph layout algorithms

**Phase 5: Advanced Features** (2 weeks)
- Tool library (drag-drop onto canvas)
- Conditional routing in workflows
- Multi-model swapping (Claude ↔ GPT-4)
- Workflow templates

**Phase 6: Collaboration** (3 weeks, optional)
- Real-time sync (Liveblocks)
- Live cursors and presence
- Conflict resolution
- Permissions (private/shared/public)

**Total: 9-13 weeks**

---

## Review & Approval

**Pending approval from:**
- [ ] Product Lead (strategic fit)
- [ ] Engineering Lead (technical feasibility)
- [ ] UX Designer (mode switcher design)
- [ ] Finance (Tldraw license budget)

**Approval criteria:**
- Strategic alignment with TRACE vision
- Technical feasibility within 3-month timeline
- Budget approval for Tldraw license
- UX approval for mode switcher design

---

## Related Documents

- [Research Report](../../.gcc/branches/research-interactive-canvas/RESEARCH_REPORT.md)
- [ADR-027: Visualization Strategy](./ADR-027-visualization-strategy.md)
- [ADR-023: KG Exploration UI](./ADR-023-kg-exploration-ui.md)
- [ADR-024: Trust by Design](./ADR-024-trust-by-design-ui.md)

---

**Decision Status:** PROPOSED (awaiting approval)
**Next Action:** Present to product team, gather feedback, revise if needed, then approve.
