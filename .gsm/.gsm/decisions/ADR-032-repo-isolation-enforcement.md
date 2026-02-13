# ADR-032: Repository Isolation Enforcement — CONSTITUTIONAL

**Date:** 2026-02-13 | **Status:** CONSTITUTIONAL (cannot be violated)

## Context
Previous sessions drifted Athena EU development back into `crawlq-ui` (the US CrawlQ app), violating ADR-017 repo isolation. This caused confusion about which repo is the source of truth and risked contaminating the US codebase.

## Decision

### HARD RULES (enforced on every session)

1. **crawlq-ui** = US CrawlQ app ONLY. **READ-ONLY for Athena EU work.** No Athena EU features, no EU components, no EU routes. This repo is the US product.

2. **crawlq-chat-athena-eu-frontend** = Athena EU frontend. ALL EU features (TRACE, onboarding, chat, FrictionMelt, deep research, KG, sessions, intelligence). Deployed to Amplify app d45bl3mgpjnhy.

3. **crawlq-athena-eu-backend** = Athena EU Lambdas. ALL 25+ EU Lambda functions. SemanticGraphEU/ directory.

4. **crawlq-athena-eu-canvas** = TRACE Canvas app (SEPARATE track). React Flow workflow builder. Shares backend Lambdas with #3 but has its own frontend deployment.

5. **crawlq-lambda** = US CrawlQ backend ONLY. **READ-ONLY for Athena EU work.**

### Session Start Check
Every session MUST verify:
- [ ] Which repo am I working in?
- [ ] Does my change belong in this repo?
- [ ] Am I accidentally modifying crawlq-ui or crawlq-lambda?

### Violation = Immediate Rollback
If Athena EU code is found in crawlq-ui or crawlq-lambda:
1. STOP all work
2. Migrate the code to the correct repo
3. Revert crawlq-ui/crawlq-lambda to clean state
4. Document the incident

## Consequences
- **Positive:** Clean repo boundaries, safe US app, independent deployments
- **Negative:** Slightly more overhead (4 repos instead of 2), need to sync shared types
- **Risk Mitigation:** Shared types/utils copied (not linked) between repos
