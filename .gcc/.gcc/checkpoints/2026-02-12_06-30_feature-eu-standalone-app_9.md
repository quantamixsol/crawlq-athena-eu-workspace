### COMMIT 9 — 2026-02-12T06:30:00Z
**Milestone:** Fix Lambda response unwrapping + wire KG overlay to guest flow
**State:** DONE
**Files Changed:**
- MODIFIED: `src/lib/apiclient-config.ts` — Added response interceptor to auto-unwrap Lambda Function URL format
- MODIFIED: `src/queries/deep-document-analysis/upload-deep-document.ts` — Added unwrapLambdaResponse() safety net
- MODIFIED: `src/queries/chat-eu/useEUSendMessage.ts` — Migrated from raw axios to apiClient
- MODIFIED: `src/queries/chat-eu/useEUChatHistoryQuery.ts` — Migrated from raw axios to apiClient
- MODIFIED: `src/components/trace-eu/guest-flow-eu/GuestFlowPanelEU.tsx` — Added KG overlay
**Key Decisions:**
- Response interceptor at apiClient level fixes ALL 8+ affected endpoints at once
- Chat EU queries migrated to apiClient for consistent auth + unwrapping
- Guest flow gets full KG overlay (was missing openOverlay prop)
**Verified:**
- Build: zero errors, all 10 pages compile
- Git commits e46002a + bc29ca9 pushed to main
**Next:**
- Verify Amplify deployment
- E2E smoke test
- Visual audit
