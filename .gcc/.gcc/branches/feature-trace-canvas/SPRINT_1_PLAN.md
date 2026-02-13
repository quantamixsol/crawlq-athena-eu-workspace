# Sprint 1: MVP Workflow Builder (Week 1-3)

**Duration:** 3 weeks (2026-02-12 to 2026-03-05)
**Goal:** Ship minimal viable workflow builder with React Flow integration
**Team:** Acting as full team (2 frontend engineers + 1 backend + 1 designer + QA + PM)

---

## Sprint Goals

1. ✅ **Infrastructure Setup** (Week 1, Days 1-2)
   - Feature flag system
   - DynamoDB table creation
   - Basic canvas route structure
   - Development environment config

2. ✅ **React Flow Integration** (Week 1, Days 3-5)
   - Basic canvas with 3 node types
   - Drag-drop from tool library
   - Node connections (edges)

3. ✅ **Workflow Execution** (Week 2, Days 1-3)
   - Execution engine (traverse nodes, call LLMs)
   - Progress visualization
   - Error handling

4. ✅ **State Management & Persistence** (Week 2, Days 4-5)
   - Zustand store for canvas state
   - Save/load workflows (DynamoDB)
   - Undo/redo

5. ✅ **Integration Points** (Week 3, Days 1-2)
   - "Open in Canvas" button in DocumentAnalysisPanel
   - Lambda functions (execute-workflow, save-document)

6. ✅ **Testing & Beta** (Week 3, Days 3-5)
   - Unit tests (80%+ coverage)
   - Integration tests
   - Private beta launch (10 users)

---

## Detailed Tasks

### Phase 1.1: Infrastructure Setup (Days 1-2)

#### Task 1.1.1: Feature Flag System
**File:** `src/config/feature-flags.ts`

```typescript
// New file
export const FEATURE_FLAGS = {
  ENABLE_TRACE_CANVAS: process.env.NEXT_PUBLIC_ENABLE_CANVAS === 'true',
  ENABLE_CANVAS_3D_GRAPH: process.env.NEXT_PUBLIC_ENABLE_CANVAS_3D === 'true', // Phase 3
  ENABLE_CANVAS_COLLAB: process.env.NEXT_PUBLIC_ENABLE_CANVAS_COLLAB === 'true', // Phase 6
}
```

**Environment Variables:**
```bash
# .env.local (for development)
NEXT_PUBLIC_ENABLE_CANVAS=true

# .env.production (for prod, disabled initially)
NEXT_PUBLIC_ENABLE_CANVAS=false
```

**Testing:**
- Verify flag returns `false` by default
- Verify flag returns `true` when env var set

---

#### Task 1.1.2: DynamoDB Table Schema
**Table:** `trace-canvas-documents`

**CloudFormation Template:**
```yaml
# infrastructure/canvas-table.yaml
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

**Deployment:**
```bash
aws cloudformation create-stack \
  --stack-name trace-canvas-table \
  --template-body file://infrastructure/canvas-table.yaml \
  --region eu-central-1
```

---

#### Task 1.1.3: Directory Structure
**Create:**
```bash
mkdir -p src/app/canvas
mkdir -p src/components/canvas/workflow
mkdir -p src/components/canvas/shared
mkdir -p src/lib/canvas
mkdir -p __tests__/canvas
```

**Files to Create:**
1. `src/app/canvas/page.tsx` — Canvas listing page
2. `src/app/canvas/new/page.tsx` — Create new canvas
3. `src/app/canvas/[id]/page.tsx` — Edit canvas
4. `src/app/canvas/layout.tsx` — Canvas layout wrapper

---

### Phase 1.2: React Flow Integration (Days 3-5)

#### Task 1.2.1: Install Dependencies
```bash
npm install @xyflow/react zustand @tanstack/react-query
```

**Package Versions:**
- `@xyflow/react`: ^12.0.0 (latest stable)
- `zustand`: ^4.5.0
- `@tanstack/react-query`: ^5.0.0

---

#### Task 1.2.2: Basic Canvas Component
**File:** `src/components/canvas/workflow/WorkflowCanvas.tsx`

```typescript
'use client'

import { useCallback } from 'react'
import ReactFlow, {
  Background,
  Controls,
  MiniMap,
  addEdge,
  useNodesState,
  useEdgesState,
  type Connection,
} from '@xyflow/react'
import '@xyflow/react/dist/style.css'

import { InputNode } from './nodes/InputNode'
import { LLMNode } from './nodes/LLMNode'
import { OutputNode } from './nodes/OutputNode'

const nodeTypes = {
  input: InputNode,
  llm: LLMNode,
  output: OutputNode,
}

export function WorkflowCanvas() {
  const [nodes, setNodes, onNodesChange] = useNodesState([])
  const [edges, setEdges, onEdgesChange] = useEdgesState([])

  const onConnect = useCallback(
    (connection: Connection) => setEdges((eds) => addEdge(connection, eds)),
    [setEdges]
  )

  return (
    <div className="h-screen w-full">
      <ReactFlow
        nodes={nodes}
        edges={edges}
        onNodesChange={onNodesChange}
        onEdgesChange={onEdgesChange}
        onConnect={onConnect}
        nodeTypes={nodeTypes}
        fitView
      >
        <Background />
        <Controls />
        <MiniMap />
      </ReactFlow>
    </div>
  )
}
```

---

#### Task 1.2.3: Node Components
**File:** `src/components/canvas/workflow/nodes/InputNode.tsx`

```typescript
'use client'

import { memo } from 'react'
import { Handle, Position, type NodeProps } from '@xyflow/react'

export const InputNode = memo(({ data }: NodeProps) => {
  return (
    <div className="px-4 py-2 shadow-md rounded-md bg-white border-2 border-blue-500">
      <div className="font-bold text-sm text-gray-700">Input</div>
      <textarea
        className="mt-2 w-full text-sm border rounded p-1"
        placeholder="Enter text or upload file..."
        value={data.value || ''}
        onChange={(e) => data.onChange?.(e.target.value)}
      />
      <Handle type="source" position={Position.Right} />
    </div>
  )
})

InputNode.displayName = 'InputNode'
```

**File:** `src/components/canvas/workflow/nodes/LLMNode.tsx`

```typescript
'use client'

import { memo } from 'react'
import { Handle, Position, type NodeProps } from '@xyflow/react'

export const LLMNode = memo(({ data }: NodeProps) => {
  return (
    <div className="px-4 py-2 shadow-md rounded-md bg-white border-2 border-purple-500">
      <Handle type="target" position={Position.Left} />
      <div className="font-bold text-sm text-gray-700">LLM</div>
      <select
        className="mt-2 w-full text-sm border rounded p-1"
        value={data.model || 'claude-3-5-sonnet'}
        onChange={(e) => data.onModelChange?.(e.target.value)}
      >
        <option value="claude-3-5-sonnet">Claude 3.5 Sonnet</option>
        <option value="gpt-4o">GPT-4o</option>
        <option value="gemini-1.5-pro">Gemini 1.5 Pro</option>
      </select>
      <textarea
        className="mt-2 w-full text-sm border rounded p-1"
        placeholder="Enter prompt..."
        value={data.prompt || ''}
        onChange={(e) => data.onPromptChange?.(e.target.value)}
        rows={3}
      />
      <Handle type="source" position={Position.Right} />
    </div>
  )
})

LLMNode.displayName = 'LLMNode'
```

**File:** `src/components/canvas/workflow/nodes/OutputNode.tsx`

```typescript
'use client'

import { memo } from 'react'
import { Handle, Position, type NodeProps } from '@xyflow/react'

export const OutputNode = memo(({ data }: NodeProps) => {
  return (
    <div className="px-4 py-2 shadow-md rounded-md bg-white border-2 border-green-500">
      <Handle type="target" position={Position.Left} />
      <div className="font-bold text-sm text-gray-700">Output</div>
      {data.result ? (
        <div className="mt-2 text-sm bg-gray-50 border rounded p-2 whitespace-pre-wrap">
          {data.result}
        </div>
      ) : (
        <div className="mt-2 text-sm text-gray-400">Run workflow to see output...</div>
      )}
    </div>
  )
})

OutputNode.displayName = 'OutputNode'
```

---

#### Task 1.2.4: Tool Library (Drag-Drop Palette)
**File:** `src/components/canvas/workflow/ToolLibrary.tsx`

```typescript
'use client'

const tools = [
  { type: 'input', label: 'Input', icon: '📥' },
  { type: 'llm', label: 'LLM', icon: '🤖' },
  { type: 'output', label: 'Output', icon: '📤' },
]

export function ToolLibrary() {
  const onDragStart = (event: React.DragEvent, nodeType: string) => {
    event.dataTransfer.setData('application/reactflow', nodeType)
    event.dataTransfer.effectAllowed = 'move'
  }

  return (
    <aside className="w-64 bg-gray-50 border-r p-4">
      <h3 className="font-bold mb-4">Tools</h3>
      <div className="space-y-2">
        {tools.map((tool) => (
          <div
            key={tool.type}
            className="flex items-center gap-2 p-2 bg-white border rounded cursor-move hover:shadow"
            draggable
            onDragStart={(e) => onDragStart(e, tool.type)}
          >
            <span>{tool.icon}</span>
            <span className="text-sm">{tool.label}</span>
          </div>
        ))}
      </div>
    </aside>
  )
}
```

---

### Phase 1.3: Workflow Execution Engine (Week 2, Days 1-3)

#### Task 1.3.1: Execution Engine
**File:** `src/lib/canvas/workflowExecutor.ts`

```typescript
import type { Node, Edge } from '@xyflow/react'

export interface WorkflowResult {
  nodeId: string
  output: string
  status: 'success' | 'error'
  error?: string
}

export class WorkflowExecutor {
  private nodes: Node[]
  private edges: Edge[]

  constructor(nodes: Node[], edges: Edge[]) {
    this.nodes = nodes
    this.edges = edges
  }

  /**
   * Execute workflow by traversing nodes in topological order
   */
  async execute(): Promise<WorkflowResult[]> {
    const results: WorkflowResult[] = []
    const sortedNodes = this.topologicalSort()

    for (const node of sortedNodes) {
      try {
        const result = await this.executeNode(node, results)
        results.push(result)
      } catch (error) {
        results.push({
          nodeId: node.id,
          output: '',
          status: 'error',
          error: error instanceof Error ? error.message : 'Unknown error',
        })
        break // Stop execution on error
      }
    }

    return results
  }

  private async executeNode(node: Node, previousResults: WorkflowResult[]): Promise<WorkflowResult> {
    switch (node.type) {
      case 'input':
        return {
          nodeId: node.id,
          output: node.data.value || '',
          status: 'success',
        }

      case 'llm':
        const inputNode = this.findInputNode(node)
        const inputData = previousResults.find((r) => r.nodeId === inputNode?.id)?.output || ''

        // Call LLM API
        const response = await fetch('/api/canvas/execute-llm', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            model: node.data.model,
            prompt: node.data.prompt,
            input: inputData,
          }),
        })

        const data = await response.json()

        return {
          nodeId: node.id,
          output: data.output,
          status: 'success',
        }

      case 'output':
        const llmNode = this.findInputNode(node)
        const llmData = previousResults.find((r) => r.nodeId === llmNode?.id)?.output || ''

        return {
          nodeId: node.id,
          output: llmData,
          status: 'success',
        }

      default:
        throw new Error(`Unknown node type: ${node.type}`)
    }
  }

  private findInputNode(node: Node): Node | undefined {
    const incomingEdge = this.edges.find((e) => e.target === node.id)
    if (!incomingEdge) return undefined
    return this.nodes.find((n) => n.id === incomingEdge.source)
  }

  private topologicalSort(): Node[] {
    // Simple topological sort (assumes DAG, no cycles)
    const visited = new Set<string>()
    const sorted: Node[] = []

    const visit = (node: Node) => {
      if (visited.has(node.id)) return
      visited.add(node.id)

      const incomingEdges = this.edges.filter((e) => e.target === node.id)
      incomingEdges.forEach((edge) => {
        const sourceNode = this.nodes.find((n) => n.id === edge.source)
        if (sourceNode) visit(sourceNode)
      })

      sorted.push(node)
    }

    this.nodes.forEach((node) => visit(node))
    return sorted
  }
}
```

---

### Phase 1.4: State Management (Week 2, Days 4-5)

#### Task 1.4.1: Zustand Store
**File:** `src/lib/canvas/canvasStore.ts`

```typescript
import { create } from 'zustand'
import { persist } from 'zustand/middleware'
import type { Node, Edge } from '@xyflow/react'

interface CanvasState {
  canvasId: string | null
  nodes: Node[]
  edges: Edge[]
  mode: 'workflow' | 'freeform' | 'graph'

  setCanvasId: (id: string) => void
  setNodes: (nodes: Node[]) => void
  setEdges: (edges: Edge[]) => void
  setMode: (mode: 'workflow' | 'freeform' | 'graph') => void

  saveCanvas: () => Promise<void>
  loadCanvas: (id: string) => Promise<void>
}

export const useCanvasStore = create<CanvasState>()(
  persist(
    (set, get) => ({
      canvasId: null,
      nodes: [],
      edges: [],
      mode: 'workflow',

      setCanvasId: (id) => set({ canvasId: id }),
      setNodes: (nodes) => set({ nodes }),
      setEdges: (edges) => set({ edges }),
      setMode: (mode) => set({ mode }),

      saveCanvas: async () => {
        const { canvasId, nodes, edges, mode } = get()

        const response = await fetch('/api/canvas/save', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            canvasId,
            workflowData: { nodes, edges },
            mode,
          }),
        })

        if (!response.ok) throw new Error('Failed to save canvas')
      },

      loadCanvas: async (id) => {
        const response = await fetch(`/api/canvas/load?id=${id}`)
        if (!response.ok) throw new Error('Failed to load canvas')

        const data = await response.json()
        set({
          canvasId: data.canvasId,
          nodes: data.workflowData.nodes,
          edges: data.workflowData.edges,
          mode: data.mode,
        })
      },
    }),
    {
      name: 'canvas-storage',
    }
  )
)
```

---

### Phase 1.5: API Routes (Week 3, Days 1-2)

#### Task 1.5.1: Execute LLM API Route
**File:** `src/app/api/canvas/execute-llm/route.ts`

```typescript
import { NextResponse } from 'next/server'

export async function POST(request: Request) {
  const { model, prompt, input } = await request.json()

  // Call existing chat-athena-eu Lambda
  const response = await fetch(process.env.CHAT_ATHENA_EU_URL!, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      model,
      messages: [
        { role: 'system', content: prompt },
        { role: 'user', content: input },
      ],
    }),
  })

  const data = await response.json()

  return NextResponse.json({ output: data.content })
}
```

#### Task 1.5.2: Save Canvas API Route
**File:** `src/app/api/canvas/save/route.ts`

```typescript
import { NextResponse } from 'next/server'
import { DynamoDBClient, PutItemCommand } from '@aws-sdk/client-dynamodb'
import { marshall } from '@aws-sdk/util-dynamodb'
import { v4 as uuidv4 } from 'uuid'

const dynamodb = new DynamoDBClient({ region: 'eu-central-1' })

export async function POST(request: Request) {
  const { canvasId, workflowData, mode } = await request.json()
  const userId = 'temp-user' // TODO: Get from auth session

  const id = canvasId || uuidv4()

  await dynamodb.send(
    new PutItemCommand({
      TableName: 'trace-canvas-documents',
      Item: marshall({
        userId,
        canvasId: id,
        mode,
        workflowData,
        createdAt: new Date().toISOString(),
        updatedAt: new Date().toISOString(),
      }),
    })
  )

  return NextResponse.json({ canvasId: id })
}
```

---

## Sprint 1 Success Criteria

- [ ] Feature flag system working (can enable/disable canvas)
- [ ] DynamoDB table created and accessible
- [ ] Canvas route accessible at `/canvas/new`
- [ ] React Flow renders with drag-drop tool library
- [ ] Can create workflow: Input → LLM → Output
- [ ] Can execute workflow (calls LLM API, displays result)
- [ ] Can save workflow to DynamoDB
- [ ] Can load workflow from DynamoDB
- [ ] "Open in Canvas" button appears in DocumentAnalysisPanel (feature-flagged)
- [ ] 80%+ test coverage for canvas components
- [ ] Private beta launched with 10 users
- [ ] Zero regressions in existing TRACE UI (all existing tests pass)

---

## Sprint 1 Demo (End of Week 3)

**Demo Script:**
1. Show feature flag toggle (enable/disable canvas)
2. Navigate to `/canvas/new`
3. Drag Input node onto canvas
4. Configure: "Analyze this PDF for GDPR compliance risks"
5. Drag LLM node, connect to Input
6. Configure: Model = Claude 3.5 Sonnet, Prompt = "Extract compliance risks"
7. Drag Output node, connect to LLM
8. Click "Run Workflow"
9. Show real-time execution (Input → green, LLM → processing, Output → displays result)
10. Click "Save" → confirm saved to DynamoDB
11. Refresh page → load workflow from DynamoDB
12. Show "Open in Canvas" button in existing DocumentAnalysisPanel

**Metrics to Track:**
- Workflow execution success rate (target: >90%)
- Average execution time (target: <10s for simple workflow)
- User feedback (target: 8/10 NPS from beta users)

---

## Next Sprint Preview (Sprint 2: Markdown + Freeform)

- Markdown card overlays
- Wikilink parsing
- Tldraw freeform canvas integration
- Mode switcher (Workflow ↔ Freeform)
- Public beta launch

---

**End of Sprint 1 Plan**
