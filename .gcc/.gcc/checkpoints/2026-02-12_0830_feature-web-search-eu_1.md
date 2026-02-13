### COMMIT 1 — 2026-02-12T08:30:00Z (feature-web-search-eu)
**Milestone:** Web search frontend integration complete — store toggles, API wiring, source citations, page.tsx store binding
**State:** DONE
**Files Changed:**
- MODIFIED: `src/store/chat-eu/useChatEUStore.ts` — Added web_sources metadata, webSearchEnabled/deepResearchEnabled toggles
- CREATED: `src/components/chat-eu/WebSourcesCard.tsx` — Perplexity-style source citations with expand/collapse
- MODIFIED: `src/queries/chat-eu/useEUStreamingMessage.ts` — web_search flag in payload, web_sources in metadata
- MODIFIED: `src/components/chat-eu/ChatContainer.tsx` — Pass webSearch: webSearchEnabled to sendStreamingMessage
- MODIFIED: `src/components/chat-eu/ChatMessageBubble.tsx` — Render WebSourcesCard when web_sources present
- MODIFIED: `src/app/(protected)/chat-athena-eu/page.tsx` — Store-backed toolbar toggles replace local useState
**Git:** Commit 8143741 pushed to main
**Blockers:** None
