# Interactive Canvas Research Report
**Branch:** research-interactive-canvas
**Date:** 2026-02-12
**Researcher:** Claude (GCC Session)

---

## Executive Summary

This report evaluates technologies for adding an **interactive canvas/board system** (Miro-like) to the CrawlQ TRACE platform. The goal is to create a unique research workspace that enables:
- **Visual TRACE analysis** with knowledge graphs on infinite canvas
- **Workflow builder** with multi-model pipelines and tool connectors
- **Markdown document rendering** on canvas with interlinkages
- **Advanced visualizations** (dynamic charts, interactive diagrams, Mermaid+)
- **Unique value** for PhD research, B2B analysis, compliance auditing, cascade impact analysis

**Key Recommendation:** Hybrid architecture using **React Flow** (workflow builder) + **Tldraw** (freeform canvas) + **Visx** (advanced charts) + **Reagraph** (3D knowledge graphs).

---

## 1. Canvas Technology Evaluation

### 1.1 React Flow (Xyflow)

**Overview:** Node-based UI library for React. React Flow has rebranded as part of Xyflow ecosystem.

**Strengths:**
- ✅ **Purpose-built for workflow builders** with drag-drop, connectors, node/edge customization
- ✅ **Production-ready** — used by Stripe, Typeform, Instagram
- ✅ **React-first** — nodes are React components (flexible but less performant than canvas)
- ✅ **Rich plugin ecosystem** — minimap, controls, background grid
- ✅ **TypeScript-native** with excellent documentation
- ✅ **Handles multi-selection**, keyboard shortcuts, undo/redo
- ✅ **Commercial license available** (watermark-free)

**Weaknesses:**
- ❌ Trades performance for flexibility (DOM-based, not canvas)
- ❌ Not ideal for freeform drawing/whiteboarding
- ❌ Limited to node-edge paradigm (less flexible than true canvas)

**Use Cases in TRACE:**
- Multi-model workflow builder (LLM → RAG → Validator → Output)
- Research pipeline designer (Search → Extract → Analyze → Report)
- TRACE audit trail visualization (data lineage as connected nodes)

**Performance:** Handles 1000+ nodes/edges but can lag with very complex graphs.

**Sources:**
- [React Flow Official Site](https://reactflow.dev/)
- [Xyflow GitHub](https://github.com/xyflow/xyflow)
- [Xyflow Overview](https://xyflow.com/)

---

### 1.2 Tldraw

**Overview:** Infinite canvas SDK for whiteboards and freeform drawing in React.

**Strengths:**
- ✅ **Infinite canvas** with pan/zoom (best-in-class UX)
- ✅ **Freeform drawing** tools (pen, shapes, arrows, text, sticky notes)
- ✅ **Real-time collaboration** with live cursors and viewport following
- ✅ **Production-ready UI** library (toolbars, menus, OpenGL minimap)
- ✅ **Extensible** — can embed custom React components
- ✅ **TypeScript-native** with 44K+ GitHub stars
- ✅ **Custom shapes/tools** can be added
- ✅ **Image/PDF export** built-in

**Weaknesses:**
- ❌ **Watermark required** ("Made with tldraw") unless commercial license
- ❌ Less suited for structured workflows (better for freeform)
- ❌ Not optimized for large knowledge graphs (better for sketching)

**Use Cases in TRACE:**
- Freeform research canvas (mind maps, concept sketches)
- Annotating document insights visually
- Collaborative whiteboarding for compliance review sessions
- Sticky note style insight exploration

**Performance:** OpenGL-powered, handles large canvases smoothly.

**Sources:**
- [Tldraw Official Site](https://tldraw.dev/)
- [Tldraw GitHub](https://github.com/tldraw/tldraw)
- [Tldraw SDK 4.2 Release](https://tldraw.dev/blog/tldraw-4-2-release)

---

### 1.3 Excalidraw

**Overview:** Open-source virtual whiteboard with hand-drawn style aesthetic.

**Strengths:**
- ✅ **Hand-drawn aesthetic** (unique, friendly, less corporate)
- ✅ **Infinite canvas** with core shapes (rectangles, ellipses, diamonds, arrows, lines)
- ✅ **Real-time collaboration** built-in
- ✅ **Export to PNG/SVG/JSON**
- ✅ **Fully open source** (MIT license)
- ✅ **Dark mode**, i18n, undo/redo
- ✅ **Arrow binding** to connect objects
- ✅ **React integration** via `@excalidraw/excalidraw` package

**Weaknesses:**
- ❌ Hand-drawn style may not suit enterprise/compliance contexts
- ❌ Limited to basic shapes (no advanced chart types)
- ❌ Less extensible than Tldraw or React Flow
- ❌ No built-in workflow builder features

**Use Cases in TRACE:**
- Casual brainstorming/wireframing
- Less formal research sessions
- Educational/onboarding materials

**Verdict:** Good for casual use, but Tldraw is superior for our needs (more extensible, better UX).

**Sources:**
- [Excalidraw GitHub](https://github.com/excalidraw/excalidraw)
- [Excalidraw Integration Docs](https://docs.excalidraw.com/docs/@excalidraw/excalidraw/integration)

---

### 1.4 Fabric.js

**Overview:** Low-level HTML5 canvas library with object manipulation focus.

**Strengths:**
- ✅ **Object-oriented canvas API** (treat shapes as objects)
- ✅ **SVG import/export** (canvas ↔ SVG conversion)
- ✅ **Rich filters and effects** (blur, brightness, color matrix)
- ✅ **Text editing** on canvas with full typography control
- ✅ **Image manipulation** (crop, resize, filters)
- ✅ **Event-driven** (click, drag, rotate, scale)

**Weaknesses:**
- ❌ **Manual memory management** required (prone to leaks in long-running apps)
- ❌ **No React integration** (vanilla JS, requires wrapper)
- ❌ **Lower performance** than Konva for large scenes
- ❌ **No SVG export** from canvas (only SVG import)
- ❌ Flat object model (less organized than scene graph)

**Use Cases in TRACE:**
- Custom drawing tools (annotation layers over documents)
- Image manipulation (PDF page markup)

**Verdict:** Useful as low-level tool but not ideal for primary canvas. Consider for specific use cases only.

**Sources:**
- [Fabric.js vs Konva Comparison](https://stackshare.io/stackups/fabricjs-vs-konva)
- [Konva vs Fabric Technical Analysis](https://medium.com/@www.blog4j.com/konva-js-vs-fabric-js-in-depth-technical-comparison-and-use-case-analysis-9c247968dd0f)

---

### 1.5 Konva.js

**Overview:** High-performance 2D canvas library with layered scene graph.

**Strengths:**
- ✅ **High performance** with dirty region detection (only redraws changed areas)
- ✅ **Layer-based rendering** (static background cached, dynamic elements updated)
- ✅ **Scene graph architecture** (organized hierarchy of objects)
- ✅ **Proactive memory management** (automatic cleanup of orphaned objects)
- ✅ **React integration** via `react-konva`
- ✅ **Touch-friendly** with mobile optimization
- ✅ **Filters/effects** built-in (blur, shadow, HSL, etc.)

**Weaknesses:**
- ❌ **No SVG export** (canvas-only)
- ❌ Steeper learning curve than React Flow
- ❌ Lower-level than React Flow (more manual setup)

**Use Cases in TRACE:**
- Large-scale knowledge graph rendering (10K+ nodes)
- Custom interactive visualizations requiring high FPS
- Mobile-optimized canvas experiences

**Verdict:** Excellent for performance-critical canvas work. Good complement to React Flow.

**Sources:**
- [Konva Official Site](https://konvajs.org/)
- [Konva vs Fabric Comparison](https://www.oreateai.com/blog/konvajs-vs-fabricjs-choosing-your-canvas-companion/9d255e8dbd093ab89c868295b2d20187)

---

## 2. Visualization Library Evaluation

### 2.1 D3.js

**Overview:** Low-level data visualization library for DOM manipulation.

**Strengths:**
- ✅ **Most powerful** and flexible charting library
- ✅ **Fine-grained control** over every visual element
- ✅ **Large ecosystem** (15+ years, massive community)
- ✅ **Custom visualizations** not possible with other libraries
- ✅ **Data binding** to DOM with transitions

**Weaknesses:**
- ❌ **Steep learning curve** (imperative API, not declarative)
- ❌ **Verbose code** for simple charts (100+ lines for basic bar chart)
- ❌ **Not React-friendly** (DOM manipulation conflicts with React)
- ❌ **Slow development** time

**Use Cases in TRACE:**
- Highly custom visualizations (force-directed graphs, Sankey diagrams)
- One-off specialized charts not available in other libraries

**Verdict:** Use sparingly via wrapper libraries (Visx, Recharts). Avoid direct D3 usage.

**Sources:**
- [D3.js vs Recharts Comparison](https://medium.com/@ebojacky/javascript-data-visualization-d3-js-vs-react-recharts-c64cff3642b1)
- [React Charts Built on D3](https://medium.com/react-courses/react-charts-built-on-d3-what-should-you-pick-rechart-visx-niv-react-vi-or-victory-adc64406caa1)

---

### 2.2 Visx (Airbnb)

**Overview:** Low-level React visualization components built on D3 primitives.

**Strengths:**
- ✅ **Modular** — pick only components you need (tree shaking)
- ✅ **React-first** — no D3 learning curve
- ✅ **Composable primitives** (Axis, Grid, Shape, Scale, Gradient)
- ✅ **Full control** without D3 verbosity
- ✅ **TypeScript** support
- ✅ **Used by Airbnb** in production

**Weaknesses:**
- ❌ **Lower-level than Recharts** (more code for simple charts)
- ❌ Smaller community than D3 or Recharts
- ❌ Less documentation than Recharts

**Use Cases in TRACE:**
- Complex custom charts (radar, heatmap, treemap)
- TRACE confidence gauge (custom SVG arcs with D3 scales)
- Animated lineage timelines
- Multi-axis compliance dashboards

**Verdict:** **Best choice for custom TRACE visualizations** — balance of power and React idioms.

**Sources:**
- [Visx GitHub](https://github.com/airbnb/visx)
- [Best React Chart Libraries 2025](https://blog.logrocket.com/best-react-chart-libraries-2025/)

---

### 2.3 Recharts

**Overview:** Declarative React charting library built on D3.

**Strengths:**
- ✅ **Simplest API** — declarative JSX syntax
- ✅ **Rapid development** for standard chart types
- ✅ **24.8K GitHub stars** with excellent docs
- ✅ **Responsive** out of the box
- ✅ **Gentle learning curve**
- ✅ **Good for dashboards** with standard charts

**Weaknesses:**
- ❌ **Less customizable** than Visx or D3
- ❌ **Performance issues** with large datasets (>10K points)
- ❌ Limited chart types (no radar, treemap, Sankey)

**Use Cases in TRACE:**
- Quick prototypes and MVP dashboards
- Standard charts (line, bar, area, pie)
- Internal admin dashboards

**Verdict:** Use for simple charts, but Visx is better for production TRACE visualizations.

**Sources:**
- [Recharts vs D3 Comparison](https://solutions.lykdat.com/blog/recharts-vs-d3-js/)
- [React Chart Libraries Comparison](https://npm-compare.com/@visx/visx,chart.js,d3,react-vis,recharts)

---

### 2.4 ECharts (Apache)

**Overview:** Enterprise-grade charting library with advanced features.

**Strengths:**
- ✅ **Best performance** — handles millions of data points in real time
- ✅ **Advanced chart types** (3D, globe, calendar heatmap, Sankey, graph)
- ✅ **Rich interactions** (brush, data zoom, drill-down)
- ✅ **WebGL rendering** for large datasets
- ✅ **Cross-platform** (web, mobile, Node.js)
- ✅ **Export to PNG/SVG/PDF**

**Weaknesses:**
- ❌ **Not React-native** (requires wrapper like `echarts-for-react`)
- ❌ **Larger bundle size** (~300KB min+gzip)
- ❌ Imperative API (less React-friendly)

**Use Cases in TRACE:**
- Real-time TRACE monitoring dashboards (live updates)
- Large-scale data visualizations (10K+ entities)
- 3D knowledge graph visualizations
- Heatmaps for compliance risk matrices

**Verdict:** **Excellent for large-scale/real-time visualizations**. Complement to Visx.

**Sources:**
- [Best React Chart Libraries](https://embeddable.com/blog/react-chart-libraries)
- [JavaScript Chart Libraries 2026](https://www.luzmo.com/blog/best-javascript-chart-libraries)

---

## 3. Knowledge Graph Visualization

### Current Solution: Neo4j NVL
- Works well for basic graph exploration
- Limited customization and styling
- No 3D or advanced layout algorithms

### Recommended Alternatives/Complements

**3.1 Reagraph (WebGL 3D)**
- WebGL-powered graph visualization for React
- **3D force-directed layouts** with smooth animations
- Handles large graphs (10K+ nodes) smoothly
- Customizable nodes/edges with React components
- **Unique differentiator** — no competitors offer this in research tools

**Sources:**
- [Reagraph GitHub](https://github.com/reaviz/reagraph)
- [Knowledge Graph Visualization Libraries](https://www.getfocal.co/post/top-10-javascript-libraries-for-knowledge-graph-visualization)

**3.2 Cytoscape.js**
- Graph theory library with 100+ layout algorithms
- Excellent for complex network analysis
- Supports graph algorithms (shortest path, centrality, clustering)
- Can run analyses in browser

**Sources:**
- [Cytoscape.js Official Site](https://js.cytoscape.org/)

**3.3 Sigma.js**
- WebGL-based graph rendering (faster than SVG/Canvas)
- React integration via `@react-sigma`
- Good for large graphs (100K+ edges)
- Less feature-rich than Cytoscape but faster

**Sources:**
- [Sigma.js Official Site](https://www.sigmajs.org/)

---

## 4. Markdown Rendering on Canvas

### Challenge
Standard markdown renderers (remark, MDX) output DOM/React elements, not canvas.

### Solutions

**4.1 HTML-to-Canvas (html2canvas, dom-to-image)**
- Render markdown to hidden DOM, convert to canvas image
- ❌ Not truly interactive (just an image)
- ❌ Poor text rendering quality
- ✅ Simple implementation

**4.2 Custom Canvas Markdown Renderer**
- Parse markdown AST (unified, remark-parse)
- Render text nodes with Canvas 2D API
- Handle headings, lists, code blocks manually
- ❌ High implementation cost (2-3 weeks)
- ✅ Full control over rendering

**4.3 Hybrid Approach (Recommended)**
- **Render markdown as HTML overlay on canvas**
- Position HTML elements using absolute positioning
- Canvas for connectors/background, HTML for markdown content
- ✅ Best text quality (native HTML)
- ✅ Interactive (links, code copy buttons)
- ✅ Fast development

**Implementation:**
```tsx
<div className="canvas-container">
  <Canvas /> {/* Tldraw or React Flow */}
  <MarkdownOverlay position={{x, y}}>
    <ReactMarkdown>{content}</ReactMarkdown>
  </MarkdownOverlay>
</div>
```

**Sources:**
- [Canvas Markdown Editor](https://nyxtom.dev/2020/08/25/canvas-markdown-editor)
- [WYSIWYG Canvas Markdown](https://github.com/markboard-io/wysiwyg-canvas-markdown)

---

## 5. Academic Research Tool Analysis

### Key Findings from 2026 Landscape

**5.1 Citation Network Visualization**
- **Litmaps**: Interactive visualizations of research trends and citation networks
- Shows influential papers, research lineages, emerging trends
- Citation network analysis is now **standard in serious research tools**

**5.2 AI-Powered Literature Review**
- **Connected Papers**: Maps research fields for dissertation work
- **Elicit**: Extracts key findings from hundreds of papers
- PhD students use these tools to build comprehensive literature reviews

**5.3 Data Analysis & Compliance**
- **Julius**: Natural language data exploration
- **NVivo, ATLAS.ti**: Structured features aligned with academic standards
- **Privacy compliance** is critical — tools must be transparent about data usage

**Gap in Market:**
**No tool combines:**
1. ✅ Knowledge graph visualization
2. ✅ TRACE explainability (audit trails)
3. ✅ Workflow builder (research pipelines)
4. ✅ Compliance checking (GDPR, EU AI Act)
5. ✅ Interactive canvas workspace

**This is our unique differentiator.**

**Sources:**
- [Best AI Tools for Academic Research 2026](https://cognitivefuture.ai/best-ai-tools-for-academic-research/)
- [AI Tools for Scientific Literature Review](https://www.cypris.ai/insights/11-best-ai-tools-for-scientific-literature-review-in-2026)
- [Research Data Visualization Tools](https://peerrecognized.com/dataviz/)

---

## 6. Multi-Modal Workflow Builder Analysis

### Key Platforms in 2026

**6.1 Visual Programming Platforms**
- **Dify**: Open-source LLMOps with visual workflow builder
- **Flowise**: Drag-drop AI agent builder on LangChain
- **Rivet**: Visual AI programming by Ironclad
- **StackAI**: Clean visual builder with document-driven workflows

**6.2 Multi-Modal Capabilities**
- LangChain supports GPT-4o, Gemini 1.5, Claude 3 (text/image/audio)
- Workflow builders now handle **multimodal pipelines** (PDF → Vision LLM → Text Analysis)
- **Agent orchestration** is core infrastructure in 2026

**6.3 Key Features for TRACE**
- **Visual node-based editor** (React Flow powers most of them)
- **Connector logic** with conditional routing
- **Model swapping** (swap Claude ↔ GPT-4 ↔ Gemini mid-workflow)
- **Tool integration** (web search, vector DB, calculators)
- **Observability** (trace execution, debug failed nodes)

**Gap in Market:**
Most workflow builders are **developer-focused**. None are designed for **research analysts** or **compliance officers** who need:
- Domain-specific nodes (GDPR checker, citation extractor)
- Explainability at each step (TRACE protocol)
- Compliance audit trails
- Export to regulatory reports (PDF/DOCX with lineage)

**This is our opportunity.**

**Sources:**
- [Top 7 Multimodal AI Agent Platforms](https://www.creolestudios.com/top-platforms-to-build-multimodal-ai-agents/)
- [Best AI Workflow Builders 2026](https://emergent.sh/learn/best-ai-workflow-builders)
- [AI Agent Tools Landscape 2026](https://www.stackone.com/blog/ai-agent-tools-landscape-2026)

---

## 7. Unique Value Propositions

### 7.1 For PhD Researchers

**Problem:** PhD students spend weeks manually building literature review networks, extracting insights from papers, and tracking citations. Tools like Litmaps solve visualization but lack **workflow automation** and **AI-powered extraction**.

**Our Solution: TRACE Research Canvas**
- Upload 100 PDFs → Automatic citation network extraction
- **Visual canvas** with papers as nodes, citations as edges
- Click paper → AI extracts key findings with TRACE confidence scores
- **Workflow builder** for custom research pipelines:
  - Example: PDF → Extract Methodology → Compare Against Baseline → Highlight Gaps
- **Lineage tracking** — know which AI model extracted which insight
- **Export** to LaTeX, BibTeX, or Obsidian markdown

**Unique Value:** Only tool that combines citation networks + AI extraction + workflow automation + explainability.

---

### 7.2 For B2B Research Analysts

**Problem:** Market research analysts gather data from 20+ sources (reports, news, LinkedIn, earnings calls) and synthesize insights. Tools like Miro help with whiteboarding but lack **AI integration** and **structured workflows**.

**Our Solution: TRACE Intelligence Canvas**
- **Multi-source ingestion** (PDF, DOCX, web scrape, API)
- **Canvas workspace** for visual synthesis (sticky notes, mindmaps, connectors)
- **AI-powered clustering** — auto-group related insights by theme
- **TRACE confidence scoring** — flag low-confidence insights for human review
- **Workflow builder** for recurring research processes:
  - Example: Weekly Competitor Analysis → Scrape News → Extract Positioning → Compare vs Last Week → Flag Changes
- **Export** to PowerPoint with TRACE audit trail (for client transparency)

**Unique Value:** Only tool that combines freeform canvas + structured workflows + AI confidence scoring + client-ready exports.

---

### 7.3 For Compliance Officers

**Problem:** GDPR/AI Act compliance reviews involve reading 50+ page documents, extracting requirements, mapping to company processes, and tracking cascade impacts. Tools like Confluence are static; tools like Miro lack compliance domain knowledge.

**Our Solution: TRACE Compliance Canvas**
- Upload regulatory document (GDPR, DPIA template, AI Act)
- **Auto-extract** requirements with article/section references
- **Canvas view** with requirements as nodes
- **Impact analysis** — map requirements to affected business processes
- **Cascade visualization** — see how one requirement impacts 10 downstream systems
- **TRACE audit trail** — prove to auditors how you identified each requirement
- **Workflow builder** for compliance processes:
  - Example: New Regulation → Extract Requirements → Map to Processes → Assign Owners → Track Remediation
- **Export** to regulatory report (PDF/DOCX with lineage and evidence)

**Unique Value:** Only tool with **domain-specific compliance intelligence** + visual impact analysis + audit trails.

---

### 7.4 For Cross-Functional Teams (Cascade Impact Analysis)

**Problem:** Product changes can have unexpected impacts across engineering, legal, marketing, sales. No tool visualizes cascade impacts with AI-powered risk scoring.

**Our Solution: TRACE Impact Canvas**
- Define change (e.g., "Add email export feature")
- AI analyzes impacts across:
  - Legal (GDPR data export requirements)
  - Engineering (3 affected systems)
  - Marketing (new messaging needed)
  - Support (new FAQ items)
- **Canvas view** with change at center, impacts radiating outward
- **TRACE confidence scores** for each impact (high = certain, low = speculative)
- **Interactive exploration** — click impact → see reasoning and evidence
- **Workflow builder** for change management:
  - Example: Proposed Change → AI Impact Analysis → Stakeholder Review → Approval → Execution

**Unique Value:** Only tool with **AI-powered cascade analysis** + visual impact mapping + confidence scoring.

---

## 8. Advanced Capabilities Roadmap

### 8.1 Interactive Panels

**Concept:** Multi-panel canvas workspace (similar to Obsidian Canvas but with AI).

**Panels:**
1. **Input Panel** — upload documents, enter queries
2. **Model Selector** — choose LLM (Claude, GPT-4, Gemini)
3. **Tool Library** — drag tools onto canvas (web search, calculator, PDF extractor)
4. **Canvas Panel** — main workspace with nodes, connectors, markdown overlays
5. **Results Panel** — AI outputs with TRACE confidence scores
6. **Lineage Panel** — trace execution path
7. **Export Panel** — download as PDF/DOCX/MD

**Implementation:**
- React Flow for workflow canvas
- Tldraw for freeform annotation layer
- Drag-drop from tool library to canvas
- Connectors auto-update on node repositioning

---

### 8.2 Dynamic Charts & Visualizations

**Capabilities:**
- **Real-time confidence gauge** (animates as AI processes)
- **Heatmap** for compliance risk matrices (Article 22 high-risk areas)
- **Radar chart** for TRACE scores (Transparency, Reasoning, Auditability, Compliance, Explainability)
- **Sankey diagram** for data flow (Sources → Processing → Outputs)
- **Force-directed graph** for knowledge networks (auto-layout with physics)
- **3D graph** (Reagraph) for large knowledge bases (toggle 2D ↔ 3D)
- **Timeline** for audit trails (vertical lineage)

**Libraries:**
- Visx for custom SVG charts (radar, heatmap)
- ECharts for real-time dashboards (live updates)
- Reagraph for 3D knowledge graphs
- Mermaid for sequence/flow diagrams (already implemented)

---

### 8.3 Markdown Interlinkages on Canvas

**Concept:** Render .md files as cards on canvas with clickable links between them.

**Features:**
- **Wikilinks** — `[[Related Document]]` auto-creates connector
- **Hover preview** — hover link → see preview tooltip
- **Bi-directional links** — see backlinks panel
- **Tag-based clustering** — auto-group cards by `#tags`
- **Search & filter** — filter canvas by tag, date, confidence
- **Collapsible sections** — expand/collapse headings
- **Code execution** — run code blocks inline (Python, JS)

**Implementation:**
- Tldraw for canvas
- Custom "MarkdownCard" shape
- Remark plugins for wikilinks and tag parsing
- D3 force layout for auto-positioning related cards

---

### 8.4 Algorithmic Differentiators

**8.4.1 Auto-Layout Algorithms**
- **Force-directed** — physics simulation for knowledge graphs
- **Hierarchical** — top-down for audit trails
- **Circular** — for cyclic dependencies
- **Treemap** — for nested categories
- **Timeline** — for temporal data

**8.4.2 Intelligent Clustering**
- **Topic modeling** (LDA, BERTopic) — auto-group insights by theme
- **Semantic similarity** — cluster based on embedding distance
- **Citation network clustering** — identify research communities

**8.4.3 Anomaly Detection**
- **Outlier insights** — flag insights that don't fit established patterns
- **Confidence anomalies** — warn if AI is uncertain about important claims
- **Bias detection** — flag potential bias in source material

**8.4.4 Recommendation Engine**
- **Next best action** — suggest next step in workflow
- **Related insights** — "Users who analyzed this also analyzed..."
- **Gap analysis** — identify missing evidence for claims

---

## 9. Competitive Landscape

| Feature | CrawlQ TRACE Canvas | Litmaps | Miro | Obsidian Canvas | n8n (workflow) |
|---------|---------------------|---------|------|-----------------|----------------|
| Visual canvas | ✅ | ✅ | ✅ | ✅ | ❌ |
| Knowledge graph | ✅ 3D WebGL | ✅ 2D | ❌ | ❌ | ❌ |
| AI extraction | ✅ Multi-model | ❌ | ❌ | ❌ | ⚠️ Limited |
| Workflow builder | ✅ | ❌ | ❌ | ❌ | ✅ Dev-focused |
| TRACE explainability | ✅ | ❌ | ❌ | ❌ | ❌ |
| Compliance domain | ✅ GDPR/AI Act | ❌ | ❌ | ❌ | ❌ |
| Audit trails | ✅ Cryptographic | ❌ | ❌ | ❌ | ⚠️ Basic logs |
| Citation networks | ✅ | ✅ | ❌ | ❌ | ❌ |
| Markdown rendering | ✅ On canvas | ❌ | ⚠️ Text only | ✅ | ❌ |
| Export to reports | ✅ PDF/DOCX/MD | ⚠️ Image | ⚠️ Image | ✅ MD | ❌ |
| Real-time collab | 🔜 Roadmap | ✅ | ✅ | ❌ | ❌ |
| Target user | Researchers, compliance | Academia | Designers | Note-takers | Developers |

**Conclusion:** No competitor offers all features. CrawlQ TRACE Canvas is **category-defining**.

---

## 10. Technical Architecture Recommendation

### 10.1 Hybrid Canvas System

**Rationale:** No single library solves all use cases. Use best tool for each job.

**Architecture:**

```
┌─────────────────────────────────────────────────────────┐
│  TRACE Canvas Workspace                                 │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌──────────────────────────┐  ┌────────────────────┐  │
│  │  Mode Selector           │  │  Panel Toggles     │  │
│  │  ○ Workflow Builder      │  │  □ Tools Library   │  │
│  │  ● Freeform Canvas       │  │  ☑ Lineage Panel   │  │
│  │  ○ Knowledge Graph       │  │  □ Results Panel   │  │
│  └──────────────────────────┘  └────────────────────┘  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐ │
│  │                                                    │ │
│  │         Canvas Area (mode-dependent renderer)     │ │
│  │                                                    │ │
│  │  [React Flow] ──OR── [Tldraw] ──OR── [Reagraph]  │ │
│  │                                                    │ │
│  │         Markdown Overlays (HTML positioned)       │ │
│  │                                                    │ │
│  └────────────────────────────────────────────────────┘ │
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │  Visualization Layer (Visx/ECharts)              │  │
│  │  [Confidence Gauge] [Radar Chart] [Heatmap]      │  │
│  └──────────────────────────────────────────────────┘  │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

**Mode 1: Workflow Builder**
- **Engine:** React Flow
- **Use case:** Research pipelines, multi-model orchestration
- **Nodes:** Input, LLM, Tool, Condition, Output
- **Connectors:** Data flow with conditional routing

**Mode 2: Freeform Canvas**
- **Engine:** Tldraw
- **Use case:** Brainstorming, mind maps, annotation
- **Tools:** Pen, shapes, sticky notes, text, arrows
- **Collaboration:** Real-time cursors (future)

**Mode 3: Knowledge Graph**
- **Engine:** Reagraph (3D WebGL) or Cytoscape.js (2D with algorithms)
- **Use case:** Visualize entity relationships, citation networks
- **Interactions:** Zoom, pan, rotate (3D), node click → detail panel
- **Algorithms:** Force-directed, hierarchical, circular

**Shared Components:**
- **Markdown overlay system** (HTML positioned above canvas)
- **TRACE panel** (confidence scores, lineage)
- **Export engine** (PDF/DOCX/MD with embedded images)
- **Tool library** (drag-drop onto canvas)

---

### 10.2 Data Model

```typescript
// Canvas Document (MongoDB/DynamoDB)
interface ICanvasDocument {
  id: string
  title: string
  mode: 'workflow' | 'freeform' | 'knowledge-graph'
  createdAt: string
  updatedAt: string

  // React Flow data (if mode = workflow)
  workflowData?: {
    nodes: IWorkflowNode[]
    edges: IWorkflowEdge[]
  }

  // Tldraw data (if mode = freeform)
  freeformData?: {
    shapes: ITldrawShape[]
    bindings: ITldrawBinding[]
  }

  // Graph data (if mode = knowledge-graph)
  graphData?: {
    nodes: IGraphNode[]
    relationships: IGraphRel[]
  }

  // Markdown overlays (all modes)
  markdownCards: IMarkdownCard[]

  // TRACE metadata
  traceData: {
    auditHash: string
    lineage: ILineageStep[]
    confidenceScores: Record<string, number>
  }
}

// Workflow Node
interface IWorkflowNode {
  id: string
  type: 'input' | 'llm' | 'tool' | 'condition' | 'output' | 'markdown'
  position: { x: number; y: number }
  data: {
    label: string
    config: Record<string, any> // type-specific config
  }
}

// Markdown Card (overlays)
interface IMarkdownCard {
  id: string
  position: { x: number; y: number }
  size: { width: number; height: number }
  content: string // markdown text
  wikilinks: string[] // [[linked documents]]
  tags: string[] // #tags
  traceScore?: number
}
```

---

### 10.3 Implementation Phases

**Phase 1: MVP Workflow Builder (2-3 weeks)**
- React Flow integration
- Basic nodes: Input, LLM, Output
- Connector logic (data passes from node to node)
- Execute workflow → display results
- Export to JSON

**Phase 2: Markdown Overlays (1 week)**
- Render markdown cards on React Flow canvas
- Wikilink parsing and connector creation
- Tag-based filtering
- Markdown export

**Phase 3: Freeform Canvas (1-2 weeks)**
- Tldraw integration
- Mode switcher (workflow ↔ freeform)
- Annotation layer over documents
- Image/PDF export

**Phase 4: Knowledge Graph (1-2 weeks)**
- Reagraph 3D integration
- Graph layout algorithms (Cytoscape.js)
- Node click → TRACE detail panel
- 2D ↔ 3D toggle

**Phase 5: Advanced Visualizations (1-2 weeks)**
- Visx charts (radar, heatmap, Sankey)
- ECharts real-time dashboards
- Dynamic confidence gauges
- Interactive timelines

**Phase 6: Collaboration (2-3 weeks)**
- WebSocket for real-time sync
- Live cursors (Tldraw supports this)
- Conflict resolution
- Presence indicators

**Total: 8-13 weeks for full feature set**

---

## 11. Recommended Tech Stack

| Layer | Technology | Rationale |
|-------|-----------|-----------|
| **Workflow Canvas** | React Flow (@xyflow/react) | Industry standard, production-ready, TypeScript-native |
| **Freeform Canvas** | Tldraw | Best UX, infinite canvas, extensible, real-time ready |
| **Knowledge Graph (3D)** | Reagraph | Unique 3D WebGL, no competitors offer this |
| **Knowledge Graph (2D)** | Cytoscape.js | 100+ layout algorithms, graph analysis built-in |
| **Custom Charts** | Visx (@visx/*) | React-first, composable, TypeScript, moderate learning curve |
| **Real-Time Charts** | ECharts (echarts-for-react) | Best performance, millions of data points |
| **Markdown Rendering** | React-Markdown + remark-gfm | Standard, extensible with plugins |
| **Markdown Parsing** | unified + remark-parse | Extract AST for wikilinks, tags |
| **Diagram Rendering** | Mermaid (already integrated) | Sequence, flowchart, Gantt |
| **Export PDF** | jsPDF + html2canvas | Generate PDFs in browser |
| **Export DOCX** | docx.js | Generate Word docs with images |
| **State Management** | Zustand | Lightweight, simpler than Redux |
| **Real-Time Sync** | Liveblocks or Yjs | CRDT-based collaboration (Phase 6) |

---

## 12. Risk Analysis

### 12.1 Technical Risks

**Risk:** Multiple canvas libraries increase bundle size.
**Mitigation:** Code splitting, lazy load modes (load React Flow only when in workflow mode).

**Risk:** Markdown rendering on canvas has poor text quality.
**Mitigation:** Use HTML overlays, not canvas text rendering.

**Risk:** 3D graph (Reagraph) may not work on low-end devices.
**Mitigation:** Provide 2D fallback, detect GPU capability.

**Risk:** Real-time collaboration is complex (CRDT, conflict resolution).
**Mitigation:** Phase 6 (optional), use proven library (Liveblocks, Yjs).

---

### 12.2 Business Risks

**Risk:** Users don't understand workflow builder (too complex).
**Mitigation:** Provide templates (pre-built workflows), wizard mode.

**Risk:** Freeform canvas doesn't add value over Miro.
**Mitigation:** Differentiate with AI integration, TRACE scoring on sticky notes.

**Risk:** PhD researchers prefer Obsidian Canvas (offline, local-first).
**Mitigation:** Offer local-first mode (IndexedDB), offline sync.

---

## 13. Success Metrics

**Adoption:**
- 50+ canvas documents created in first month
- 10+ users using workflow builder weekly
- 5+ users switching from Miro/Obsidian

**Engagement:**
- Avg 15+ minutes per canvas session
- 3+ canvases per user
- 20+ nodes per workflow

**Value Delivery:**
- 80% of workflows execute successfully
- 90% of users find TRACE scores helpful
- 70% of users export canvases to reports

---

## 14. Conclusion & Next Steps

### Key Decisions

1. **Canvas Technology:** Hybrid approach (React Flow + Tldraw + Reagraph)
2. **Visualization:** Visx for custom charts, ECharts for real-time
3. **Markdown:** HTML overlays with absolute positioning
4. **Unique Value:** Only tool combining canvas + workflows + AI + TRACE + compliance

### Immediate Next Steps

1. ✅ **Create ADR-026:** Canvas technology selection
2. ✅ **Create ADR-027:** Visualization strategy
3. ✅ **Design prototype:** Workflow builder mockup (Figma or code)
4. ✅ **Spike:** React Flow integration (1 day proof-of-concept)
5. ✅ **Implementation roadmap:** Phased rollout plan

### Long-Term Vision

TRACE Canvas becomes the **default workspace for AI-powered research**, replacing:
- Miro (for researchers)
- Obsidian Canvas (for knowledge workers)
- n8n (for non-developer workflow automation)
- Litmaps (for citation networks)

**Market positioning:** "The only research workspace with AI explainability built-in."

---

## Sources

### Canvas Technologies
- [React Flow Official Site](https://reactflow.dev/)
- [Xyflow GitHub](https://github.com/xyflow/xyflow)
- [Tldraw Official Site](https://tldraw.dev/)
- [Tldraw GitHub](https://github.com/tldraw/tldraw)
- [Excalidraw GitHub](https://github.com/excalidraw/excalidraw)
- [Fabric.js vs Konva Comparison](https://stackshare.io/stackups/fabricjs-vs-konva)
- [Konva Technical Analysis](https://medium.com/@www.blog4j.com/konva-js-vs-fabric-js-in-depth-technical-comparison-and-use-case-analysis-9c247968dd0f)

### Visualization Libraries
- [D3.js vs Recharts](https://medium.com/@ebojacky/javascript-data-visualization-d3-js-vs-react-recharts-c64cff3642b1)
- [Best React Chart Libraries 2025](https://blog.logrocket.com/best-react-chart-libraries-2025/)
- [React Chart Libraries Comparison](https://npm-compare.com/@visx/visx,chart.js,d3,react-vis,recharts)
- [JavaScript Chart Libraries 2026](https://www.luzmo.com/blog/best-javascript-chart-libraries)

### Knowledge Graphs
- [Reagraph GitHub](https://github.com/reaviz/reagraph)
- [Cytoscape.js](https://js.cytoscape.org/)
- [Sigma.js](https://www.sigmajs.org/)
- [Knowledge Graph Visualization Libraries](https://www.getfocal.co/post/top-10-javascript-libraries-for-knowledge-graph-visualization)

### Markdown & Canvas
- [Canvas Markdown Editor](https://nyxtom.dev/2020/08/25/canvas-markdown-editor)
- [WYSIWYG Canvas Markdown](https://github.com/markboard-io/wysiwyg-canvas-markdown)

### Academic Research Tools
- [Best AI Tools for Academic Research 2026](https://cognitivefuture.ai/best-ai-tools-for-academic-research/)
- [AI Tools for Scientific Literature Review](https://www.cypris.ai/insights/11-best-ai-tools-for-scientific-literature-review-in-2026)
- [Research Data Visualization Tools](https://peerrecognized.com/dataviz/)

### Workflow Builders
- [Top 7 Multimodal AI Agent Platforms](https://www.creolestudios.com/top-platforms-to-build-multimodal-ai-agents/)
- [Best AI Workflow Builders 2026](https://emergent.sh/learn/best-ai-workflow-builders)
- [AI Agent Tools Landscape 2026](https://www.stackone.com/blog/ai-agent-tools-landscape-2026)
- [Top 5 Drag-and-Drop Libraries for React](https://puckeditor.com/blog/top-5-drag-and-drop-libraries-for-react)

---

**End of Research Report**
