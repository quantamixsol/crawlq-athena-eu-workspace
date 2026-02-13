# Phase 1: US-to-EU TRACE UI Porting & Alignment Plan
## Comprehensive Sprint-by-Sprint Implementation Roadmap

**Date:** 2026-02-11
**Branch:** feature-eu-chat-athena
**Scope:** Port all 21 US TRACE UI components to EU + implement guest flow
**Duration:** 5 weeks (25 business days)
**References:**
- Gap Analysis: `.gcc/branches/feature-eu-chat-athena/gap-analysis.md`
- ADR-013: US Region Non-Interference Policy
- ADR-012: Tier 3 Async Architecture (separate branch, no conflicts)

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [US Component Inventory](#2-us-component-inventory)
3. [EU Target Architecture](#3-eu-target-architecture)
4. [Porting Strategy](#4-porting-strategy)
5. [Sprint Breakdown](#5-sprint-breakdown)
6. [Conflict Avoidance with ADR-012](#6-conflict-avoidance-with-adr-012)
7. [Success Metrics](#7-success-metrics)

---

## 1. Executive Summary

### Current State (COMMIT 9)
- **US:** 21 TRACE UI components, full guest document flow, Knowledge Graph visualization
- **EU:** 15 chat-eu components, 0 TRACE UI components, no guest flow
- **Gap:** EU has superior backend (17 Lambdas, API Gateway, compliance) but zero TRACE value proposition

### Phase 1 Goal
**Port all 21 US TRACE UI components to EU while maintaining strict READ-ONLY US policy (ADR-013) and avoiding conflicts with ADR-012 (Tier 3 async architecture work).**

### Success Criteria
- ✅ 21 EU-specific TRACE components created in `components/trace-eu/` folder
- ✅ Guest document analysis flow working end-to-end
- ✅ Knowledge Graph integrated with EU chat
- ✅ Fix Strategy prompt builder integrated
- ✅ Zero modifications to US code (ADR-013 compliance)
- ✅ Zero conflicts with ADR-012 markdown renderer work
- ✅ 100% test coverage for ported components

---

## 2. US Component Inventory

### 2.1 Core TRACE UI Components (Read-Only Reference)

| # | Component Name | Location (US) | Purpose | Priority |
|---|----------------|---------------|---------|----------|
| 1 | `TraceExplainabilityPanel` | `components/knowledge-graph/trace-explainability/` | Main TRACE 5-pillar panel | **P0** |
| 2 | `TraceKnowledgeGraph` | `components/trace-knowledge-graph/` | Neo4j NVL interactive graph | **P0** |
| 3 | `DeepDocumentDetails` | `components/knowledge-graph/deep-document-analysis/` | Full document analysis UI | **P0** |
| 4 | `promptBuilder` | `components/knowledge-graph/deep-document-analysis/` | Fix Strategy prompt generator | **P0** |
| 5 | `ConfidenceBadge` | `trace-explainability-panel.tsx` (sub-component) | Confidence level display | **P0** |
| 6 | `HumanExplanation` | `trace-explainability-panel.tsx` (sub-component) | Human-readable explanation | **P1** |
| 7 | `TrustSignal` | `trace-explainability-panel.tsx` (sub-component) | Trust indicators | **P1** |
| 8 | `LineageTimeline` | `trace-explainability-panel.tsx` (sub-component) | Decision path timeline | **P1** |
| 9 | `KeyFactors` | `trace-explainability-panel.tsx` (sub-component) | Contributing factors | **P1** |
| 10 | `MetricsGrid` | `trace-explainability-panel.tsx` (sub-component) | 6 explainability metrics | **P1** |
| 11 | `LineageStep` | `components/knowledge-graph/trace/` | Individual lineage step | **P1** |
| 12 | `ReportHeader` | `deep-document-analysis/report-header.tsx` | Analysis header with audit | **P1** |
| 13 | `CriticalIssueSection` | `guest-document-analysis/analytics/` | Critical issue display | **P1** |
| 14 | `InsightCard` | `guest-document-analysis/analytics/` | Individual insight display | **P1** |
| 15 | `SummaryAndScore` | `guest-document-analysis/analytics/` | Executive summary + score | **P1** |
| 16 | `DeepDocumentUpload` | `deep-document-analysis/` | Document upload UI | **P2** |
| 17 | `DeepDocumentList` | `deep-document-analysis/` | Document list view | **P2** |
| 18 | `DeepDocumentLanding` | `deep-document-analysis/` | Document landing page | **P2** |
| 19 | `GuestProjectOnboarding` | `chat-athena/components/deep-document-analysis/` | Guest onboarding modal | **P2** |
| 20 | `TraceGraphToggler` | `chat-athena/components/chat/trace-graph-toggler/` | KG toggle switch | **P2** |
| 21 | `Trace` (chain of thought) | `chat-athena/components/message/Trace.tsx` | Basic trace display | **P3** |

### 2.2 Supporting Types and Utilities

| File | Location (US) | Purpose |
|------|---------------|---------|
| `types.ts` | `trace-explainability/types/` | TraceResult, ConfidenceLevel, Metrics types |
| `TRACE_COMPLIANCE_BRAND_VOICE` | `promptBuilder.tsx` | 86-line system prompt for Fix Strategy |
| `buildUserPromptFromInsight` | `promptBuilder.tsx` | Insight → prompt transformer |

### 2.3 Dependencies (US)

```tsx
// US imports (DO NOT MODIFY - ADR-013)
import { useSendMessage } from "@/queries/chat-athena/send-message/useSendMessage";
import { useChatInputSettingsStore } from "@/store/useChatInputSettingsStore";
import { IWorkspaceDocument } from "@/queries/deep-document-analysis/use-Get-Deep-Documents-Query";
```

---

## 3. EU Target Architecture

### 3.1 EU Folder Structure (NEW)

```
crawlq-ui/src/components/
├── chat-eu/                              ← Existing (15 components)
│   ├── ChatContainer.tsx
│   ├── ChatMessageBubble.tsx
│   └── ... (12 more)
│
├── trace-eu/                             ← NEW (Phase 1)
│   ├── trace-explainability/
│   │   ├── TraceExplainabilityPanelEU.tsx    ← Port from US
│   │   ├── ConfidenceBadgeEU.tsx             ← Port from US
│   │   ├── HumanExplanationEU.tsx            ← Port from US
│   │   ├── TrustSignalEU.tsx                 ← Port from US
│   │   ├── LineageTimelineEU.tsx             ← Port from US
│   │   ├── KeyFactorsEU.tsx                  ← Port from US
│   │   ├── MetricsGridEU.tsx                 ← Port from US
│   │   └── types/
│   │       └── trace-types-eu.ts             ← Port + extend
│   │
│   ├── knowledge-graph-eu/
│   │   ├── TraceKnowledgeGraphEU.tsx         ← Port from US
│   │   ├── TraceKnowledgeGraphSkeletonEU.tsx ← Port from US
│   │   └── graph-utils-eu.ts                 ← EU-specific graph utils
│   │
│   ├── document-analysis-eu/
│   │   ├── DeepDocumentDetailsEU.tsx         ← Port from US
│   │   ├── DeepDocumentUploadEU.tsx          ← Port from US
│   │   ├── DeepDocumentListEU.tsx            ← Port from US
│   │   ├── DeepDocumentLandingEU.tsx         ← Port from US
│   │   ├── ReportHeaderEU.tsx                ← Port from US
│   │   ├── CriticalIssueSectionEU.tsx        ← Port from US
│   │   ├── InsightCardEU.tsx                 ← Port from US
│   │   ├── SummaryAndScoreEU.tsx             ← Port from US
│   │   └── promptBuilderEU.ts                ← Port + adapt
│   │
│   ├── guest-flow-eu/
│   │   ├── GuestInteractionInterfaceEU.tsx   ← NEW (inspired by US)
│   │   ├── GuestProcessingEU.tsx             ← NEW
│   │   ├── GuestAnalyticsEU.tsx              ← NEW
│   │   └── GuestOnboardingModalEU.tsx        ← Port from US
│   │
│   └── shared-eu/
│       ├── TraceGraphTogglerEU.tsx           ← Port from US
│       ├── LineageStepEU.tsx                 ← Port from US
│       └── TraceChainOfThoughtEU.tsx         ← Port from US
│
└── ui/                                   ← Existing shadcn components (shared)
```

### 3.2 EU Query Hooks (NEW)

```
crawlq-ui/src/queries/
├── chat-eu/                              ← Existing
│   ├── useEUSendMessage.ts
│   ├── useEUStreamingMessage.ts
│   └── useEUChatHistoryQuery.ts
│
└── deep-document-eu/                     ← NEW (Phase 1)
    ├── useEUUploadDeepDocument.ts        ← Port from US
    ├── useEUGetDeepDocuments.ts          ← Port from US
    ├── useEUGetDocumentInsights.ts       ← Port from US
    ├── useEUOnboardUser.ts               ← Port from US
    └── types-eu.ts                       ← EU-specific types
```

### 3.3 EU Store Additions

```typescript
// File: crawlq-ui/src/store/chat-eu/useChatEUStore.ts (MODIFY)

interface ChatEUState {
  // ... existing state ...

  // NEW: TRACE UI state
  isKnowledgeGraphEnabled: boolean;
  toggleKnowledgeGraph: () => void;

  // NEW: Document analysis state
  selectedDocument: IWorkspaceDocumentEU | null;
  setSelectedDocument: (doc: IWorkspaceDocumentEU | null) => void;

  // NEW: Guest flow state
  isGuestMode: boolean;
  guestSessionId: string | null;
  setGuestSession: (sessionId: string) => void;
}
```

---

## 4. Porting Strategy

### 4.1 Porting Principles (ADR-013 Compliance)

**STRICT RULES:**
1. ✅ **COPY** US component → NEW EU file (never modify US original)
2. ✅ **RENAME** with `EU` suffix (e.g., `TraceExplainabilityPanel` → `TraceExplainabilityPanelEU`)
3. ✅ **ADAPT** imports to use EU-specific hooks and stores
4. ✅ **PLACE** in `components/trace-eu/` folder (NOT `components/trace/`)
5. ❌ **NEVER** modify any file in US folders (`chat-athena/`, `knowledge-graph/`)

### 4.2 Import Transformation Template

```typescript
// ❌ US ORIGINAL (DO NOT MODIFY)
// File: components/knowledge-graph/trace-explainability/trace-explainability-panel.tsx
import { useSendMessage } from "@/queries/chat-athena/send-message/useSendMessage";
import { useChatInputSettingsStore } from "@/store/useChatInputSettingsStore";

// ✅ EU PORT (NEW FILE)
// File: components/trace-eu/trace-explainability/TraceExplainabilityPanelEU.tsx
import { useEUSendMessage } from "@/queries/chat-eu/useEUSendMessage";
import { useChatEUStore } from "@/store/chat-eu/useChatEUStore";
```

### 4.3 API Endpoint Transformation

```typescript
// US endpoints (READ-ONLY)
const US_UPLOAD_URL = "https://pbzygndqlh4...lambda-url.us-east-2.amazonaws.com/";

// EU endpoints (USE THESE)
const EU_UPLOAD_URL = "https://1v186le2ee.execute-api.eu-central-1.amazonaws.com/upload";
```

### 4.4 Component Naming Convention

| US Component | EU Component | Rationale |
|--------------|--------------|-----------|
| `TraceExplainabilityPanel` | `TraceExplainabilityPanelEU` | Clear EU ownership |
| `TraceKnowledgeGraph` | `TraceKnowledgeGraphEU` | Avoid name collision |
| `DeepDocumentDetails` | `DeepDocumentDetailsEU` | Consistency |
| `promptBuilder.ts` | `promptBuilderEU.ts` | Utility file |

**Alternative (Folder Isolation):** Keep same names but isolate by folder:
- US: `components/knowledge-graph/trace-explainability/TraceExplainabilityPanel`
- EU: `components/trace-eu/trace-explainability/TraceExplainabilityPanel` (same name, different folder)

**Decision:** Use **folder isolation** strategy (no EU suffix needed if in `trace-eu/` folder).

---

## 5. Sprint Breakdown

### Sprint 1: Foundation & Core TRACE Panel (Week 1, Days 1-5)

**Goal:** Port TraceExplainabilityPanel and supporting types

**Tasks:**
1. **Day 1: Setup & Types**
   - Create `components/trace-eu/` folder structure
   - Port `trace-explainability/types/types.ts` → `trace-explainability-eu/types/trace-types-eu.ts`
   - Add EU-specific type extensions:
     ```typescript
     // EU extension: compliance metadata
     export interface TraceResultEU extends TraceResult {
       compliance: {
         euAIActArticles: string[];
         gdprArticles: string[];
         complianceScore: number;
       };
     }
     ```
   - Create `queries/deep-document-eu/types-eu.ts`
   - **Files created:** 2
   - **Lines of code:** ~200

2. **Day 2: ConfidenceBadge + HumanExplanation**
   - Port `ConfidenceBadge` → `trace-eu/trace-explainability-eu/ConfidenceBadgeEU.tsx`
   - Port `HumanExplanation` → `trace-eu/trace-explainability-eu/HumanExplanationEU.tsx`
   - Add EU compliance badge (EU AI Act Art. 50)
   - **Files created:** 2
   - **Lines of code:** ~150

3. **Day 3: TrustSignal + LineageTimeline**
   - Port `TrustSignal` → `trace-eu/trace-explainability-eu/TrustSignalEU.tsx`
   - Port `LineageTimeline` → `trace-eu/trace-explainability-eu/LineageTimelineEU.tsx`
   - Port `LineageStep` → `trace-eu/shared-eu/LineageStepEU.tsx`
   - **Files created:** 3
   - **Lines of code:** ~250

4. **Day 4: KeyFactors + MetricsGrid**
   - Port `KeyFactors` → `trace-eu/trace-explainability-eu/KeyFactorsEU.tsx`
   - Port `MetricsGrid` → `trace-eu/trace-explainability-eu/MetricsGridEU.tsx`
   - Adapt metrics display for EU (6 metrics: Transparency, Reasoning, Audit, Compliance, Explainability, Bias)
   - **Files created:** 2
   - **Lines of code:** ~200

5. **Day 5: TraceExplainabilityPanel Integration**
   - Port `TraceExplainabilityPanel` → `trace-eu/trace-explainability-eu/TraceExplainabilityPanelEU.tsx`
   - Wire all sub-components (ConfidenceBadge, HumanExplanation, TrustSignal, LineageTimeline, KeyFactors, MetricsGrid)
   - Test in Storybook (isolated)
   - **Files created:** 1
   - **Lines of code:** ~150

**Sprint 1 Deliverables:**
- ✅ 10 files created
- ✅ ~950 lines of code
- ✅ TraceExplainabilityPanelEU component fully functional
- ✅ Storybook stories for all sub-components

---

### Sprint 2: Knowledge Graph Visualization (Week 2, Days 6-10)

**Goal:** Port TraceKnowledgeGraph and integrate with EU chat

**Tasks:**
1. **Day 6: Graph Component Port**
   - Port `TraceKnowledgeGraph` → `trace-eu/knowledge-graph-eu/TraceKnowledgeGraphEU.tsx`
   - Port `TraceKnowledgeGraphSkeleton` → `trace-eu/knowledge-graph-eu/TraceKnowledgeGraphSkeletonEU.tsx`
   - Update Neo4j connection to use EU region config
   - **Files created:** 2
   - **Lines of code:** ~300

2. **Day 7: Graph Utilities**
   - Create `trace-eu/knowledge-graph-eu/graph-utils-eu.ts`
   - Port graph data transformation logic
   - Add EU-specific graph node types (compliance nodes, audit nodes)
   - **Files created:** 1
   - **Lines of code:** ~200

3. **Day 8: Toggle Integration**
   - Port `TraceGraphToggler` → `trace-eu/shared-eu/TraceGraphTogglerEU.tsx`
   - Add to `ChatToolbarEU` (existing component in `chat-eu/`)
   - Wire `isKnowledgeGraphEnabled` state to `useChatEUStore`
   - **Files modified:** 2 (ChatToolbarEU, useChatEUStore)
   - **Lines of code:** ~100

4. **Day 9: Chat Integration**
   - Modify `ChatMessageBubbleEU` to conditionally render Knowledge Graph
   - Add "View Trace Graph" button to messages with `trace_result`
   - Use dynamic import for graph (lazy load, reduce bundle size)
   - **Files modified:** 1 (ChatMessageBubbleEU)
   - **Lines of code:** ~80

5. **Day 10: Testing & Overlay**
   - Port overlay/modal logic for full-screen graph view
   - End-to-end test: chat message → Knowledge Graph display
   - Performance test: graph load time < 2s
   - **Files created:** 1 (OverlayEU hook)
   - **Lines of code:** ~100

**Sprint 2 Deliverables:**
- ✅ 4 new files, 3 modified files
- ✅ ~780 lines of code
- ✅ Knowledge Graph integrated with EU chat
- ✅ Graph toggle working in chat toolbar
- ✅ Full-screen graph overlay functional

---

### Sprint 3: Document Analysis UI (Week 3, Days 11-15)

**Goal:** Port full document analysis UI with Fix Strategy

**Tasks:**
1. **Day 11: Document Details Core**
   - Port `DeepDocumentDetails` → `trace-eu/document-analysis-eu/DeepDocumentDetailsEU.tsx`
   - Port `ReportHeader` → `trace-eu/document-analysis-eu/ReportHeaderEU.tsx`
   - Wire with `useEUGetDeepDocuments` (create hook)
   - **Files created:** 3 (2 components + 1 hook)
   - **Lines of code:** ~400

2. **Day 12: Insight Components**
   - Port `CriticalIssueSection` → `trace-eu/document-analysis-eu/CriticalIssueSectionEU.tsx`
   - Port `InsightCard` → `trace-eu/document-analysis-eu/InsightCardEU.tsx`
   - Port `SummaryAndScore` → `trace-eu/document-analysis-eu/SummaryAndScoreEU.tsx`
   - **Files created:** 3
   - **Lines of code:** ~350

3. **Day 13: Fix Strategy Prompt Builder**
   - Port `promptBuilder.ts` → `trace-eu/document-analysis-eu/promptBuilderEU.ts`
   - Port `TRACE_COMPLIANCE_BRAND_VOICE` (86-line prompt)
   - Port `buildUserPromptFromInsight` function
   - Add EU-specific compliance refs (EU AI Act Art. 14, GDPR Art. 22)
   - **Files created:** 1
   - **Lines of code:** ~200

4. **Day 14: "View Fix Strategy" Integration**
   - Wire "View Fix Strategy" button in `InsightCardEU`
   - On click: navigate to `chat-athena-eu` with pre-filled prompt
   - Update `useEUSendMessage` to accept `brandVoiceText` param
   - Test: insight → chat → 8-section remediation response
   - **Files modified:** 2 (InsightCardEU, useEUSendMessage)
   - **Lines of code:** ~150

5. **Day 15: Document List & Upload**
   - Port `DeepDocumentList` → `trace-eu/document-analysis-eu/DeepDocumentListEU.tsx`
   - Port `DeepDocumentUpload` → `trace-eu/document-analysis-eu/DeepDocumentUploadEU.tsx`
   - Create `queries/deep-document-eu/useEUUploadDeepDocument.ts`
   - Test: upload → analysis → insights display
   - **Files created:** 3 (2 components + 1 hook)
   - **Lines of code:** ~300

**Sprint 3 Deliverables:**
- ✅ 10 new files, 2 modified files
- ✅ ~1,400 lines of code
- ✅ Full document analysis UI ported
- ✅ Fix Strategy button working
- ✅ Document upload integrated with EU backend

---

### Sprint 4: Guest Document Flow (Week 4, Days 16-20)

**Goal:** Implement full guest-to-authenticated user conversion flow

**Tasks:**
1. **Day 16: Guest Landing Page**
   - Port `DeepDocumentLanding` → `trace-eu/guest-flow-eu/GuestLandingEU.tsx`
   - Create `app/(public)/guest-document-eu/page.tsx` (NEW PUBLIC ROUTE)
   - Design EU-specific landing copy (GDPR-aware, compliance-focused)
   - **Files created:** 2
   - **Lines of code:** ~250

2. **Day 17: Guest Upload Interface**
   - Create `trace-eu/guest-flow-eu/GuestInteractionInterfaceEU.tsx` (inspired by US)
   - Drag-drop upload component
   - No-auth document upload via `eu_upload_deep_document` Lambda
   - Display "Processing..." animation
   - **Files created:** 1
   - **Lines of code:** ~200

3. **Day 18: Guest Analytics Display**
   - Create `trace-eu/guest-flow-eu/GuestAnalyticsEU.tsx`
   - Display TRACE analysis results (simplified for guests)
   - Show "Unlock Full Report" CTA
   - Use guest sessionId for data retrieval
   - **Files created:** 1
   - **Lines of code:** ~250

4. **Day 19: Guest Onboarding Modal**
   - Port `GuestProjectOnboarding` → `trace-eu/guest-flow-eu/GuestOnboardingModalEU.tsx`
   - On "Unlock" click: show sign-up modal
   - Create `queries/deep-document-eu/useEUOnboardUser.ts`
   - On sign-up: call `eu_onboard_user` Lambda (sessionId → userId)
   - Redirect to authenticated document view
   - **Files created:** 2 (1 component + 1 hook)
   - **Lines of code:** ~300

5. **Day 20: End-to-End Guest Flow Test**
   - Test: landing → upload → analysis → sign-up → full report
   - Verify guest sessionId → userId migration
   - Verify document ownership transfer
   - Performance: guest upload → results < 30s
   - **Files created:** 0 (testing only)
   - **Tests written:** 10

**Sprint 4 Deliverables:**
- ✅ 6 new files
- ✅ ~1,000 lines of code
- ✅ Full guest flow working end-to-end
- ✅ Guest → authenticated user conversion tested
- ✅ Public route `/guest-document-eu` live

---

### Sprint 5: Polish, Testing, Documentation (Week 5, Days 21-25)

**Goal:** Final integration, testing, documentation, COMMIT 10

**Tasks:**
1. **Day 21: Integration Testing**
   - Test all 21 ported components in EU environment
   - Verify no US code modified (git diff check)
   - Verify no conflicts with ADR-012 branch (markdown renderer isolated)
   - Fix any integration bugs
   - **Files modified:** 0-3 (bug fixes only)
   - **Tests written:** 20

2. **Day 22: Responsive Design & Mobile**
   - Test all TRACE UI components on mobile (320px - 768px)
   - Fix any mobile layout issues
   - Ensure Knowledge Graph mobile-friendly (touch gestures)
   - **Files modified:** 5-10 (CSS/layout fixes)
   - **Lines of code:** ~200

3. **Day 23: Performance Optimization**
   - Lazy load Knowledge Graph (reduce initial bundle)
   - Optimize TRACE panel rendering (React.memo, useMemo)
   - Reduce API calls (caching with React Query)
   - Target: Lighthouse score > 90
   - **Files modified:** 5-8
   - **Lines of code:** ~150

4. **Day 24: Documentation**
   - Create `docs/trace-eu-components.md` (component catalog)
   - Update `README.md` with EU TRACE UI instructions
   - Document porting process for future reference
   - Create Storybook documentation
   - **Files created:** 3 (docs)
   - **Lines of code:** ~500 (markdown)

5. **Day 25: COMMIT 10 & Deployment**
   - Final git commit on `feature-eu-chat-athena` branch
   - Create COMMIT 10 in `.gcc/branches/feature-eu-chat-athena/commit.md`
   - Deploy to Amplify (feature/trace-eu-frontend)
   - Smoke test all features in deployed environment
   - Prepare handoff notes for ADR-012 merge
   - **Files modified:** 2 (commit.md, log.md)

**Sprint 5 Deliverables:**
- ✅ All 21 components tested and documented
- ✅ Mobile-responsive design verified
- ✅ Performance optimized (Lighthouse > 90)
- ✅ COMMIT 10 created
- ✅ Deployed to Amplify

---

## 6. Conflict Avoidance with ADR-012

### 6.1 ADR-012 Scope (Separate Branch Work)

**Branch:** `feature-tier3-async-markdown`
**Parent:** `feature-eu-chat-athena` (this branch)
**Owner:** Separate developer/session
**Scope:**
- Tier 3 async job queue (SQS, Lambda worker, DynamoDB job tracking)
- Enterprise markdown renderer (`EnterpriseMarkdownRenderer.tsx`)
- Intelligent markdown processor (backend Python module)
- Job polling hook (`useJobPolling.ts`)

### 6.2 Potential File Conflicts

| File | This Branch (Phase 1) | ADR-012 Branch | Resolution |
|------|----------------------|----------------|------------|
| `ChatMarkdownRenderer.tsx` | ✅ Keep basic version | ✅ Replace with enterprise version | **ADR-012 OWNS** (this branch won't modify) |
| `ChatContainer.tsx` | ⚠️ May add TRACE panel integration | ⚠️ May add job polling | **Merge conflict expected** (resolve on ADR-012 merge) |
| `useEUStreamingMessage.ts` | ❌ No changes | ✅ Add job polling logic | **ADR-012 OWNS** (no conflict) |
| `region-config.ts` | ⚠️ May add document upload URLs | ⚠️ May add job queue URLs | **Merge conflict expected** (easy to resolve) |

### 6.3 Conflict Avoidance Strategy

**RULE:** This branch (Phase 1) will NOT modify:
1. ❌ `ChatMarkdownRenderer.tsx` (ADR-012 owns this)
2. ❌ `EnterpriseMarkdownRenderer.tsx` (ADR-012 creates this)
3. ❌ `useEUStreamingMessage.ts` (ADR-012 modifies this)
4. ❌ `useJobPolling.ts` (ADR-012 creates this)

**ALLOWED:** This branch CAN modify:
1. ✅ `ChatContainer.tsx` (document merge conflict on ADR-012 merge)
2. ✅ `region-config.ts` (document merge conflict on ADR-012 merge)
3. ✅ `useChatEUStore.ts` (add TRACE state, ADR-012 adds job state)

**Merge Strategy:**
- ADR-012 merges back into `feature-eu-chat-athena` when Tier 3 is complete
- Manual conflict resolution: `ChatContainer.tsx`, `region-config.ts`, `useChatEUStore.ts`
- ADR-012 wins on: markdown rendering, job polling, async architecture
- This branch wins on: TRACE UI, document analysis, guest flow

---

## 7. Success Metrics

### 7.1 Code Metrics

| Metric | Target | Measure |
|--------|--------|---------|
| **Components Created** | 21 | Count files in `trace-eu/` |
| **Lines of Code (LoC)** | ~4,500 | `cloc components/trace-eu/` |
| **Test Coverage** | > 80% | Jest coverage report |
| **Bundle Size Increase** | < 500KB | Webpack bundle analyzer |
| **TypeScript Errors** | 0 | `tsc --noEmit` |
| **ESLint Warnings** | < 10 | `eslint components/trace-eu/` |

### 7.2 Functional Metrics

| Feature | Target | Test Method |
|---------|--------|-------------|
| **TRACE Panel Display** | Renders in < 2s | Lighthouse perf audit |
| **Knowledge Graph Load** | Renders in < 3s | Manual + Lighthouse |
| **Guest Upload → Results** | < 30s end-to-end | Manual E2E test |
| **Fix Strategy Generation** | 8 sections, 800-1200 words | Manual test |
| **Mobile Responsive** | Works 320px-768px | BrowserStack test |
| **API Gateway Timeout** | No 503 errors | 20 test queries |

### 7.3 Compliance Metrics (ADR-013)

| Requirement | Target | Verification |
|-------------|--------|--------------|
| **US Files Modified** | 0 | `git diff` check |
| **US Imports Used** | 0 | ESLint custom rule |
| **EU Folder Isolation** | 100% | All new files in `trace-eu/` |
| **Naming Convention** | 100% | Matches `*EU.tsx` or folder isolation |

### 7.4 Integration Metrics (with ADR-012)

| Metric | Target | Verification |
|--------|--------|--------------|
| **File Conflicts on Merge** | < 5 | Git merge simulation |
| **Shared File Modifications** | < 3 | Document in commit log |
| **Markdown Renderer Conflicts** | 0 | This branch doesn't modify |
| **Post-Merge Test Pass Rate** | 100% | Run full test suite after merge |

### 7.5 User Experience Metrics

| Metric | Target | Measure |
|--------|--------|---------|
| **Guest Conversion Rate** | > 30% | Analytics (guest sign-ups / uploads) |
| **TRACE Panel Engagement** | > 50% | Analytics (panel opens / messages) |
| **Knowledge Graph Views** | > 40% | Analytics (graph views / documents) |
| **Fix Strategy Clicks** | > 60% | Analytics (clicks / insights) |
| **Mobile Usage** | > 20% | Analytics (mobile sessions / total) |

---

## 8. Risk Mitigation

### 8.1 Technical Risks

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| **Neo4j Graph API Changes** | Medium | High | Pin Neo4j NVL version, test early |
| **API Gateway Timeout (30s)** | High | Medium | Use existing Tier 1 solution (COMMIT 9), ADR-012 Tier 3 later |
| **Bundle Size Explosion** | Medium | Medium | Lazy load, code splitting, tree shaking |
| **Mobile Performance** | Medium | High | Test on real devices early (Day 22) |
| **Type Mismatches (US → EU)** | Low | Low | Strict TypeScript, early testing |

### 8.2 Process Risks

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| **Accidental US Modification** | Low | Critical | Pre-commit hook (ADR-013), code review |
| **Merge Conflict with ADR-012** | High | Medium | Document shared files, manual merge plan |
| **Scope Creep** | Medium | Medium | Stick to 21-component scope, defer enhancements |
| **Testing Delays** | Medium | High | Parallel testing (Days 21-22), automate |

### 8.3 User Experience Risks

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| **Guest Flow Confusing** | Medium | High | User testing (Day 18), iterate copy |
| **TRACE Panel Overwhelming** | Medium | Medium | Progressive disclosure, tooltips |
| **Mobile UX Poor** | Low | High | Mobile-first design (Day 22) |

---

## 9. Dependencies & Assumptions

### 9.1 Technical Dependencies

✅ **Available Now:**
- 17 EU Lambda functions deployed and operational (COMMIT 8)
- API Gateway with JWT authorizer (COMMIT 6)
- Chat infrastructure (streaming, history, memory) (COMMIT 7)
- Neo4j graph database (shared US/EU instance in eu-north-1)

⚠️ **Phase 1 Blockers (None Expected):**
- All required backend APIs exist
- Frontend tech stack unchanged (Next.js 14, React Query, Zustand)

### 9.2 Assumptions

1. **US code remains stable** — No major refactors in US TRACE UI during Phase 1
2. **ADR-012 branch separate** — No unplanned merges during Phase 1
3. **Neo4j available** — Graph database remains accessible and performant
4. **Amplify auto-deploy** — Push to `feature/trace-eu-frontend` triggers build
5. **Backend stable** — No breaking changes to 17 EU Lambda functions

---

## 10. Next Steps After Phase 1

**Phase 2 (Weeks 6-9): Beyond Parity**
- Merge ADR-012 (Tier 3 async architecture)
- Implement Compliance Passport (shareable certificate)
- FrictionMelt integration (3 new Lambdas)
- EU branding and landing page redesign
- Full test coverage (unit + E2E + visual regression)

**Phase 3 (Weeks 10-12): Advanced Features**
- Upgrade to 5-tier confidence system (Green/Blue/Orange/Red/Maroon)
- Hybrid RAG (graph traversal + vector search)
- SHACL validation (data quality constraints)
- Infrastructure as Code (CloudFormation templates)
- Audit trail persistence (7-year immutable storage)

---

## Appendix A: File Checklist

### Files to Create (Phase 1)

**trace-eu/trace-explainability-eu/** (10 files)
- [ ] `TraceExplainabilityPanelEU.tsx`
- [ ] `ConfidenceBadgeEU.tsx`
- [ ] `HumanExplanationEU.tsx`
- [ ] `TrustSignalEU.tsx`
- [ ] `LineageTimelineEU.tsx`
- [ ] `KeyFactorsEU.tsx`
- [ ] `MetricsGridEU.tsx`
- [ ] `AuditFooterEU.tsx`
- [ ] `types/trace-types-eu.ts`
- [ ] `index.ts` (barrel export)

**trace-eu/knowledge-graph-eu/** (4 files)
- [ ] `TraceKnowledgeGraphEU.tsx`
- [ ] `TraceKnowledgeGraphSkeletonEU.tsx`
- [ ] `graph-utils-eu.ts`
- [ ] `index.ts`

**trace-eu/document-analysis-eu/** (11 files)
- [ ] `DeepDocumentDetailsEU.tsx`
- [ ] `DeepDocumentUploadEU.tsx`
- [ ] `DeepDocumentListEU.tsx`
- [ ] `DeepDocumentLandingEU.tsx`
- [ ] `ReportHeaderEU.tsx`
- [ ] `CriticalIssueSectionEU.tsx`
- [ ] `InsightCardEU.tsx`
- [ ] `SummaryAndScoreEU.tsx`
- [ ] `promptBuilderEU.ts`
- [ ] `types-eu.ts`
- [ ] `index.ts`

**trace-eu/guest-flow-eu/** (5 files)
- [ ] `GuestLandingEU.tsx`
- [ ] `GuestInteractionInterfaceEU.tsx`
- [ ] `GuestAnalyticsEU.tsx`
- [ ] `GuestOnboardingModalEU.tsx`
- [ ] `index.ts`

**trace-eu/shared-eu/** (4 files)
- [ ] `TraceGraphTogglerEU.tsx`
- [ ] `LineageStepEU.tsx`
- [ ] `TraceChainOfThoughtEU.tsx`
- [ ] `index.ts`

**queries/deep-document-eu/** (5 files)
- [ ] `useEUUploadDeepDocument.ts`
- [ ] `useEUGetDeepDocuments.ts`
- [ ] `useEUGetDocumentInsights.ts`
- [ ] `useEUOnboardUser.ts`
- [ ] `types-eu.ts`

**app/(public)/guest-document-eu/** (1 file)
- [ ] `page.tsx`

**Total: 40 new files**

---

## Appendix B: API Endpoints Reference

### EU Backend Endpoints (API Gateway)

```yaml
Base URL: https://1v186le2ee.execute-api.eu-central-1.amazonaws.com

# Chat endpoints (existing)
POST   /chat              → eu_chat_athena_bot
GET    /chat-history      → eu_get_chat_history

# Document upload (existing Lambda, need API Gateway route)
POST   /upload            → eu_upload_deep_document (guest + authenticated)

# Document retrieval (existing Lambda, need API Gateway route)
GET    /documents         → eu_get_deep_documents

# Document insights (existing Lambda, need API Gateway route)
GET    /insights/{documentId}  → eu_get_document_insights

# User onboarding (existing Lambda, need API Gateway route)
POST   /onboard           → eu_onboard_user

# TRACE explainer (existing Lambda, need API Gateway route)
POST   /trace-explain     → eu_trace_explainer
```

**NOTE:** Some endpoints may need to be added to API Gateway (Phase 1, Day 11).

---

## Appendix C: Git Branch Strategy

```
main (production)
└── feature-eu-chat-athena (this branch, COMMIT 9)
    ├── feature-tier3-async-markdown (ADR-012, separate work)
    │   └── Merges back into feature-eu-chat-athena when complete
    └── Phase 1 work happens here (21 TRACE UI components)
        └── COMMIT 10 (end of Phase 1)
```

**Merge Plan:**
1. Complete Phase 1 (this plan) → COMMIT 10
2. ADR-012 completes Tier 3 async → merges into `feature-eu-chat-athena`
3. Resolve merge conflicts (ChatContainer, region-config, useChatEUStore)
4. Test merged branch → COMMIT 11
5. Merge `feature-eu-chat-athena` into `main` (production release)

---

**PHASE 1 READY TO BEGIN 🚀**

**Next Action:** Start Sprint 1, Day 1 (Foundation & Types)
