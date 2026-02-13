# ADR-027: Visualization Strategy for TRACE Canvas

**Date:** 2026-02-12
**Status:** PROPOSED
**Decision Makers:** Claude (Research), Product Team (Approval Pending)
**Related:** ADR-026 (Canvas Technology), ADR-024 (Trust by Design UI)

---

## Context

The TRACE Canvas (ADR-026) provides interactive workspaces for research and compliance analysis. However, canvas alone is insufficient—users need **advanced visualizations** to understand:

1. **TRACE confidence scores** — how trustworthy is each insight?
2. **Knowledge graph relationships** — how are entities connected?
3. **Audit trails** — what steps led to this conclusion?
4. **Compliance risk matrices** — which requirements are high-risk?
5. **Real-time analysis progress** — how far along is document processing?
6. **Multi-dimensional comparisons** — how do 5 TRACE dimensions compare across insights?

**Current State:**
- Basic SVG gauges (circular progress)
- Static knowledge graphs (Neo4j NVL)
- Tables and cards (no interactive charts)
- Mermaid diagrams (static, not data-driven)

**Gap:** No advanced interactive visualizations (radar charts, heatmaps, Sankey diagrams, real-time dashboards).

**Opportunity:** Add best-in-class visualization library to create:
- **Dynamic TRACE dashboards** (live updating as AI processes)
- **Interactive heatmaps** (compliance risk matrices)
- **Radar charts** (5-dimensional TRACE scores)
- **Sankey diagrams** (data flow lineage)
- **Force-directed graphs** (knowledge networks)
- **3D graphs** (large-scale entity visualization)
- **Timeline visualizations** (audit trails)

**Research:** Evaluated 5 visualization libraries:
1. **D3.js** — low-level, powerful, steep learning curve
2. **Visx** (Airbnb) — React-first D3 primitives
3. **Recharts** — declarative, simple, limited customization
4. **ECharts** (Apache) — enterprise-grade, real-time, WebGL
5. **Reagraph** — 3D WebGL graphs

Full research: [.gcc/branches/research-interactive-canvas/RESEARCH_REPORT.md](../../.gcc/branches/research-interactive-canvas/RESEARCH_REPORT.md)

---

## Decision

We will adopt a **multi-library strategy** using:

### 1. Visx (@visx/*) — Primary Custom Visualization Library

**For:** TRACE-specific visualizations requiring custom designs

**Use Cases:**
- **Radar chart** — 5-dimensional TRACE scores (Transparency, Reasoning, Auditability, Compliance, Explainability)
- **Heatmap** — compliance risk matrix (Article × Risk Level)
- **Sankey diagram** — data lineage (Sources → Processing → Outputs)
- **Custom gauges** — confidence score arcs with gradient fills
- **Animated timelines** — audit trail visualization (vertical lineage)
- **Treemap** — hierarchical entity categorization

**Rationale:**
- ✅ **React-first** — composable primitives, no D3 learning curve
- ✅ **Modular** — import only what you need (tree shaking)
- ✅ **TypeScript-native** — full type safety
- ✅ **Airbnb production-tested** — battle-tested at scale
- ✅ **Flexible** — full control without D3 verbosity
- ✅ **SVG-based** — crisp rendering at any zoom level

**Trade-offs:**
- ❌ More code than Recharts for simple charts (acceptable for custom designs)
- ❌ Smaller community than D3 (but excellent docs)

**Example: TRACE Radar Chart**
```tsx
import { Group } from '@visx/group'
import { scaleLinear } from '@visx/scale'
import { Point } from '@visx/point'
import { Line } from '@visx/shape'

const TraceRadarChart = ({ scores }) => {
  const dimensions = ['Transparency', 'Reasoning', 'Auditability', 'Compliance', 'Explainability']
  const radiusScale = scaleLinear({ domain: [0, 100], range: [0, 150] })

  return (
    <svg width={400} height={400}>
      <Group top={200} left={200}>
        {dimensions.map((dim, i) => {
          const angle = (Math.PI * 2 * i) / 5 - Math.PI / 2
          const point = new Point({ angle, radius: radiusScale(scores[dim]) })
          return <circle key={dim} cx={point.x} cy={point.y} r={4} fill="#3b82f6" />
        })}
      </Group>
    </svg>
  )
}
```

---

### 2. ECharts (Apache) — Real-Time & Large-Scale Dashboards

**For:** High-performance visualizations with millions of data points

**Use Cases:**
- **Real-time TRACE monitoring** — live confidence score updates as AI processes documents
- **Large-scale knowledge graphs** (10K+ nodes) — WebGL rendering
- **3D visualizations** — 3D scatter plots, globe maps
- **Calendar heatmaps** — compliance review activity over time
- **Parallel coordinates** — multi-dimensional entity comparison
- **Graph charts** — complex network visualizations (complement to Reagraph)

**Rationale:**
- ✅ **Best performance** — handles millions of data points with WebGL
- ✅ **Real-time updates** — streaming data support
- ✅ **Advanced chart types** — 3D, globe, calendar, Sankey, parallel
- ✅ **Rich interactions** — brush, zoom, drill-down, data region select
- ✅ **Export built-in** — PNG/SVG/PDF download
- ✅ **Cross-platform** — web, mobile, Node.js server-side rendering

**Trade-offs:**
- ❌ **Larger bundle** (~300KB min+gzip, mitigated by lazy loading)
- ❌ **Imperative API** (less React-friendly, use `echarts-for-react` wrapper)
- ❌ Configuration-heavy (verbose JSON configs)

**Example: Real-Time TRACE Gauge**
```tsx
import ReactECharts from 'echarts-for-react'

const RealTimeTraceGauge = ({ confidenceScore }) => {
  const option = {
    series: [{
      type: 'gauge',
      startAngle: 180,
      endAngle: 0,
      min: 0,
      max: 100,
      progress: { show: true, width: 18 },
      axisLine: { lineStyle: { width: 18 } },
      data: [{ value: confidenceScore, name: 'Confidence' }],
      detail: {
        valueAnimation: true,
        formatter: '{value}%',
        color: confidenceScore >= 85 ? '#10b981' : confidenceScore >= 70 ? '#3b82f6' : '#f59e0b'
      }
    }]
  }

  return <ReactECharts option={option} style={{ height: 300 }} />
}
```

---

### 3. Reagraph — 3D WebGL Knowledge Graphs

**For:** Large-scale interactive knowledge graph visualization

**Use Cases:**
- **3D citation networks** — papers connected by references
- **Entity relationship exploration** — 10K+ entities with force-directed layout
- **Knowledge graph navigation** — zoom/pan/rotate to explore connections
- **TRACE knowledge maps** — insights → entities → sources in 3D space

**Rationale:**
- ✅ **Unique differentiator** — no competitor offers 3D graph visualization
- ✅ **WebGL-powered** — handles 10K+ nodes smoothly
- ✅ **Interactive** — zoom, pan, rotate, node click
- ✅ **React-native** — seamless integration
- ✅ **Force-directed layout** — auto-positioning
- ✅ **Customizable nodes** — render custom React components

**Trade-offs:**
- ❌ **Requires GPU** — fallback to Cytoscape.js 2D on low-end devices
- ❌ Smaller community (newer library)

**Example: 3D Knowledge Graph**
```tsx
import { GraphCanvas } from 'reagraph'

const KnowledgeGraph3D = ({ nodes, edges }) => {
  return (
    <GraphCanvas
      nodes={nodes.map(n => ({
        id: n.id,
        label: n.label,
        size: n.importance * 2,
        fill: n.type === 'RISK' ? '#ef4444' : '#3b82f6'
      }))}
      edges={edges.map(e => ({
        id: e.id,
        source: e.from,
        target: e.to,
        label: e.relationship
      }))}
      layoutType="forceDirected3d"
      onNodeClick={(node) => showTraceDetail(node.id)}
    />
  )
}
```

---

### 4. Mermaid — Declarative Diagrams (Already Implemented)

**For:** Static workflow diagrams, sequence diagrams, Gantt charts

**Use Cases:**
- **Sequence diagrams** — show API call sequences
- **Flowcharts** — visualize decision trees
- **Gantt charts** — project timelines
- **ER diagrams** — database schemas

**Status:** Already integrated (see ADR-024, feature-markdown-viz)

**Keep:** Continue using for static diagrams in markdown.

---

### 5. Recharts — Simple Charts (Prototyping Only)

**For:** Quick prototypes and internal dashboards

**Use Cases:**
- **Bar charts** — simple comparisons
- **Line charts** — trend analysis
- **Pie charts** — category distribution

**Rationale:**
- ✅ **Fastest development** for standard charts
- ✅ Declarative JSX API
- ✅ Good for MVPs and internal tools

**Trade-offs:**
- ❌ **Limited customization** (use Visx for production)
- ❌ **Performance issues** with >10K data points

**Decision:** Use Recharts only for internal dashboards. Use Visx for user-facing TRACE visualizations.

---

## Visualization Inventory

### TRACE-Specific Visualizations

| Visualization | Library | Use Case | Priority |
|---------------|---------|----------|----------|
| **Radar Chart** | Visx | 5-dimensional TRACE scores | P0 (MVP) |
| **Confidence Gauge** | ECharts | Real-time confidence updates | P0 (MVP) |
| **Heatmap** | Visx | Compliance risk matrix | P1 (Phase 2) |
| **Sankey Diagram** | Visx | Data lineage (sources → outputs) | P1 (Phase 2) |
| **Timeline** | Visx | Audit trail visualization | P0 (MVP) |
| **3D Graph** | Reagraph | Large knowledge graphs | P1 (Phase 2) |
| **Force Graph (2D)** | ECharts | Citation networks | P1 (Phase 2) |
| **Treemap** | Visx | Hierarchical entity categorization | P2 (Phase 3) |
| **Parallel Coordinates** | ECharts | Multi-entity comparison | P2 (Phase 3) |
| **Calendar Heatmap** | ECharts | Compliance activity over time | P2 (Phase 3) |
| **Mermaid Diagrams** | Mermaid | Static flowcharts, sequence diagrams | P0 (Existing) |

---

## Implementation Architecture

### Component Structure

```
src/components/visualizations/
├── trace/
│   ├── TraceRadarChart.tsx          # Visx — 5-dimensional scores
│   ├── ConfidenceGaugeRealTime.tsx  # ECharts — animated gauge
│   ├── AuditTimeline.tsx            # Visx — vertical lineage
│   ├── ComplianceHeatmap.tsx        # Visx — risk matrix
│   ├── DataLineageSankey.tsx        # Visx — Sankey diagram
│   └── TrustScoreCard.tsx           # Visx — custom SVG gauge
├── graphs/
│   ├── KnowledgeGraph3D.tsx         # Reagraph — 3D WebGL
│   ├── KnowledgeGraph2D.tsx         # ECharts — 2D fallback
│   ├── CitationNetwork.tsx          # ECharts — force-directed
│   └── EntityTreemap.tsx            # Visx — hierarchical
├── shared/
│   ├── ChartContainer.tsx           # Wrapper with loading/error states
│   ├── ChartLegend.tsx              # Reusable legend component
│   ├── ChartTooltip.tsx             # Visx tooltip provider
│   └── ExportButton.tsx             # PNG/SVG export utility
└── utils/
    ├── colorScales.ts               # Consistent color palettes
    ├── formatters.ts                # Number/date formatters
    └── responsive.ts                # Responsive sizing helpers
```

### Color Palette (Consistent Across All Charts)

```typescript
// src/components/visualizations/utils/colorScales.ts
export const TRACE_COLORS = {
  // Confidence tiers (5-tier EU system)
  confidence: {
    veryHigh: '#10b981',  // GREEN 85-100%
    high: '#3b82f6',      // BLUE 70-84%
    medium: '#f59e0b',    // ORANGE 50-69%
    low: '#ef4444',       // RED 30-49%
    veryLow: '#f43f5e'    // MAROON 0-29%
  },
  // TRACE dimensions
  trace: {
    transparency: '#8b5cf6',    // PURPLE
    reasoning: '#3b82f6',       // BLUE
    auditability: '#10b981',    // GREEN
    compliance: '#f59e0b',      // ORANGE
    explainability: '#ec4899'   // PINK
  },
  // Entity types
  entities: {
    risk: '#ef4444',       // RED
    entity: '#3b82f6',     // BLUE
    policy: '#10b981',     // GREEN
    person: '#f59e0b',     // ORANGE
    document: '#8b5cf6'    // PURPLE
  },
  // Status
  status: {
    success: '#10b981',
    warning: '#f59e0b',
    error: '#ef4444',
    info: '#3b82f6'
  }
}
```

---

## Responsive Design Strategy

All visualizations must be responsive (mobile, tablet, desktop).

### Breakpoint Strategy

```typescript
const useResponsiveChart = () => {
  const width = useWindowWidth()

  return {
    chartWidth: width < 640 ? width - 32 : width < 1024 ? 600 : 800,
    fontSize: width < 640 ? 10 : 12,
    margin: width < 640 ? { top: 10, right: 10, bottom: 30, left: 30 } : { top: 20, right: 40, bottom: 40, left: 60 }
  }
}
```

### Mobile-First Rules

1. **Simplify on mobile** — reduce data density (show top 10 items instead of 100)
2. **Touch-friendly** — min 44px touch targets
3. **Horizontal scroll** — allow horizontal scroll for wide charts
4. **Collapsible legends** — hide legend on mobile, show on click
5. **Tooltip on tap** — not hover (no hover on touch devices)

---

## Performance Optimization

### Bundle Size Strategy

| Library | Size (min+gzip) | Load Strategy |
|---------|-----------------|---------------|
| Visx (core + shape + scale) | ~40KB | Eager (critical path) |
| ECharts | ~300KB | Lazy load on demand |
| Reagraph | ~80KB | Lazy load on demand |
| Mermaid | ~150KB | Lazy load on markdown parse |

**Code splitting example:**
```tsx
// Lazy load ECharts only when needed
const RealTimeGauge = lazy(() => import('./visualizations/trace/ConfidenceGaugeRealTime'))

// In component
{showRealTimeGauge && (
  <Suspense fallback={<ChartSkeleton />}>
    <RealTimeGauge confidenceScore={score} />
  </Suspense>
)}
```

### Rendering Optimization

1. **Memoization** — memoize chart components (React.memo)
2. **Throttle updates** — throttle real-time updates to 60fps
3. **Virtualization** — render only visible chart elements (large datasets)
4. **WebGL fallback** — auto-detect GPU, fallback to Canvas/SVG if WebGL unsupported

---

## Export Capabilities

All visualizations must support export to:

1. **PNG** — raster image (high DPI, 2x resolution)
2. **SVG** — vector image (scalable, editable in Illustrator)
3. **PDF** — embedded in regulatory reports
4. **JSON** — data export for analysis in Excel/Python

**Implementation:**
```typescript
// src/components/visualizations/shared/ExportButton.tsx
export const ExportButton = ({ chartRef, chartData, format }) => {
  const exportChart = async () => {
    if (format === 'png') {
      const canvas = await html2canvas(chartRef.current)
      const link = document.createElement('a')
      link.download = 'trace-chart.png'
      link.href = canvas.toDataURL()
      link.click()
    } else if (format === 'svg') {
      // ECharts and Visx support native SVG export
      const svg = chartRef.current.querySelector('svg')
      const serializer = new XMLSerializer()
      const svgBlob = new Blob([serializer.serializeToString(svg)], { type: 'image/svg+xml' })
      saveAs(svgBlob, 'trace-chart.svg')
    } else if (format === 'json') {
      const dataBlob = new Blob([JSON.stringify(chartData, null, 2)], { type: 'application/json' })
      saveAs(dataBlob, 'trace-data.json')
    }
  }

  return <Button onClick={exportChart}>Export {format.toUpperCase()}</Button>
}
```

---

## Accessibility (WCAG 2.1 AA)

All visualizations must be accessible:

1. **Color contrast** — 4.5:1 for text, 3:1 for UI components
2. **Colorblind-safe palettes** — use patterns/textures in addition to color
3. **Keyboard navigation** — arrow keys to navigate data points
4. **Screen reader support** — ARIA labels, data tables for screen readers
5. **Focus indicators** — visible focus rings on interactive elements

**Example:**
```tsx
<svg aria-label="TRACE confidence radar chart">
  <desc>
    This radar chart shows 5 dimensions of TRACE scores: Transparency 85%,
    Reasoning 72%, Auditability 90%, Compliance 68%, Explainability 78%.
  </desc>
  {/* Chart elements */}
</svg>
```

---

## Testing Strategy

### Unit Tests (Jest + React Testing Library)

```typescript
// TraceRadarChart.test.tsx
describe('TraceRadarChart', () => {
  it('renders all 5 TRACE dimensions', () => {
    const scores = { transparency: 85, reasoning: 72, auditability: 90, compliance: 68, explainability: 78 }
    render(<TraceRadarChart scores={scores} />)
    expect(screen.getByText(/Transparency/)).toBeInTheDocument()
    expect(screen.getByText(/Reasoning/)).toBeInTheDocument()
    // ... etc
  })

  it('applies correct color for high confidence', () => {
    const scores = { transparency: 92, reasoning: 88, auditability: 95, compliance: 87, explainability: 90 }
    render(<TraceRadarChart scores={scores} />)
    const circles = screen.getAllByRole('graphics-symbol')
    expect(circles[0]).toHaveAttribute('fill', TRACE_COLORS.confidence.veryHigh)
  })
})
```

### Visual Regression Tests (Chromatic or Percy)

- Capture screenshots of all chart types
- Detect visual regressions on color/layout changes
- Test across breakpoints (mobile, tablet, desktop)

### Performance Tests

```typescript
// Measure render time for large datasets
it('renders 1000-node knowledge graph in <2 seconds', async () => {
  const nodes = generateNodes(1000)
  const edges = generateEdges(nodes, 5000)

  const start = performance.now()
  render(<KnowledgeGraph3D nodes={nodes} edges={edges} />)
  await waitFor(() => screen.getByRole('graphics-document'))
  const end = performance.now()

  expect(end - start).toBeLessThan(2000)
})
```

---

## Consequences

### Positive

1. **Best-in-class visualizations** — Visx + ECharts cover all use cases
2. **Performance optimized** — ECharts handles millions of data points, WebGL for 3D
3. **Unique differentiator** — 3D knowledge graphs (Reagraph) not offered by competitors
4. **React ecosystem** — all libraries integrate seamlessly with Next.js
5. **Future-proof** — modular approach allows swapping libraries if needed
6. **Accessibility** — WCAG 2.1 AA compliant out of the box

### Negative

1. **Bundle size** — ~420KB combined (mitigated by lazy loading to ~40KB initial)
2. **Learning curve** — team must learn Visx and ECharts APIs
3. **Maintenance** — 4 visualization libraries to keep updated
4. **Testing complexity** — visual regression tests required for all charts

### Neutral

1. **No D3.js directly** — use Visx instead (lower learning curve)
2. **Recharts only for prototypes** — not production-facing
3. **Mermaid for static diagrams** — keep existing implementation

---

## Risks & Mitigation

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| ECharts bundle too large | Medium | Medium | Lazy load, only load chart types used |
| Visx has breaking changes | Low | Medium | Pin major version, test upgrades thoroughly |
| 3D graphs don't work on low-end devices | Medium | Low | Auto-detect GPU, fallback to 2D (ECharts/Cytoscape.js) |
| Charts not accessible | Low | High | WCAG audit, screen reader testing, ARIA labels |
| Real-time updates cause performance issues | Medium | Medium | Throttle to 60fps, use requestAnimationFrame |

---

## Alternatives Considered

### Alternative 1: D3.js Only
- **Pro:** Most powerful, full control
- **Con:** Steep learning curve, slow development, not React-friendly
- **Rejected:** Visx provides 90% of D3 power with 50% of complexity

### Alternative 2: Recharts Only
- **Pro:** Simplest API, fastest development
- **Con:** Limited customization, poor performance with large data
- **Rejected:** Insufficient for TRACE-specific visualizations

### Alternative 3: Chart.js
- **Pro:** Simple, popular, good performance
- **Con:** Imperative API (not React-friendly), limited chart types
- **Rejected:** Visx is more React-native

### Alternative 4: Victory (Formidable)
- **Pro:** React-first, declarative, modular
- **Con:** Smaller community than Recharts, less performance than ECharts
- **Rejected:** Visx has better DX and Airbnb backing

---

## Dependencies

**NPM Packages (Production):**
```json
{
  "@visx/group": "^3.3.0",
  "@visx/scale": "^3.5.0",
  "@visx/shape": "^3.5.0",
  "@visx/axis": "^3.10.1",
  "@visx/grid": "^3.5.0",
  "@visx/legend": "^3.5.0",
  "@visx/tooltip": "^3.3.0",
  "echarts": "^5.5.0",
  "echarts-for-react": "^3.0.2",
  "reagraph": "^4.15.0",
  "mermaid": "^10.8.0"
}
```

**Total bundle impact:**
- Eager load (critical): ~40KB (Visx core)
- Lazy load (on-demand): ~380KB (ECharts + Reagraph)

---

## Success Metrics

**Adoption (Month 1):**
- 100+ TRACE radar charts generated
- 50+ real-time gauges rendered
- 20+ knowledge graphs explored (3D mode)

**Performance:**
- <100ms render time for standard charts (radar, gauge)
- <2s render time for 1K-node knowledge graph
- 60fps real-time updates (confidence gauge)

**User Feedback:**
- 85% find visualizations "helpful" or "very helpful"
- 90% prefer new visualizations over old SVG gauges
- 70% use export feature (PNG/SVG)

---

## Implementation Roadmap

**Phase 1: Core TRACE Visualizations** (1-2 weeks)
- Visx radar chart (5-dimensional TRACE scores)
- Visx timeline (audit trail)
- ECharts confidence gauge (real-time animated)

**Phase 2: Knowledge Graphs** (1 week)
- Reagraph 3D graph integration
- ECharts 2D fallback
- GPU detection + auto-fallback

**Phase 3: Advanced Charts** (1-2 weeks)
- Visx heatmap (compliance risk matrix)
- Visx Sankey (data lineage)
- ECharts calendar heatmap (activity over time)

**Phase 4: Export & Accessibility** (1 week)
- PNG/SVG/JSON export for all charts
- WCAG 2.1 AA audit
- Screen reader testing

**Total: 4-6 weeks**

---

## Review & Approval

**Pending approval from:**
- [ ] Product Lead (strategic fit)
- [ ] Engineering Lead (technical feasibility)
- [ ] UX Designer (chart design language)
- [ ] Accessibility Lead (WCAG compliance)

**Approval criteria:**
- Visualization designs align with brand
- Performance targets achievable
- Accessibility requirements met
- Budget approved (no additional license costs)

---

## Related Documents

- [ADR-026: Canvas Technology](./ADR-026-interactive-canvas-technology.md)
- [ADR-024: Trust by Design UI](./ADR-024-trust-by-design-ui.md)
- [Research Report](../../.gcc/branches/research-interactive-canvas/RESEARCH_REPORT.md)

---

**Decision Status:** PROPOSED (awaiting approval)
**Next Action:** Create chart design mockups, present to UX/product team, gather feedback.
