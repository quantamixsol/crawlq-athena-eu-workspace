# ADR-017: EU Standalone App Extraction Strategy
**Date:** 2026-02-12 | **Status:** ACCEPTED

**Context:** CrawlQ has a monorepo with US and EU code mixed together. Over 20 commits, we've built 82 EU-specific frontend files (~17,500 LoC), 20 Lambda functions, and 15 shared compliance modules. The EU code needs to be extracted into standalone repos to prevent cross-contamination with US code and enable independent development/deployment.

**Decision:** Build two new standalone GitHub repos on `quantamixsol` account:
1. `crawlq-chat-athena-eu-frontend` — Next.js 14 app with all EU components + shared dependencies
2. `crawlq-athena-eu-backend` — 20 Lambda functions + shared compliance modules + CI/CD

This is a **copy-and-compose** approach, NOT a repo split. The original monorepo continues to exist. The new repos are independent extracts that can evolve separately.

**What gets extracted:**
- Frontend: 82 EU files + ~300 shared deps (ui/, lib/, hooks/, stores, queries, config)
- Backend: 20 Lambda dirs + 15 shared modules + deploy.sh + provision_aws.sh + GitHub Actions

**What stays behind (US-only, excluded):**
- US chat pages (`app/(protected)/chat-athena/`)
- US KG components (`components/knowledge-graph/`, `trace-knowledge-graph/`)
- US-only stores, queries, agents, market research, socials
- US Lambda functions (outside SemanticGraphEU/)

**Consequences:**
- (+) Complete isolation — EU and US can never contaminate each other
- (+) Independent CI/CD, deployment, and versioning
- (+) Clean dependency graph — no dead US code in EU repo
- (+) Faster builds (fewer files to compile)
- (-) Shared code duplication (ui/, lib/, hooks/ exist in both monorepo and EU repo)
- (-) Changes to shared code must be manually synced
- (-) Initial extraction effort (~300 files to copy and verify)

**Mitigation for duplication:** The EU repo is the source of truth for EU code going forward. The monorepo's EU files become frozen/archived.
