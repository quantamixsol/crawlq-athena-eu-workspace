# ADR-036: TRACE Canvas Integration into Athena EU Main App

**Date:** 2026-02-14 | **Status:** ACCEPTED

## Context

TRACE Canvas has been developed as a separate application (`crawlq-athena-eu-canvas/`) through 22 commits spanning 14 builds. It now has:

- **21 components** (6 shared, 10 workflow, 5 node types)
- **9 lib modules** (store, executor, health analyzer, coach engine, connection suggester, prompt suggester, templates, example workflows)
- **5 API routes** (execute-llm, save, load, list, delete)
- **4 protected routes** (canvas list, new, [id], layout)
- **11 test suites, 125 tests** — all passing
- **1 type definition** file (canvas.ts)

The main Athena EU app (`crawlq-chat-athena-eu-frontend/`) already has:
- Feature flags for canvas (`ENABLE_TRACE_CANVAS: false` in `src/config/feature-flags.ts`)
- EU plan tier system with feature gating (`eu-plans.ts`, `useEUFeatureGate.ts`)
- Same Cognito auth, same DynamoDB backend, same Lambda functions

**Problem:** Canvas lives in a separate repo/app. Users must visit a different URL. There is no subscription gating, no tier-based limits, and no Chat↔Canvas bridge. To ship Canvas as a production feature, it must be integrated into the main app.

## Decision

### Integration Strategy: Monorepo Merge (Code Copy)

Copy all canvas code into the main Athena EU frontend repo as isolated modules. This is preferred over:

| Alternative | Why Rejected |
|-------------|-------------|
| **Separate add-on product** | Two subscriptions confuse users, separate auth flows, separate billing |
| **Microfrontend (Module Federation)** | Runtime complexity, version drift, debugging overhead, CORS issues |
| **Route proxy / iframe** | Performance overhead, auth token passing complexity, broken UX |
| **npm package** | Versioning overhead for internal-only code, deployment lag |

**Rationale:** Canvas code is already isolated by convention (ADR-028: `/components/canvas/*`, `/lib/canvas/*`, `/app/canvas/*`). Copying it into the main repo preserves this isolation while enabling shared auth, shared navigation, shared subscription gating, and single deployment.

### Subscription Strategy: Tiered Canvas Access (NOT Separate Add-on)

Canvas is included in existing EU plan tiers with progressive limits. This maximizes conversion because:
1. Explorer users get a taste → hit limits → upgrade prompt appears
2. No separate purchase decision → lower friction
3. Canvas usage data informs upsell timing

#### Canvas Limits by Tier

| Feature | Explorer (€0) | Professional (€39/mo) | Business (€99/seat/mo) | Enterprise (€499+/mo) |
|---------|--------------|----------------------|----------------------|---------------------|
| Canvas access | Yes (limited) | Yes | Yes | Yes |
| Max canvases | 1 | 10 | Unlimited (-1) | Unlimited (-1) |
| Runs per day | 3 | 50 | Unlimited (-1) | Unlimited (-1) |
| Models | Claude only | All 3 | All 3 | All 3 + custom |
| Branch node | No | Yes | Yes | Yes |
| Canvas export | MD only | MD, PDF | MD, PDF, DOCX | MD, PDF, DOCX |
| Template library | 3 basic | All | All | All + custom |
| Canvas API access | No | No | No | Yes |

#### New `EUPlanFeatures` Fields

```typescript
// Added to existing EUPlanFeatures interface in eu-plans.ts
canvasEnabled: boolean;
maxCanvases: number;        // -1 = unlimited
canvasRunsPerDay: number;   // -1 = unlimited
canvasBranch: boolean;
canvasModels: string[];     // e.g., ["claude-3-5-sonnet"] or ["claude-3-5-sonnet", "gpt-4o", "gemini-1.5-pro"]
canvasExportFormats: string[];
canvasTemplateAccess: "basic" | "all" | "custom";
canvasApiAccess: boolean;
```

#### New `useEUFeatureGate` Returns

```typescript
// Added to existing hook return
canUseCanvas: boolean;
canvasLimit: number;
canvasRunsPerDay: number;
canUseBranch: boolean;
canUseCanvasApi: boolean;
canvasModels: string[];
canvasExportFormats: string[];
canvasTemplateAccess: "basic" | "all" | "custom";
```

### Code Copy Manifest

| Source (canvas repo) | Destination (main app) | Notes |
|---------------------|----------------------|-------|
| `src/components/canvas/**` | `src/components/canvas/**` | Direct copy, same paths |
| `src/lib/canvas/**` | `src/lib/canvas/**` | Direct copy, same paths |
| `src/types/canvas.ts` | `src/types/canvas.ts` | Direct copy |
| `src/app/(protected)/canvas/**` | `src/app/(protected)/canvas/**` | Direct copy, add feature gate |
| `src/app/api/canvas/**` | `src/app/api/canvas/**` | Direct copy |
| `__tests__/canvas/**` | `__tests__/canvas/**` | Direct copy |

### Integration Points (Main App Modifications)

| File | Change |
|------|--------|
| `src/constants/eu-plans.ts` | Add 8 canvas fields to `EUPlanFeatures` interface and all 4 tier configs |
| `src/hooks/useEUFeatureGate.ts` | Add 8 canvas gate return values |
| `src/config/feature-flags.ts` | Flip `ENABLE_TRACE_CANVAS: true` |
| `src/app/(protected)/canvas/layout.tsx` | Wrap with feature gate check |
| Sidebar / Navigation component | Add "Canvas" link (gated by `canUseCanvas`) |

### Chat↔Canvas Bridge (Phase 2)

After initial merge, add "Open in Canvas" button to chat responses:
- Button appears on AI responses that contain structured content
- Clicking creates a new canvas with Input node pre-filled from the chat response
- Uses existing `OpenInCanvasButton.tsx` component (already built in canvas repo)

### Conversion Triggers

When a user hits a canvas limit:
1. Show existing `UpgradeModal` with canvas-specific messaging
2. Track `canvas_limit_reached` event via FrictionMelt
3. Display comparison of current tier vs next tier's canvas capabilities

## Consequences

### Positive
- Single app, single URL, single subscription — minimal user friction
- Shared auth (Cognito), shared navigation, shared billing (Stripe)
- Canvas feature-gated by tier → natural conversion funnel
- Canvas code remains isolated (ADR-028 still holds) — can be reverted by deleting directories
- Explorer users get taste → Professional upgrade for full access
- "World's first" positioning: Chat + Workflow + TRACE + Compliance + Friction Intelligence in one app

### Negative
- Main app bundle size increases (~200KB for React Flow + canvas components)
- Must keep canvas tests passing alongside existing 208+ tests
- Feature flag flip is a one-way door (users will expect Canvas to stay)

### Risks & Mitigations
| Risk | Mitigation |
|------|-----------|
| Bundle size bloat | Next.js dynamic imports for canvas routes (loaded only when visited) |
| Test interference | Canvas tests isolated in `__tests__/canvas/` (ADR-028) |
| Breaking existing app | Feature flag + ADR-028 isolation = kill switch available |
| Subscription confusion | Clear tier comparison table in pricing page |

## Implementation Phases

### Phase 1: Merge + Gate (This Sprint)
1. Create ADR-036 (this document)
2. Update `eu-plans.ts` with canvas fields
3. Extend `useEUFeatureGate.ts` with canvas gates
4. Copy all canvas code to main repo
5. Wire navigation link
6. Flip feature flag
7. Build + Test + Deploy

### Phase 2: Polish + Conversion (Next Sprint)
1. Chat↔Canvas bridge ("Open in Canvas" button)
2. Limit-reached upgrade modals
3. FrictionMelt canvas event tracking
4. Pricing page canvas comparison table

### Phase 3: Advanced Features (Future)
1. Real Lambda execution (CANVAS_MOCK_LLM=false)
2. PatternDetection node, ComplianceGate node
3. Canvas API access for Enterprise tier
4. Custom templates for Enterprise
5. Collaborative canvas (multi-user editing)

## Dependencies

- ADR-028: Constitutional Non-Breaking Implementation (canvas isolation)
- ADR-034: BTDI Workflow (mandatory build-test-deploy cycle)
- EU Plan Tiers: `eu-plans.ts` is single source of truth
- Feature Flags: `feature-flags.ts` controls rollout
