## Session 2026-02-12 06:00 — hotfix-insights-polling
### Actions
- [06:00] BRANCH created hotfix-insights-polling from feature-eu-standalone-app
- [06:00] Read region-config.ts, useJobPolling.ts, use-get-document-insights-query.ts, upload-deep-document.ts
- [06:00] Read DeepDocumentDetailsEU.tsx, GuestFlowPanelEU.tsx, RightPanelEU.tsx, DeepDocumentUploadEU.tsx
- [06:00] ROOT CAUSE: insights_data returns {status: "processing"} but frontend never polls — useGetDocumentInsights exists but is never called
### Files Touched
### Summary
Starting Sprint 1 — fix insights polling for async document analysis.
