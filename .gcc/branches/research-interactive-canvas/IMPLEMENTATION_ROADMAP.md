# TRACE Canvas — Implementation Roadmap

**Version:** 1.0
**Date:** 2026-02-12
**Timeline:** 12 weeks (3 months)
**Team Size:** 2 engineers + 1 designer
**Status:** PROPOSED

---

## Executive Summary

This roadmap details the **phased implementation** of TRACE Canvas over 12 weeks. The approach prioritizes:

1. **Early value delivery** — MVP workflow builder in Week 3
2. **Incremental complexity** — add modes/features progressively
3. **User feedback loops** — beta testing after each phase
4. **Risk mitigation** — validate technical assumptions early (spikes in Week 1)

**Key Milestones:**
- **Week 3:** MVP Workflow Builder (private beta)
- **Week 6:** Freeform Canvas + Markdown Overlays (public beta)
- **Week 9:** Knowledge Graph 3D + Advanced Visualizations (general availability)
- **Week 12:** Export Engine + Polish (production ready)

**Post-launch (Month 4+):** Real-time collaboration (Phase 6)

---

## Development Phases

### **Phase 0: Research & Validation** (Week 1)

**Goal:** Validate technical assumptions, set up infrastructure.

#### Spike 1: React Flow Integration (2 days)
**Objective:** Prove React Flow works with our Next.js setup.

**Tasks:**
- [ ] Install `@xyflow/react`
- [ ] Create basic workflow with 3 nodes (Input → LLM → Output)
- [ ] Test data flow (JSON payload passing between nodes)
- [ ] Measure performance (render 100 nodes)
- [ ] Document findings

**Success Criteria:** 100 nodes render in <200ms.

---

#### Spike 2: Tldraw Integration (1 day)
**Objective:** Test Tldraw infinite canvas in Next.js.

**Tasks:**
- [ ] Install `tldraw`
- [ ] Create basic canvas with shapes and text
- [ ] Test export to PNG/SVG
- [ ] Check license requirements (watermark)
- [ ] Document findings

**Success Criteria:** Smooth pan/zoom, PNG export works.

---

#### Spike 3: Reagraph 3D Graph (1 day)
**Objective:** Validate 3D graph performance.

**Tasks:**
- [ ] Install `reagraph`
- [ ] Render 1000-node graph
- [ ] Test on GPU (desktop) and no-GPU (mobile)
- [ ] Measure FPS (target: 60fps)
- [ ] Document fallback strategy (Cytoscape.js)

**Success Criteria:** 1000 nodes at 60fps on GPU, graceful fallback without GPU.

---

#### Spike 4: Visx Custom Charts (1 day)
**Objective:** Build custom TRACE radar chart.

**Tasks:**
- [ ] Install `@visx/group`, `@visx/scale`, `@visx/shape`
- [ ] Build radar chart component (5 dimensions)
- [ ] Test responsiveness (mobile, tablet, desktop)
- [ ] Measure bundle impact (~40KB expected)

**Success Criteria:** Radar chart renders correctly, bundle size acceptable.

---

#### Infrastructure Setup (Parallel, 2 days)
**Tasks:**
- [ ] Create DynamoDB table: `trace-canvas-documents`
- [ ] Create S3 bucket: `trace-canvas-exports`
- [ ] Set up CloudFront for static assets
- [ ] Create Lambda function stubs (execute-workflow, save-document, load-document)
- [ ] Configure IAM roles and permissions

**Success Criteria:** Infrastructure ready for development.

---

### **Phase 1: MVP Workflow Builder** (Week 2-3)

**Goal:** Ship minimal viable workflow builder with 3 node types.

#### Week 2: Core Workflow Engine

**Day 1-2: React Flow Setup**
- [ ] Create `/app/canvas/page.tsx` route
- [ ] Install React Flow dependencies
- [ ] Set up basic canvas with grid background
- [ ] Add minimap and controls (zoom, fit view)

**Day 3-4: Node Components**
- [ ] Build `InputNode.tsx` (upload file, enter text)
- [ ] Build `LLMNode.tsx` (select model: Claude, GPT-4, Gemini)
- [ ] Build `OutputNode.tsx` (display results)
- [ ] Add node configuration panel (sidebar)

**Day 5: Edge/Connector Logic**
- [ ] Implement data flow (JSON payloads)
- [ ] Validate connections (Input must connect to LLM, LLM to Output)
- [ ] Add edge labels (show data type)

---

#### Week 3: Workflow Execution + Beta Launch

**Day 1-2: Execution Engine**
- [ ] Create `workflowExecutor.ts` (traverse nodes, execute in order)
- [ ] Integrate with existing Lambda (chat-athena-eu/send-message)
- [ ] Handle errors (red node border on failure)
- [ ] Show progress (green = success, yellow = in-progress, red = error)

**Day 3: State Management**
- [ ] Set up Zustand store (`canvasState.ts`)
- [ ] Implement save/load workflow (DynamoDB)
- [ ] Add undo/redo (React Flow built-in)

**Day 4: UI Polish**
- [ ] Add toolbar (Run, Save, Load, Clear)
- [ ] Add keyboard shortcuts (Cmd+S = save, Cmd+Z = undo)
- [ ] Responsive design (hide minimap on mobile)

**Day 5: Testing + Private Beta**
- [ ] Unit tests (node rendering, execution logic)
- [ ] Integration test (end-to-end workflow)
- [ ] **Launch private beta** — invite 10 users
- [ ] Set up feedback form (Typeform/Google Forms)

**Success Criteria:**
- 10 beta users create workflows
- 80% success rate on workflow execution
- 5+ pieces of feedback collected

---

### **Phase 2: Markdown Overlays + Freeform Canvas** (Week 4-6)

#### Week 4: Markdown Card System

**Day 1-2: Markdown Rendering**
- [ ] Create `MarkdownCard.tsx` (HTML overlay on canvas)
- [ ] Position using absolute positioning (x, y from canvas)
- [ ] Integrate `react-markdown` + `remark-gfm`
- [ ] Add TRACE confidence badge to card

**Day 3: Wikilink Parser**
- [ ] Parse markdown AST (unified + remark-parse)
- [ ] Extract `[[wikilinks]]` and `#tags`
- [ ] Auto-create connectors between linked cards
- [ ] Build backlinks panel (which cards link here)

**Day 4: Tag Filtering**
- [ ] Add tag filter dropdown (filter by #tag)
- [ ] Highlight filtered cards
- [ ] Hide non-matching cards

**Day 5: Testing**
- [ ] Test 100-card canvas (performance)
- [ ] Test wikilink parsing edge cases
- [ ] User testing with beta group

---

#### Week 5: Freeform Canvas (Tldraw)

**Day 1-2: Tldraw Integration**
- [ ] Install `tldraw` and configure
- [ ] Create `FreeformCanvas.tsx` wrapper
- [ ] Add to mode switcher (Workflow | Freeform)
- [ ] Test tools (pen, shapes, text, sticky notes, arrows)

**Day 3: Auto-Clustering**
- [ ] Build `AutoCluster.tsx` (AI clusters sticky notes by tags)
- [ ] Call Lambda: `canvas-ai-cluster` (LLM groups notes)
- [ ] Visualize clusters (color-coded borders)

**Day 4: Export to PNG/SVG**
- [ ] Implement Tldraw export API
- [ ] Add "Export Canvas" button
- [ ] Test high-resolution export (2x for retina)

**Day 5: Testing + Iteration**
- [ ] Test annotation layer over PDFs
- [ ] Beta testing (collect feedback on freeform mode)

---

#### Week 6: Mode Switcher + Public Beta

**Day 1-2: Mode Switcher UI**
- [ ] Build `ModeSelector.tsx` (radio buttons: Workflow | Freeform | Graph)
- [ ] Lazy load modes (code splitting)
- [ ] Persist mode selection (localStorage)

**Day 3: Data Migration**
- [ ] Handle mode switching (convert workflow to freeform?)
- [ ] Prompt user: "Switching modes will clear canvas. Continue?"

**Day 4: UI Polish**
- [ ] Add onboarding wizard (first-time users)
- [ ] Create video tutorial (2 min demo)
- [ ] Update docs (canvas user guide)

**Day 5: Public Beta Launch**
- [ ] **Launch to all users**
- [ ] Announce on social media, email newsletter
- [ ] Monitor analytics (Mixpanel/PostHog)

**Success Criteria:**
- 50+ canvas documents created
- 30% of active users try canvas feature
- NPS score >40

---

### **Phase 3: Knowledge Graph + Advanced Visualizations** (Week 7-9)

#### Week 7: 3D Knowledge Graph (Reagraph)

**Day 1-2: Reagraph Integration**
- [ ] Install `reagraph`
- [ ] Create `KnowledgeGraph3D.tsx` component
- [ ] Integrate with existing graph data (from document analysis)
- [ ] Add to mode switcher (Workflow | Freeform | Graph)

**Day 3: Interactions**
- [ ] Implement node click → TRACE detail panel
- [ ] Add zoom/pan/rotate controls
- [ ] Filter by entity type (risk, entity, policy)

**Day 4: 2D Fallback (Cytoscape.js)**
- [ ] Install `cytoscape`
- [ ] Create `KnowledgeGraph2D.tsx` fallback
- [ ] GPU detection (use Reagraph if GPU, else Cytoscape)
- [ ] Test on mobile (should fallback to 2D)

**Day 5: Testing**
- [ ] Performance test (10K nodes)
- [ ] Cross-browser testing (Chrome, Firefox, Safari)
- [ ] Accessibility audit (keyboard navigation, ARIA labels)

---

#### Week 8: Advanced Visualizations (Visx)

**Day 1: Radar Chart**
- [ ] Build `TraceRadarChart.tsx` (5 TRACE dimensions)
- [ ] Integrate into TRACE panel (expand from badge)
- [ ] Test responsiveness (mobile, tablet, desktop)

**Day 2: Heatmap**
- [ ] Build `ComplianceHeatmap.tsx` (Article × Risk Level)
- [ ] Color scale (red = high risk, orange = medium, green = low)
- [ ] Hover tooltip (show risk details)

**Day 3: Sankey Diagram**
- [ ] Build `DataLineageSankey.tsx` (Sources → Processing → Outputs)
- [ ] Width = data volume
- [ ] Hover flow → show data count

**Day 4: ECharts Real-Time Gauge**
- [ ] Install `echarts-for-react`
- [ ] Build `ConfidenceGaugeRealTime.tsx`
- [ ] Animate confidence score (count-up effect)
- [ ] Test real-time updates (WebSocket simulation)

**Day 5: Testing**
- [ ] Visual regression tests (Chromatic/Percy)
- [ ] Performance tests (render time <100ms)
- [ ] Accessibility audit

---

#### Week 9: Integration + General Availability

**Day 1-2: Embed Visualizations in Canvas**
- [ ] Allow dragging charts onto canvas (as nodes)
- [ ] Resize charts (drag handles)
- [ ] Update charts on data change (reactive)

**Day 3: Workflow Templates**
- [ ] Build 5 templates (Literature Review, Competitor Analysis, Compliance Audit, etc.)
- [ ] Add "Use Template" button on workflow canvas
- [ ] Pre-populate nodes with template config

**Day 4: UI Polish**
- [ ] Dark mode support (all visualizations)
- [ ] Loading states (skeletons for charts)
- [ ] Error states (fallback UI)

**Day 5: General Availability Launch**
- [ ] **Announce GA** (blog post, social media)
- [ ] Press release (TechCrunch, Product Hunt)
- [ ] Monitor analytics, user feedback

**Success Criteria:**
- 100+ canvas documents created in Week 1
- 10+ users create workflows using templates
- 5+ 3D graphs explored

---

### **Phase 4: Export Engine + Production Hardening** (Week 10-12)

#### Week 10: PDF/DOCX Export

**Day 1-2: PDF Exporter**
- [ ] Install `jsPDF` + `html2canvas`
- [ ] Build `PDFExporter.tsx`
- [ ] Render canvas to image (high DPI, 2x resolution)
- [ ] Add title page, TRACE audit trail, metadata
- [ ] Test multi-page PDFs (long canvases)

**Day 3: DOCX Exporter**
- [ ] Install `docx.js`
- [ ] Build `DOCXExporter.tsx`
- [ ] Embed canvas image as PNG
- [ ] Add markdown text as editable text
- [ ] Add TRACE table (5 dimensions)

**Day 4: Markdown/JSON Export**
- [ ] Build `MarkdownExporter.tsx` (preserve wikilinks)
- [ ] Build `JSONExporter.tsx` (full canvas state)
- [ ] Test import/export round-trip (lossless)

**Day 5: Testing**
- [ ] Export 100-node workflow to PDF (verify quality)
- [ ] Export freeform canvas to DOCX (verify layout)
- [ ] Test on Windows/Mac (Word compatibility)

---

#### Week 11: Production Hardening

**Day 1: Error Handling**
- [ ] Add try-catch blocks to all async operations
- [ ] Display user-friendly error messages
- [ ] Log errors to CloudWatch
- [ ] Add Sentry error tracking

**Day 2: Performance Optimization**
- [ ] Lazy load visualizations (code splitting)
- [ ] Memoize expensive computations (React.memo, useMemo)
- [ ] Throttle real-time updates (60fps max)
- [ ] Virtualize large lists (react-window)

**Day 3: Security Audit**
- [ ] Validate all user inputs (prevent XSS, injection)
- [ ] Sanitize markdown (no script tags)
- [ ] Rate limit workflow execution (prevent abuse)
- [ ] CSRF protection

**Day 4: Accessibility Audit**
- [ ] WCAG 2.1 AA compliance check (axe DevTools)
- [ ] Keyboard navigation testing
- [ ] Screen reader testing (NVDA, JAWS)
- [ ] Color contrast audit (4.5:1 minimum)

**Day 5: Load Testing**
- [ ] Simulate 100 concurrent users (Artillery, k6)
- [ ] Test DynamoDB throughput (scale on-demand)
- [ ] Test Lambda cold starts (pre-warming)

---

#### Week 12: Final Polish + Documentation

**Day 1-2: UI Polish**
- [ ] Add animations (smooth transitions)
- [ ] Improve loading states (skeletons, progress bars)
- [ ] Add empty states ("No canvases yet, create one!")
- [ ] Mobile optimization (touch-friendly, responsive)

**Day 3: Documentation**
- [ ] User guide (canvas.md in docs)
- [ ] Video tutorials (workflow builder, freeform canvas, 3D graph)
- [ ] API docs (for developers integrating workflows)
- [ ] FAQ (common questions)

**Day 4: Marketing Assets**
- [ ] Landing page (canvas feature page)
- [ ] Demo video (90 sec, post on YouTube)
- [ ] Blog post (launch announcement)
- [ ] Social media posts (Twitter, LinkedIn)

**Day 5: Production Launch**
- [ ] **Production deployment** (Amplify + Lambda)
- [ ] Monitor analytics (PostHog, Mixpanel)
- [ ] On-call rotation (24/7 monitoring)
- [ ] Celebrate 🎉

---

## Post-Launch (Month 4+)

### **Phase 5: Real-Time Collaboration** (Week 13-16)

**Goal:** Add real-time collaboration (live cursors, simultaneous editing).

#### Week 13-14: Liveblocks Integration
- [ ] Install `@liveblocks/client` + `@liveblocks/react`
- [ ] Set up Liveblocks account (API key)
- [ ] Integrate CRDT sync (canvas state)
- [ ] Add presence (live cursors, user avatars)

#### Week 15: Conflict Resolution
- [ ] Test simultaneous edits (two users edit same node)
- [ ] Implement CRDT merge strategy
- [ ] Add undo/redo with collaboration

#### Week 16: Comments + Permissions
- [ ] Add comment threads (click node → add comment)
- [ ] Add permissions (private, shared, public)
- [ ] Add version history (revert to previous state)
- [ ] Launch collaboration feature

---

### **Phase 6: Mobile App** (Month 5-6)

**Goal:** React Native app for mobile canvas editing.

- [ ] Set up React Native project
- [ ] Port canvas components (React Flow, Tldraw)
- [ ] Touch-optimized UI (gestures, pinch-to-zoom)
- [ ] Offline mode (IndexedDB sync)
- [ ] Launch on iOS and Android

---

## Resource Allocation

### Team Structure

| Role | Allocation | Responsibilities |
|------|------------|------------------|
| **Frontend Engineer 1** | Full-time (12 weeks) | React Flow, Tldraw, Reagraph integration |
| **Frontend Engineer 2** | Full-time (12 weeks) | Visx charts, export engine, UI polish |
| **Backend Engineer** | 50% (6 weeks) | Lambda functions, DynamoDB, API integration |
| **UX Designer** | 50% (6 weeks) | Wireframes, user testing, onboarding wizard |
| **QA Engineer** | 25% (3 weeks) | Testing, bug triage, accessibility audit |
| **Product Manager** | 25% (3 weeks) | Roadmap, prioritization, user interviews |

**Total Effort:** ~18 person-weeks (frontend) + 6 (backend) + 3 (design) + 3 (QA) + 3 (PM) = **33 person-weeks**

---

## Budget Estimate

| Category | Cost | Notes |
|----------|------|-------|
| **Tldraw License** | $5,000/year | Commercial license (remove watermark) |
| **Liveblocks Subscription** | $200/month (Phase 6) | Real-time collaboration (CRDT) |
| **AWS Infrastructure** | $500/month | DynamoDB, S3, CloudFront, Lambda |
| **Third-Party APIs** | $300/month | Anthropic API (LLM calls in workflows) |
| **Tools & Software** | $500/month | Figma, Chromatic (visual regression), Sentry |
| **Total (Year 1)** | $23,600 | $5K upfront + $1.5K/month recurring |

---

## Risk Register

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| **React Flow performance issues (10K+ nodes)** | High | Medium | Implement virtualization, pagination |
| **Tldraw license negotiation fails** | Medium | Low | Fallback to Excalidraw (MIT license) |
| **Reagraph 3D doesn't work on low-end devices** | Medium | Medium | Auto-fallback to 2D (Cytoscape.js) |
| **Users find canvas too complex** | High | Medium | Provide templates, onboarding wizard, video tutorials |
| **Low adoption (users don't discover feature)** | High | Medium | Prominent placement in UI, marketing campaign |
| **Export quality poor (blurry images)** | Medium | Low | Use 2x resolution (retina), test on multiple devices |
| **Real-time collaboration too expensive (Liveblocks)** | Low | Low | Evaluate Yjs (open-source CRDT) as alternative |

---

## Success Metrics (OKRs)

### Objective 1: Ship Canvas MVP on Time
**KR1:** Phase 1 complete by Week 3 (private beta)
**KR2:** Phase 2 complete by Week 6 (public beta)
**KR3:** Phase 3 complete by Week 9 (GA)

### Objective 2: Drive Adoption
**KR1:** 100+ canvas documents created in Month 1
**KR2:** 30% of active users try canvas feature
**KR3:** 50+ workflows executed weekly

### Objective 3: Validate Product-Market Fit
**KR1:** NPS score >40
**KR2:** 10+ user testimonials
**KR3:** 5+ users migrate from competitor tools (Miro, Obsidian, Litmaps)

### Objective 4: Demonstrate Business Impact
**KR1:** 20% increase in Pro plan conversions (canvas as differentiator)
**KR2:** $10K+ ARR attributed to canvas feature
**KR3:** 80% of Enterprise customers use canvas

---

## Dependencies & Blockers

### External Dependencies
- **Tldraw license approval** — requires budget approval from Finance
- **Liveblocks account** (Phase 6) — requires vendor evaluation
- **AWS quota increases** — may need higher DynamoDB throughput

### Internal Dependencies
- **Design system updates** — canvas components must match brand
- **Backend API stability** — Lambda functions must handle increased load
- **Documentation team** — user guide, video tutorials

### Known Blockers
- ❌ **None currently** (spikes in Week 1 will identify technical blockers)

---

## Communication Plan

### Weekly Standups (Monday, Wednesday, Friday)
- **Duration:** 15 min
- **Attendees:** Engineers, Designer, PM
- **Agenda:** Progress updates, blockers, next steps

### Sprint Reviews (Every 2 weeks)
- **Duration:** 1 hour
- **Attendees:** Full team + stakeholders
- **Agenda:** Demo completed features, gather feedback

### User Feedback Sessions (Every 2 weeks)
- **Duration:** 30 min
- **Attendees:** PM + 2-3 beta users
- **Agenda:** Usability testing, feature requests, bug reports

### Launch Retrospective (Week 13)
- **Duration:** 2 hours
- **Attendees:** Full team
- **Agenda:** What went well, what didn't, lessons learned

---

## Rollback Plan

If a phase fails or causes critical issues:

### Phase 1 Rollback
- **Trigger:** >50% workflow execution failures
- **Action:** Disable workflow execution, show "coming soon" message
- **Timeline:** 1 day to rollback

### Phase 2 Rollback
- **Trigger:** Tldraw causes browser crashes
- **Action:** Disable freeform mode, revert to workflow-only
- **Timeline:** 1 day to rollback

### Phase 3 Rollback
- **Trigger:** 3D graph causes performance issues on >30% of devices
- **Action:** Disable 3D mode, force 2D fallback
- **Timeline:** 1 hour to toggle feature flag

### Full Canvas Rollback
- **Trigger:** Critical security vulnerability discovered
- **Action:** Disable entire canvas feature, investigate issue
- **Timeline:** Immediate (feature flag toggle)

---

## Definition of Done

A feature is "done" when:

- [ ] **Code complete** — all code written, reviewed, merged
- [ ] **Tests passing** — unit, integration, E2E tests green
- [ ] **Design approved** — UX designer signs off on UI
- [ ] **Accessibility audit** — WCAG 2.1 AA compliant
- [ ] **Performance benchmarks met** — <2s load time, 60fps interactions
- [ ] **Documentation written** — user guide, API docs
- [ ] **Beta tested** — 10+ users test feature, feedback incorporated
- [ ] **Analytics instrumented** — track usage, errors, conversions
- [ ] **Deployed to production** — feature live for all users

---

## Related Documents

- [Research Report](./RESEARCH_REPORT.md)
- [Feature Specification](./TRACE_CANVAS_SPEC.md)
- [ADR-026: Canvas Technology](../../.gsm/decisions/ADR-026-interactive-canvas-technology.md)
- [ADR-027: Visualization Strategy](../../.gsm/decisions/ADR-027-visualization-strategy.md)

---

**Roadmap Status:** PROPOSED (awaiting approval)
**Next Action:** Present roadmap to product/engineering leads, gather feedback, get buy-in.

---

**End of Implementation Roadmap**
