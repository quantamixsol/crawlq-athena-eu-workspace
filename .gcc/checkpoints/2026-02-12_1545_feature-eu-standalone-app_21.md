### COMMIT 21 — 2026-02-12T15:45:00Z
**Milestone:** Perplexity AI integration — sonar-pro replaces Tavily for web search
**State:** DONE
**Files Changed:**
- MODIFIED: `SemanticGraphEU/EUWebSearch/handler.py` — Complete rewrite: Perplexity chat/completions API, sonar-pro model, citation extraction, synthesized answer
- MODIFIED: `SemanticGraphEU/EUDeepResearch/handler.py` — Capture Perplexity answer for chain-of-thought context
- MODIFIED: `SemanticGraphEU/EUChatAthenaBot/handler.py` — Use Perplexity answer as high-quality web context in chat
- MODIFIED: `SemanticGraphEU/shared/eu_config.py` — TAVILY_API_KEY → PERPLEXITY_API_KEY
- MODIFIED: `deploy_deep_research_lambdas.py` — Updated env vars for Perplexity
**Key Decisions:**
- sonar-pro model for high-quality citations + synthesized answers (matches US app pattern)
- Perplexity answer fed into both chat and deep research as first-class context
- Citation-to-snippet extraction: parse [N] markers from answer text, map to citation URLs
- search_recency_filter: "month" for fresh results
**Git:** 4f74cfe pushed to main (backend)
**Deployed:** eu_web_search + eu_deep_research + eu_chat_athena_bot (all 3 redeployed via boto3)
**Next:**
- [ ] Set PERPLEXITY_API_KEY on eu_web_search Lambda
- [ ] E2E smoke test: web search + deep research
- [ ] Visual audit
**Blockers:** PERPLEXITY_API_KEY not set — user needs to provide key
