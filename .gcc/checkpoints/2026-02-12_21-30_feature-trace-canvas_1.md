# Checkpoint: feature-trace-canvas — COMMIT 1

**Branch:** feature-trace-canvas
**Commit:** 1
**Timestamp:** 2026-02-12T21:30:00Z
**State:** WORKING

---

## Milestone

Branch setup complete — Constitutional ADR-028 created, Sprint 1 plan finalized, infrastructure design ready for implementation

---

## Summary

Created implementation branch for TRACE Canvas with full constitutional guarantees of zero breaking changes. Established ADR-028 as mandatory compliance framework with 7 enforceable principles (Non-Breaking Mandate, Isolated Routes, Isolated Components, Shared Resources, Backward Compatibility, Feature Flags, Testing Isolation). Designed comprehensive Sprint 1 plan (3 weeks) for MVP Workflow Builder with React Flow integration, 3 node types (Input, LLM, Output), execution engine with topological sort, Zustand state management, DynamoDB persistence, and API routes. All infrastructure designed to be additive-only (no modifications to existing TRACE system). Ready to begin Phase 1.1 (Infrastructure Setup) implementation.

---

## Files Created

### 1. **ADR-028: Constitutional Non-Breaking Implementation Principles**
**Path:** `.gsm/decisions/ADR-028-constitutional-non-breaking-implementation.md`
**Purpose:** Mandatory compliance framework guaranteeing zero breaking changes
**Size:** ~8,000 words

**7 Constitutional Principles:**
1. **Non-Breaking Mandate** — No modifications to existing TRACE components (TraceDashboard, TraceKnowledgeGraph, DocumentAnalysisPanel, etc.) unless explicitly extending with optional features
2. **Isolated Routes** — All canvas routes under `/canvas/*` (no changes to existing `/guest-document-analysis/*` or `/chat-athena-eu/*`)
3. **Isolated Components** — All canvas components in `src/components/canvas/*` (no mixing with `trace-eu/*` or `chat-eu/*`)
4. **Shared Resources (Non-Destructive)** — Use existing DynamoDB/S3/Lambda but add new resources (new table `trace-canvas-documents`, new Lambda functions, new S3 prefix)
5. **Backward Compatibility** — Existing document analysis flow unchanged; canvas is additive feature via "Open in Canvas" button
6. **Feature-Flagged Rollout** — `ENABLE_TRACE_CANVAS` env var for safe beta testing and kill-switch capability
7. **Testing Isolation** — All canvas tests in `__tests__/canvas/*` (no interference with existing test suites)

**Enforcement Mechanisms:**
- Pre-commit Git hooks (block modifications to protected files)
- ESLint rules (prevent canvas importing from trace-eu/chat-eu)
- CI/CD checks (GitHub Actions ADR compliance check)
- Code review checklist (9-point compliance verification)

**Violation Process:**
- Pre-commit hook fails → cannot commit
- CI/CD fails → cannot merge PR
- Code review rejection → must fix before approval
- Intentional violation requires ADR amendment + leadership approval

### 2. **Sprint 1 Plan: MVP Workflow Builder**
**Path:** `.gcc/branches/feature-trace-canvas/SPRINT_1_PLAN.md`
**Duration:** 3 weeks (2026-02-12 to 2026-03-05)
**Goal:** Ship minimal viable workflow builder with React Flow

**6 Phases:**
1. **Phase 1.1: Infrastructure Setup (Days 1-2)**
   - Feature flag system (`src/config/feature-flags.ts`)
   - DynamoDB table schema (CloudFormation template)
   - Directory structure (`src/app/canvas`, `src/components/canvas`, `__tests__/canvas`)

2. **Phase 1.2: React Flow Integration (Days 3-5)**
   - Install `@xyflow/react`, `zustand`, `@tanstack/react-query`
   - Build `WorkflowCanvas.tsx` (React Flow wrapper)
   - Build node components: `InputNode`, `LLMNode`, `OutputNode`
   - Build `ToolLibrary.tsx` (drag-drop palette)

3. **Phase 1.3: Workflow Execution Engine (Week 2, Days 1-3)**
   - Build `WorkflowExecutor.ts` (topological sort, node traversal)
   - Execute nodes in order (Input → LLM → Output)
   - Handle errors (stop execution, display error state)
   - Progress visualization (green = success, yellow = in-progress, red = error)

4. **Phase 1.4: State Management (Week 2, Days 4-5)**
   - Build Zustand store (`canvasStore.ts`)
   - Implement save/load workflows (DynamoDB)
   - Undo/redo (React Flow built-in)

5. **Phase 1.5: API Routes (Week 3, Days 1-2)**
   - `/api/canvas/execute-llm` — Call existing chat-athena-eu Lambda
   - `/api/canvas/save` — Save canvas to DynamoDB
   - `/api/canvas/load` — Load canvas from DynamoDB

6. **Phase 1.6: Testing & Beta (Week 3, Days 3-5)**
   - Unit tests (80%+ coverage target)
   - Integration tests (end-to-end workflow execution)
   - Private beta launch (10 users)

**Success Criteria:**
- [ ] Feature flag working (can enable/disable)
- [ ] DynamoDB table created
- [ ] Canvas accessible at `/canvas/new`
- [ ] React Flow renders with drag-drop
- [ ] Workflow execution works (Input → LLM → Output)
- [ ] Save/load to DynamoDB
- [ ] "Open in Canvas" button in DocumentAnalysisPanel
- [ ] 80%+ test coverage
- [ ] Private beta launched
- [ ] Zero regressions (all existing tests pass)

### 3. **Branch Metadata Files**
**Created:**
- `.gcc/branches/feature-trace-canvas/commit.md` — Commit log with BRANCH CREATED + COMMIT 1
- `.gcc/branches/feature-trace-canvas/log.md` — Session log template
- `.gcc/branches/feature-trace-canvas/metadata.yaml` — Branch metadata

**Registry Updated:**
- `.gcc/registry.md` — Added feature-trace-canvas entry (WORKING status, parent: feature-eu-standalone-app)

---

## Key Architectural Decisions

### Decision 1: ADR-028 as Constitutional Document
**Status:** CONSTITUTIONAL (cannot be violated)

**Rationale:** User explicitly requested "zero breaking changes" and "don't want to break other functionality". Need enforceable guarantees beyond standard best practices.

**Impact:** All canvas development subject to 7 mandatory principles. Any PR modifying protected components automatically rejected. Pre-commit hooks prevent accidental violations.

**Alternatives Considered:**
- Standard code review only → REJECTED (too error-prone, no automation)
- No formal rules → REJECTED (violates user requirement for zero breaking changes)

---

### Decision 2: New DynamoDB Table (trace-canvas-documents)
**Instead of:** Adding canvas fields to existing `trace-documents` table

**Rationale:**
- ADR-028 Principle 4: Shared Resources (non-destructive)
- Prevents schema conflicts
- Allows independent scaling (canvas may have different access patterns)
- Easier to rollback (can delete table without affecting existing data)

**Schema:**
```yaml
PK: userId (HASH)
SK: canvasId (RANGE)
GSI: canvasId-index (for sharing)
```

**Attributes:**
- `mode`: workflow | freeform | graph
- `workflowData`: { nodes, edges }
- `markdownCards`: array
- `traceMetadata`: { auditHash, lineage, confidenceScores }

---

### Decision 3: Feature Flag Rollout Strategy
**Environment Variable:** `NEXT_PUBLIC_ENABLE_CANVAS`

**Rollout:**
1. **Week 3 (Private Beta):** 10 users only (env var override per user)
2. **Week 6 (Public Beta):** All users, announced as "beta"
3. **Week 9 (GA):** Full release, flag remains as kill-switch

**Rationale:** Safe incremental rollout, ability to disable if critical issues discovered, A/B testing capability (future).

---

### Decision 4: Zustand over Redux
**State Management:** Zustand (not Redux)

**Rationale:**
- Lightweight (3KB vs 60KB for Redux + middleware)
- Simpler DX (no actions/reducers boilerplate)
- React-first (hooks-based)
- Built-in persistence (`zustand/middleware`)

**Use Cases:**
- Canvas state (nodes, edges, mode)
- Save/load workflows
- Undo/redo (React Flow handles this, Zustand stores history)

---

### Decision 5: React Flow for Workflow Builder
**Library:** `@xyflow/react` (React Flow v12)

**Rationale:** (From ADR-026)
- Production-proven (Stripe, Typeform, Instagram)
- TypeScript-native
- React-first (nodes are React components)
- Built-in features (multi-select, undo/redo, minimap, controls)

**Alternatives:**
- Tldraw → Better for freeform, not optimized for workflows
- Custom canvas → 3-6 months dev time (too slow)

---

## Infrastructure Design

### DynamoDB Table Schema
```yaml
# CloudFormation template
Resources:
  CanvasDocumentsTable:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: trace-canvas-documents
      BillingMode: PAY_PER_REQUEST
      AttributeDefinitions:
        - AttributeName: userId
          AttributeType: S
        - AttributeName: canvasId
          AttributeType: S
      KeySchema:
        - AttributeName: userId
          KeyType: HASH
        - AttributeName: canvasId
          KeyType: RANGE
      GlobalSecondaryIndexes:
        - IndexName: canvasId-index
          KeySchema:
            - AttributeName: canvasId
              KeyType: HASH
          Projection:
            ProjectionType: ALL
```

### Directory Structure
```
src/
├── app/
│   └── canvas/                       ← NEW
│       ├── page.tsx
│       ├── new/page.tsx
│       ├── [id]/page.tsx
│       └── layout.tsx
├── components/
│   └── canvas/                       ← NEW
│       ├── workflow/
│       │   ├── WorkflowCanvas.tsx
│       │   ├── nodes/
│       │   │   ├── InputNode.tsx
│       │   │   ├── LLMNode.tsx
│       │   │   └── OutputNode.tsx
│       │   └── ToolLibrary.tsx
│       └── shared/
├── lib/
│   └── canvas/                       ← NEW
│       ├── workflowExecutor.ts
│       └── canvasStore.ts
└── config/
    └── feature-flags.ts              ← NEW

__tests__/
└── canvas/                           ← NEW
    └── workflow/
```

### API Routes
```
/api/canvas/
├── execute-llm       (POST) — Execute LLM node
├── save              (POST) — Save canvas to DynamoDB
└── load              (GET)  — Load canvas from DynamoDB
```

---

## Sprint 1 Component Samples

### WorkflowCanvas.tsx (React Flow)
```tsx
'use client'

import ReactFlow, {
  Background,
  Controls,
  MiniMap,
  useNodesState,
  useEdgesState,
} from '@xyflow/react'

const nodeTypes = {
  input: InputNode,
  llm: LLMNode,
  output: OutputNode,
}

export function WorkflowCanvas() {
  const [nodes, setNodes, onNodesChange] = useNodesState([])
  const [edges, setEdges, onEdgesChange] = useEdgesState([])

  return (
    <ReactFlow
      nodes={nodes}
      edges={edges}
      onNodesChange={onNodesChange}
      onEdgesChange={onEdgesChange}
      nodeTypes={nodeTypes}
      fitView
    >
      <Background />
      <Controls />
      <MiniMap />
    </ReactFlow>
  )
}
```

### InputNode.tsx
```tsx
export const InputNode = memo(({ data }: NodeProps) => {
  return (
    <div className="px-4 py-2 border-2 border-blue-500">
      <div className="font-bold">Input</div>
      <textarea
        value={data.value || ''}
        onChange={(e) => data.onChange?.(e.target.value)}
      />
      <Handle type="source" position={Position.Right} />
    </div>
  )
})
```

### WorkflowExecutor.ts
```typescript
export class WorkflowExecutor {
  async execute(): Promise<WorkflowResult[]> {
    const sortedNodes = this.topologicalSort()

    for (const node of sortedNodes) {
      const result = await this.executeNode(node)
      // Store result, continue to next node
    }
  }

  private topologicalSort(): Node[] {
    // Sort nodes by dependencies (Input before LLM before Output)
  }
}
```

---

## Next Actions (Phase 1.1: Infrastructure Setup)

**Week 1, Days 1-2:**

1. **Create Directory Structure**
   ```bash
   mkdir -p src/app/canvas/{new,[id]}
   mkdir -p src/components/canvas/workflow/nodes
   mkdir -p src/lib/canvas
   mkdir -p __tests__/canvas
   ```

2. **Install Dependencies**
   ```bash
   npm install @xyflow/react@^12.0.0
   npm install zustand@^4.5.0
   npm install @tanstack/react-query@^5.0.0
   npm install @aws-sdk/client-dynamodb @aws-sdk/util-dynamodb
   ```

3. **Create Feature Flag System**
   ```typescript
   // src/config/feature-flags.ts
   export const FEATURE_FLAGS = {
     ENABLE_TRACE_CANVAS: process.env.NEXT_PUBLIC_ENABLE_CANVAS === 'true',
   }
   ```

4. **Deploy DynamoDB Table**
   ```bash
   aws cloudformation create-stack \
     --stack-name trace-canvas-table \
     --template-body file://infrastructure/canvas-table.yaml \
     --region eu-central-1
   ```

5. **Create Basic Canvas Route**
   ```tsx
   // src/app/canvas/new/page.tsx
   export default function NewCanvasPage() {
     if (!FEATURE_FLAGS.ENABLE_TRACE_CANVAS) {
       return <ComingSoon />
     }
     return <CanvasContainer />
   }
   ```

---

## Open Questions

1. **Guest vs Auth-Only Access:**
   - Should canvas be accessible to guest users (like existing document analysis)?
   - Or require authentication?
   - **Decision needed by:** End of Week 1
   - **Impact:** Affects API route security, DynamoDB partitioning strategy

2. **Tldraw Commercial License:**
   - Timeline for budget approval (~$5K/year)?
   - Alternative: Use Excalidraw (MIT license, hand-drawn aesthetic)
   - **Decision needed by:** Before Phase 2 (Week 4)

3. **Private Beta User Recruitment:**
   - Who are the 10 beta users? (PhD researchers, compliance officers, B2B analysts)
   - How to recruit? (Email campaign, existing power users, manual outreach)
   - **Decision needed by:** Week 2 (to launch Week 3)

4. **Performance Targets:**
   - Is <2s initial load achievable with lazy loading?
   - Is <1s canvas render (1000 nodes) realistic with React Flow?
   - May need virtualization or pagination
   - **Validate in:** Week 1 (technical spike)

---

## Risks & Mitigation

**Risk 1: React Flow performance with large graphs (1000+ nodes)**
- **Probability:** Medium
- **Impact:** High (poor UX)
- **Mitigation:** Week 1 spike to test performance, implement virtualization if needed

**Risk 2: Feature flag accidentally left enabled in production**
- **Probability:** Low
- **Impact:** High (unfinished feature visible to all users)
- **Mitigation:** CI/CD check to ensure `ENABLE_CANVAS=false` in prod env file

**Risk 3: DynamoDB table name conflict**
- **Probability:** Low
- **Impact:** Medium (deployment fails)
- **Mitigation:** Check existing table names before CloudFormation deploy

**Risk 4: Breaking change accidentally introduced**
- **Probability:** Low (with ADR-028 enforcement)
- **Impact:** Critical (violates user requirement)
- **Mitigation:** Pre-commit hooks, CI/CD checks, strict code review

---

## Constitutional Compliance Status

**ADR-028 Principles — Current Compliance:**
- ✅ **Principle 1 (Non-Breaking):** No existing components modified (all work in new `canvas/` directory)
- ✅ **Principle 2 (Isolated Routes):** All routes under `/canvas/*`
- ✅ **Principle 3 (Isolated Components):** All components in `src/components/canvas/*`
- ✅ **Principle 4 (Shared Resources):** New DynamoDB table, new S3 prefix, no modifications to existing
- ✅ **Principle 5 (Backward Compatibility):** Canvas is additive (accessed via new "Open in Canvas" button)
- ✅ **Principle 6 (Feature Flags):** `ENABLE_TRACE_CANVAS` env var designed
- ✅ **Principle 7 (Testing Isolation):** Tests in `__tests__/canvas/*`

**Violations:** 0

---

## Session Summary

Created feature-trace-canvas implementation branch with constitutional guarantees of zero breaking changes. Established ADR-028 as mandatory framework (7 principles, enforcement via pre-commit hooks/CI/CD). Designed comprehensive Sprint 1 plan (MVP Workflow Builder, 3 weeks, 6 phases). All infrastructure additive-only: new routes (`/canvas/*`), new components (`canvas/*`), new DynamoDB table (`trace-canvas-documents`), new Lambda functions. Ready to begin Phase 1.1 (Infrastructure Setup) with directory creation, npm installs, feature flags, DynamoDB deployment.

---

**Checkpoint Status:** COMPLETE
**Branch State:** WORKING (setup complete, ready for implementation)
**Next Commit:** COMMIT 2 (after Phase 1.1 Infrastructure Setup complete)
