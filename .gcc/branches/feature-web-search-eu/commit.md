# feature-web-search-eu — Commit Log

### BRANCH CREATED — 2026-02-12T07:55:00Z
**Name:** feature-web-search-eu
**Parent:** feature-eu-standalone-app
**Purpose:** Sprint 3 — Perplexity-style web search frontend: store toggles, API wiring, source citations display
**Success Criteria:**
- Toggle "Web Search" in toolbar → web_search flag sent to backend
- WebSourcesCard displays source citations with domain, title, snippet
- Store-based feature toggles (webSearch, deepResearch) in useChatEUStore
- All wiring in place — ready for backend Lambda EUWebSearch

---

### COMMIT 1 — 2026-02-12T08:30:00Z
**Milestone:** Web search frontend integration complete — store toggles, API wiring, source citations, page.tsx store binding
**State:** DONE
**Files Changed:**
- MODIFIED: `src/store/chat-eu/useChatEUStore.ts` — Added web_sources metadata, webSearchEnabled/deepResearchEnabled toggles
- CREATED: `src/components/chat-eu/WebSourcesCard.tsx` — Perplexity-style source citations with expand/collapse
- MODIFIED: `src/queries/chat-eu/useEUStreamingMessage.ts` — web_search flag in payload, web_sources in metadata
- MODIFIED: `src/components/chat-eu/ChatContainer.tsx` — Pass webSearch: webSearchEnabled to sendStreamingMessage
- MODIFIED: `src/components/chat-eu/ChatMessageBubble.tsx` — Render WebSourcesCard when web_sources present
- MODIFIED: `src/app/(protected)/chat-athena-eu/page.tsx` — Store-backed toolbar toggles replace local useState
**Key Decisions:**
- Feature toggles in Zustand store (not local state) so ChatContainer/streaming hook can access them
- WebSourcesCard shows first 3 sources collapsed, expand to see all (Perplexity pattern)
- web_search boolean sent in existing streaming POST body — backend EUChatAthenaBot reads it
**Next:**
- [ ] Backend: Create EUWebSearch Lambda (Perplexity/Tavily API proxy)
- [ ] Backend: Modify EUChatAthenaBot to invoke EUWebSearch when web_search=true
**Blockers:** None

