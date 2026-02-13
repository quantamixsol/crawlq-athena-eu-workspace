### COMMIT 22 — 2026-02-12T16:30:00Z
**Milestone:** ADR-022 — Smart chat mode rules: Web Search, TRACE, Combined, Plain + auto-scroll fix
**State:** DONE
**Files Changed:**
- MODIFIED: `SemanticGraphEU/EUChatAthenaBot/handler.py` — 4 response modes (ADR-022), web-search-only skips Bedrock, TRACE conditional
- MODIFIED: `src/store/chat-eu/useChatEUStore.ts` — Added traceEnabled toggle + mode metadata
- MODIFIED: `src/app/(protected)/chat-athena-eu/page.tsx` — Smart toggle wiring: Deep Research auto-enables both
- MODIFIED: `src/components/chat-eu/ChatToolbar.tsx` — Renamed TRACE Graph → TRACE
- MODIFIED: `src/queries/chat-eu/useEUStreamingMessage.ts` — Send trace_enabled, mode-aware thinking text, mode metadata
- MODIFIED: `src/components/chat-eu/ChatContainer.tsx` — Pass traceEnabled to streaming hook
- MODIFIED: `src/components/chat-eu/ChatMessageBubble.tsx` — TRACE card conditional on trace_dimensions
- MODIFIED: `src/components/chat-eu/ChatMessageArea.tsx` — Auto-scroll on streaming content + card appearance
- CREATED: `.gsm/decisions/ADR-022-chat-mode-rules.md` — Chat mode matrix decision record
**Key Decisions:**
- Web-search-only skips Bedrock entirely (12s vs 25s) — returns Perplexity answer directly
- TRACE defaults ON for backward compatibility
- Deep Research ON auto-enables Web Search + TRACE; disabling either auto-disables Deep Research
- Response includes `mode` field for frontend rendering decisions
**Smoke Test Results:**
- web_search: 12.6s, perplexity-sonar-pro, no TRACE
- trace: 26.4s, claude-opus-4-6, confidence 0.45
- combined: 24.4s, claude-opus-4-6, 5 sources + confidence 0.80
- chat: 7.1s, claude-opus-4-6, no TRACE
**Git:** 9a32e03 (backend), e613667 (frontend) pushed to main
**Deployed:** eu_chat_athena_bot (15.54 MB)
**Next:**
- [ ] KG exploration UI
- [ ] TRACE protocol deep research
- [ ] Visual audit
