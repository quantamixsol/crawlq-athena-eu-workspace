# ADR-028: Constitutional Non-Breaking Implementation Principles

**Date:** 2026-02-12
**Status:** CONSTITUTIONAL (Cannot be violated)
**Decision Makers:** Product Leadership + Engineering
**Related:** ADR-026 (Canvas Technology), ADR-027 (Visualization Strategy)

---

## Context

TRACE Canvas is a major new feature (12-week implementation, hybrid architecture with React Flow + Tldraw + Reagraph 3D). The existing TRACE system is production-critical with:
- 87% E2E test confidence (ADR-025)
- Complex document analysis pipeline (6-stage deep research)
- Existing knowledge graph visualization (Neo4j NVL)
- 13 advanced markdown capabilities
- Guest-to-auth conversion flows
- Export to PDF/DOCX/MD

**Risk:** Adding canvas could break existing functionality, degrade performance, or create regressions.

**Requirement:** Build TRACE Canvas as **additive-only** feature with **zero breaking changes** to existing system.

---

## Decision

This ADR establishes **constitutional principles** that **cannot be violated** during TRACE Canvas implementation. These principles are mandatory and override all other considerations.

---

## Constitutional Principles (MANDATORY)

### Principle 1: Non-Breaking Mandate

**Rule:** No modifications to existing TRACE components unless explicitly extending them with **optional** canvas features.

**Protected Components:**
- `TraceDashboard` (5-panel TRACE UI)
- `TraceKnowledgeGraph` (Neo4j NVL interactive graph)
- `DocumentAnalysisPanel` (insight cards, risk analysis)
- `ScoreCard` (circular gauge)
- `InsightCard` (expandable TRACE details)
- `Analytics` (main guest document analysis page)
- `ChatTraceCard` (EU chat TRACE display)
- `TraceExplainabilityPanelEU` (7-section EU explainability)

**Allowed Extensions:**
- Add "Open in Canvas" button to `DocumentAnalysisPanel` (new button, doesn't change existing UI)
- Add "View in 3D Graph" button to `TraceKnowledgeGraph` (optional enhancement)
- Add canvas export option to existing export menus (additive, not replacement)

**Forbidden:**
- ❌ Changing existing component props (breaking change)
- ❌ Replacing existing visualizations (must coexist)
- ❌ Modifying existing routes (e.g., `/guest-document-analysis/*`)
- ❌ Changing existing DynamoDB schemas (must create new tables)
- ❌ Modifying existing Lambda functions (must create new ones or add optional params)

---

### Principle 2: Isolated Routes

**Rule:** All canvas features live under `/canvas/*` routes.

**New Routes (Allowed):**
- `/canvas` — Canvas workspace listing page
- `/canvas/new` — Create new canvas
- `/canvas/[id]` — Edit existing canvas
- `/canvas/[id]/share` — Share canvas (if collaboration feature)
- `/canvas/templates` — Canvas templates library

**Existing Routes (Protected — No Changes):**
- `/guest-document-analysis/*` — Guest flow (unchanged)
- `/chat-athena-eu/*` — Chat interface (unchanged)
- `/` — Landing page (may add "Try Canvas" CTA, but existing CTAs stay)

**Routing Strategy:**
```tsx
// src/app/canvas/layout.tsx — New layout for canvas routes
export default function CanvasLayout({ children }) {
  return (
    <CanvasProvider>
      <CanvasNav />
      {children}
    </CanvasProvider>
  )
}
```

---

### Principle 3: Isolated Components

**Rule:** All canvas components in `src/components/canvas/*` directory, no mixing with existing components.

**Directory Structure:**
```
src/components/
├── canvas/                           ← NEW (canvas-only)
│   ├── CanvasContainer.tsx
│   ├── ModeSelector.tsx
│   ├── workflow/
│   │   ├── WorkflowCanvas.tsx
│   │   ├── nodes/
│   │   │   ├── InputNode.tsx
│   │   │   ├── LLMNode.tsx
│   │   │   ├── ToolNode.tsx
│   │   │   └── OutputNode.tsx
│   │   └── ToolLibrary.tsx
│   ├── freeform/
│   │   ├── FreeformCanvas.tsx
│   │   └── AutoCluster.tsx
│   ├── graph/
│   │   ├── KnowledgeGraph3D.tsx
│   │   ├── KnowledgeGraph2D.tsx
│   │   └── GraphControls.tsx
│   ├── markdown/
│   │   ├── MarkdownCard.tsx
│   │   ├── WikilinkParser.tsx
│   │   └── TagFilter.tsx
│   ├── visualizations/              ← NEW (canvas-specific charts)
│   │   ├── TraceRadarChart.tsx      (Visx)
│   │   ├── ConfidenceGaugeRealTime.tsx (ECharts)
│   │   ├── ComplianceHeatmap.tsx
│   │   └── DataLineageSankey.tsx
│   └── export/
│       ├── ExportModal.tsx
│       ├── PDFExporter.tsx
│       └── DOCXExporter.tsx
├── trace-eu/                         ← EXISTING (protected)
│   ├── trace-explainability-eu/
│   ├── knowledge-graph-eu/
│   └── document-analysis-eu/
└── chat-eu/                          ← EXISTING (protected)
    ├── ChatContainer.tsx
    └── ...
```

**Shared Utilities (Allowed):**
- Can import from `src/lib/*` (formatters, API clients, auth)
- Can import existing types from `src/types/*`
- Can use existing hooks from `src/hooks/*`

**Forbidden:**
- ❌ Importing canvas components into existing TRACE/chat components (creates coupling)
- ❌ Modifying existing components to support canvas (violates isolation)

---

### Principle 4: Shared Resources (Non-Destructive)

**Rule:** Use existing infrastructure but add new resources as needed without modifying existing ones.

**Shared (Allowed):**
- **DynamoDB:** Use existing connection client, but create **new table** `trace-canvas-documents`
- **S3:** Use existing bucket `crawlq-eu-documents`, but create **new prefix** `canvas/exports/`
- **Lambda:** Call existing chat/analysis Lambdas via API Gateway, but create **new Lambdas** for canvas-specific operations
- **Auth:** Use existing Cognito integration (no changes to auth flow)
- **API Gateway:** Add **new routes** under `/canvas/*` endpoints

**New Resources (Required):**
```yaml
DynamoDB:
  - trace-canvas-documents:           # NEW table
      PK: userId
      SK: canvasId
      GSI: canvasId-index (for sharing)

Lambda:
  - canvas-execute-workflow:          # NEW function
      Purpose: Execute workflow nodes
  - canvas-save-document:             # NEW function
      Purpose: Save canvas to DynamoDB
  - canvas-load-document:             # NEW function
      Purpose: Load canvas from DynamoDB
  - canvas-export-pdf:                # NEW function
      Purpose: Generate PDF export
  - canvas-ai-cluster:                # NEW function
      Purpose: AI clustering for freeform mode

S3 Prefixes:
  - canvas/exports/                   # NEW prefix
  - canvas/templates/                 # NEW prefix
```

**Forbidden:**
- ❌ Modifying existing DynamoDB tables (e.g., adding canvas fields to `trace-documents`)
- ❌ Changing existing Lambda function signatures
- ❌ Modifying existing S3 bucket policies (unless adding canvas-specific permissions)

---

### Principle 5: Backward Compatibility

**Rule:** Existing document analysis flow continues to work exactly as before; canvas is additive.

**Existing Flow (Protected):**
```
Guest uploads PDF
  → Lambda: upload-deep-document
  → DynamoDB: trace-documents
  → Frontend: /guest-document-analysis/[id]
  → Display: TraceDashboard + InsightCards + KnowledgeGraph
```

**New Flow (Additive):**
```
User clicks "Open in Canvas" button (NEW)
  → Route: /canvas/new?documentId=[id]
  → Load existing TRACE data (read-only)
  → Display: Canvas with markdown cards + knowledge graph
  → Save: New canvas document (separate from original TRACE doc)
```

**Key:**
- Original TRACE document **unchanged**
- Canvas creates **new document** (linked via `sourceDocumentId` field)
- User can view both: original TRACE UI **and** canvas (two separate views)

**Forbidden:**
- ❌ Replacing TRACE dashboard with canvas (must coexist)
- ❌ Forcing users to use canvas (must be optional)
- ❌ Breaking existing guest document upload flow

---

### Principle 6: Feature-Flagged Rollout

**Rule:** Canvas can be disabled via feature flag if issues arise.

**Feature Flag:**
```typescript
// src/config/feature-flags.ts
export const FEATURE_FLAGS = {
  ENABLE_TRACE_CANVAS: process.env.NEXT_PUBLIC_ENABLE_CANVAS === 'true',
  // ... existing flags
}
```

**Usage:**
```tsx
// Conditionally show "Open in Canvas" button
{FEATURE_FLAGS.ENABLE_TRACE_CANVAS && (
  <Button onClick={openInCanvas}>Open in Canvas</Button>
)}

// Protect canvas routes
export default function CanvasPage() {
  if (!FEATURE_FLAGS.ENABLE_TRACE_CANVAS) {
    return <ComingSoon message="Canvas feature launching soon!" />
  }
  return <CanvasContainer />
}
```

**Rollout Strategy:**
1. **Week 3 (Private Beta):** `ENABLE_CANVAS=true` for 10 beta users only (env var override)
2. **Week 6 (Public Beta):** `ENABLE_CANVAS=true` for all users, but feature announced as "beta"
3. **Week 9 (GA):** Feature fully released, flag remains for kill-switch capability

**Forbidden:**
- ❌ Removing feature flag before 3 months post-GA (must keep kill-switch)
- ❌ Tightly coupling canvas to existing code (makes disabling impossible)

---

### Principle 7: Testing Isolation

**Rule:** All canvas tests in `__tests__/canvas/*`, no interference with existing tests.

**Test Directory Structure:**
```
__tests__/
├── canvas/                          ← NEW (canvas-only)
│   ├── workflow/
│   │   ├── WorkflowCanvas.test.tsx
│   │   └── WorkflowExecutor.test.ts
│   ├── freeform/
│   │   └── FreeformCanvas.test.tsx
│   ├── graph/
│   │   └── KnowledgeGraph3D.test.tsx
│   ├── visualizations/
│   │   ├── TraceRadarChart.test.tsx
│   │   └── ConfidenceGaugeRealTime.test.tsx
│   └── integration/
│       └── canvas-e2e.test.tsx
├── trace-eu/                        ← EXISTING (protected)
│   └── ...
└── chat-eu/                         ← EXISTING (protected)
    └── ...
```

**Test Isolation:**
- Canvas tests run in separate Jest suite (can be run independently)
- No shared test fixtures between canvas and existing tests
- Canvas E2E tests use separate test data (no pollution of existing test DB)

**Coverage Requirements:**
- Canvas components: 80%+ coverage
- Workflow execution: 90%+ coverage (critical path)
- Export engine: 85%+ coverage

**Forbidden:**
- ❌ Modifying existing test setup to accommodate canvas
- ❌ Sharing mocks between canvas and existing tests (creates coupling)
- ❌ Breaking existing tests (all existing tests must continue to pass)

---

## Enforcement Mechanisms

### 1. Pre-Commit Hooks (Git)

```bash
# .husky/pre-commit
#!/bin/sh

# Check for forbidden imports (canvas importing from trace-eu)
if git diff --cached --name-only | grep 'src/components/canvas'; then
  # Run ESLint rule: no-restricted-imports
  npx eslint --rule 'no-restricted-imports: ["error", {"patterns": ["../trace-eu/*", "../chat-eu/*"]}]' src/components/canvas
fi

# Check for modifications to protected files
PROTECTED_FILES=(
  "src/components/trace-eu/trace-explainability-eu/TraceExplainabilityPanelEU.tsx"
  "src/components/trace-eu/knowledge-graph-eu/KnowledgeGraphPanelEU.tsx"
  "src/app/(components)/guest-document-analysis/analytics/page.tsx"
)

for file in "${PROTECTED_FILES[@]}"; do
  if git diff --cached --name-only | grep -q "$file"; then
    echo "❌ ERROR: Modification to protected file: $file"
    echo "   ADR-028 Principle 1: Non-Breaking Mandate violated."
    echo "   Canvas features must be additive-only."
    exit 1
  fi
done
```

---

### 2. ESLint Rules

```javascript
// .eslintrc.js
module.exports = {
  rules: {
    // Prevent canvas components from importing existing TRACE components
    'no-restricted-imports': ['error', {
      patterns: [
        {
          group: ['../trace-eu/*', '../chat-eu/*'],
          message: 'Canvas components cannot import from trace-eu or chat-eu (ADR-028 Principle 3)',
        },
      ],
    }],
  },
}
```

---

### 3. CI/CD Checks (GitHub Actions)

```yaml
# .github/workflows/canvas-adr-compliance.yml
name: ADR-028 Compliance Check

on: [pull_request]

jobs:
  check-compliance:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Check for protected file modifications
        run: |
          PROTECTED_FILES=(
            "src/components/trace-eu/**/*"
            "src/app/(components)/guest-document-analysis/**/*"
          )

          for pattern in "${PROTECTED_FILES[@]}"; do
            if git diff origin/main --name-only | grep -E "$pattern"; then
              echo "❌ Protected files modified. Review ADR-028 compliance."
              exit 1
            fi
          done

      - name: Check canvas route isolation
        run: |
          # Ensure all canvas routes are under /canvas/*
          if grep -r "const.*route.*=.*'/canvas'" src/app --exclude-dir=canvas; then
            echo "❌ Canvas routes found outside /canvas directory."
            exit 1
          fi

      - name: Run canvas tests in isolation
        run: |
          npm test -- __tests__/canvas
```

---

### 4. Code Review Checklist

Every PR touching canvas code must pass this checklist:

- [ ] No modifications to protected components (`trace-eu/*`, `chat-eu/*`)
- [ ] All new routes under `/canvas/*`
- [ ] All new components in `src/components/canvas/*`
- [ ] No imports from `trace-eu` or `chat-eu` into canvas components
- [ ] Feature flag `ENABLE_TRACE_CANVAS` used for conditional rendering
- [ ] New DynamoDB table/S3 prefix created (not modifying existing)
- [ ] Tests in `__tests__/canvas/*` (not mixed with existing tests)
- [ ] Existing tests still pass (run `npm test` before PR)
- [ ] No breaking changes to existing API endpoints

---

## Consequences

### Positive

1. **Zero Risk to Production:** Existing TRACE system cannot break (isolated architecture)
2. **Parallel Development:** Canvas team can work independently without blocking existing features
3. **Incremental Rollout:** Feature flag allows safe beta testing and gradual rollout
4. **Easy Rollback:** If canvas has issues, disable via feature flag (no code rollback needed)
5. **Clear Boundaries:** Developers know exactly what's protected vs what can be changed
6. **Testability:** Canvas tests isolated, no risk of breaking existing test suite

### Negative

1. **Some Duplication:** Cannot reuse existing components directly (must create canvas-specific versions)
2. **Complexity:** Maintaining two parallel systems (TRACE dashboard + canvas) increases codebase size
3. **Learning Curve:** Developers must understand isolation boundaries
4. **Extra Infrastructure:** New DynamoDB table, S3 prefix, Lambda functions (small cost increase)

### Neutral

1. **Longer Initial Setup:** Week 1 includes infrastructure setup (DynamoDB, Lambda, feature flags)
2. **Stricter Code Review:** All PRs must pass ADR-028 compliance checklist
3. **Documentation Overhead:** Must document "why canvas is separate" for future developers

---

## Violations & Penalties

**If ADR-028 is violated:**

1. **Pre-commit hook fails** → Developer cannot commit code
2. **CI/CD fails** → PR cannot be merged
3. **Code review rejection** → Reviewer must reject PR with reference to violated principle

**Intentional Violation Process (Emergency Only):**
1. Create ADR amendment proposal
2. Get approval from Product Lead + Engineering Lead
3. Document why violation is necessary and impact assessment
4. Implement with extra testing and monitoring

**Violation Examples:**
- ❌ Modifying `TraceDashboard` to embed canvas → **REJECTED** (violates Principle 1)
- ❌ Adding canvas fields to existing `trace-documents` DynamoDB table → **REJECTED** (violates Principle 4)
- ❌ Changing `/guest-document-analysis/*` routes → **REJECTED** (violates Principle 2)
- ✅ Adding "Open in Canvas" button to `DocumentAnalysisPanel` (via prop) → **ALLOWED** (additive extension)

---

## Review & Monitoring

**Weekly Reviews (During Implementation):**
- Every Friday: Review all canvas PRs merged that week
- Check for ADR-028 compliance violations
- Measure: number of protected files modified (target: 0)

**Post-GA Monitoring:**
- Track feature flag usage (% users with canvas enabled)
- Monitor error rates (canvas errors should not affect existing TRACE)
- Track rollback events (if feature flag disabled, why?)

**Quarterly Review:**
- Assess whether isolation strategy is working
- Consider consolidation opportunities (after 6+ months of stable canvas usage)
- Update ADR if architectural improvements identified

---

## Related Documents

- [ADR-026: Canvas Technology](./ADR-026-interactive-canvas-technology.md) — Hybrid architecture (React Flow + Tldraw + Reagraph)
- [ADR-027: Visualization Strategy](./ADR-027-visualization-strategy.md) — Visx + ECharts + Reagraph visualization stack
- [Research Report](../.gcc/branches/research-interactive-canvas/RESEARCH_REPORT.md) — Comprehensive canvas research
- [Feature Specification](../.gcc/branches/research-interactive-canvas/TRACE_CANVAS_SPEC.md) — Complete canvas feature spec
- [Implementation Roadmap](../.gcc/branches/research-interactive-canvas/IMPLEMENTATION_ROADMAP.md) — 12-week phased plan

---

**ADR Status:** CONSTITUTIONAL (Mandatory compliance for all canvas development)
**Enforcement:** Pre-commit hooks, CI/CD checks, code review checklist
**Review Cadence:** Weekly during implementation, quarterly post-GA

---

**End of ADR-028**
