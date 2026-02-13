### COMMIT 37 — 2026-02-13T19:00:00Z
**Milestone:** ATHENA MAIN SESSION — Full repo isolation enforced + Amplify repointed + Canvas separated + 4 repos clean
**State:** DONE
**Branch:** feature-eu-standalone-app

**Repo Architecture (final):**

| Repo | Purpose | Status |
|------|---------|--------|
| `crawlq-ui` | US CrawlQ app ONLY | READ-ONLY for EU work |
| `crawlq-chat-athena-eu-frontend` | Athena EU frontend (all features except Canvas) | DEPLOYED — Amplify d45bl3mgpjnhy |
| `crawlq-athena-eu-backend` | Athena EU Lambdas (25+ functions) | DEPLOYED — 4 new Lambdas active |
| `crawlq-athena-eu-canvas` | TRACE Canvas app (separate track) | PUSHED — not yet Amplify-deployed |
| `crawlq-lambda` | US CrawlQ backend ONLY | READ-ONLY for EU work |

**Key Changes:**
- Migrated 581 frontend files from crawlq-ui to crawlq-chat-athena-eu-frontend (Canvas removed)
- Migrated 2203 backend files from crawlq-lambda to crawlq-athena-eu-backend
- Created crawlq-athena-eu-canvas with 118 Canvas-specific files
- Repointed Amplify d45bl3mgpjnhy from crawlq-ui to crawlq-chat-athena-eu-frontend
- Deleted old Amplify app d27i99z4z1clr2
- Deployed 4 new Lambdas + 2 DynamoDB tables
- Created ADR-032 (CONSTITUTIONAL repo isolation)
- E2E: 12/12 tests passed

**Production URL:** https://main.d45bl3mgpjnhy.amplifyapp.com

**18 Deployed Features:** EU Chat, TRACE 5-Pillar, Deep Research, Web Search, Session KG, AI-First Onboarding, Multi-Session Workspaces, FrictionMelt Integration, Enterprise Markdown, Document Analysis, Export, Command Palette, Profile & Theme, Search & Feedback, Intelligence Tips, Guest Conversion, Suggested Actions, Artifact Panel

**Next:**
- [ ] Deploy crawlq-athena-eu-canvas to its own Amplify app
- [ ] Re-add Session KG button to ChatToolbar
- [ ] Phase 17: Full E2E Testing across all 4 repos
- [ ] Phase 18: Marketing, Website, Production Launch
