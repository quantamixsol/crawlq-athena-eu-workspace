# hotfix-insights-polling — Commit Log

### BRANCH CREATED — 2026-02-12T06:00:00Z
**Name:** hotfix-insights-polling
**Parent:** feature-eu-standalone-app
**Purpose:** Fix critical bug: document upload returns status:"processing" but frontend never polls for completed insights. Insights hooks exist but are never called.
**Success Criteria:**
- Upload PDF → see spinner → 5-15s later insights appear automatically
- Both guest and logged-in flows work
- Toast message reflects async processing
- Region-config has proper endpoint URLs

---

### COMMIT 1 — 2026-02-12T06:30:00Z
**Milestone:** Sprint 1 complete — insights polling for async document analysis
**State:** DONE
**Files Changed:**
- MODIFIED: `src/config/region-config.ts` — Fixed getDeepInsights and getDocumentInsights endpoints (were empty strings, now fallback to API Gateway routes)
- CREATED: `src/hooks/useInsightsPolling.ts` — Custom polling hook: refetchInterval 3s, max 40 retries, auto-stop on complete/fail, isInsightsProcessing helper
- MODIFIED: `src/components/trace-eu/integration/DeepDocumentDetailsEU.tsx` — Added polling when insights are processing, 5-stage progress animation, retry on timeout
- MODIFIED: `src/components/trace-eu/integration/RightPanelEU.tsx` — handleUploadSuccess now detects processing state, sets accurate status/isProcessed
- MODIFIED: `src/components/trace-eu/guest-flow-eu/GuestFlowPanelEU.tsx` — Added polling in processing phase, detects async response, transitions to analytics when insights arrive
- MODIFIED: `src/queries/deep-document-analysis/upload-deep-document.ts` — Guest upload returns documentId+sessionId for polling, both toasts reflect processing state
**Key Decisions:**
- Polling via React Query refetchInterval (3s) with max 40 retries (2 min timeout) — follows existing useJobPolling pattern
- isInsightsProcessing helper checks both status field and empty insights array
- Guest upload modified to pass through documentId and sessionId for polling
- Region-config endpoints fallback to ${EU_API_BASE}/get-document-insights and ${EU_API_BASE}/get-insights
**Next:**
- [ ] Verify /get-document-insights Lambda exists on API Gateway (may need backend deployment)
- [ ] MERGE into feature-eu-standalone-app
- [ ] Start Sprint 2 (feature-markdown-viz) or Sprint 5 (feature-guest-conversion)
**Blockers:** Backend /get-document-insights endpoint needs verification — if Lambda doesn't exist, backend team must deploy it

