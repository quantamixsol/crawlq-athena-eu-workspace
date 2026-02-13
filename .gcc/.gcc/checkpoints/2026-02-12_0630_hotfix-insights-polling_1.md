### COMMIT 1 — 2026-02-12T06:30:00Z
**Branch:** hotfix-insights-polling
**Milestone:** Sprint 1 complete — insights polling for async document analysis
**State:** DONE
**Files Changed:**
- MODIFIED: `src/config/region-config.ts` — Fixed empty endpoint URLs
- CREATED: `src/hooks/useInsightsPolling.ts` — Polling hook with auto-stop
- MODIFIED: `src/components/trace-eu/integration/DeepDocumentDetailsEU.tsx` — Added polling + progress animation
- MODIFIED: `src/components/trace-eu/integration/RightPanelEU.tsx` — Detects processing state
- MODIFIED: `src/components/trace-eu/guest-flow-eu/GuestFlowPanelEU.tsx` — Guest polling support
- MODIFIED: `src/queries/deep-document-analysis/upload-deep-document.ts` — Async-aware toasts + documentId pass-through
**Build:** SUCCESS (zero errors)
