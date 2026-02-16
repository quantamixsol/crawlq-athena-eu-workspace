### COMMIT 44 — 2026-02-14T02:25:00Z
**Branch:** feature-eu-standalone-app
**Milestone:** Fixed web search routing + workspace creation UI — Frontend deployed, 21/21 UAT PASS
**State:** DONE

**Summary:**
- Fixed web search: queries with web_search=true bypass async mode (async worker lacks web search support)
- Added workspace creation UI: FolderPlus button in sidebar header + inline name input
- Workspace creation mutation now accepts custom name (was hardcoded to "New Campaign")
- Renamed "Campaign" → "Workspace" in frontend constants
- Frontend pushed (d6bb156, 37c7e71) + Amplify deployed (Jobs 11, 12 SUCCEED)
- Backend pushed (2b2fe14) — Neo4j EU + Campaign→Workspace

**Remaining:**
- Phase 18: Marketing, Website, Production Launch
- Custom domain setup for crawlq.ai
- Add web search to EUChatJobWorker (async mode enhancement)
