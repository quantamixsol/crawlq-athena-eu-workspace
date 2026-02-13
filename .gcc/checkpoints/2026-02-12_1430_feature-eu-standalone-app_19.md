### COMMIT 19 — 2026-02-12T14:30:00Z
**Milestone:** Deployment scripts for 3 new Lambdas with Function URLs + frontend Function URL support
**State:** DONE
**Files Changed:**
- MODIFIED: `SemanticGraphEU/EUWebSearch/Dockerfile` — Fixed: python:3.9 base, match existing pattern
- MODIFIED: `SemanticGraphEU/EUDeepResearch/Dockerfile` — Fixed: python:3.9 base, match existing pattern
- MODIFIED: `SemanticGraphEU/EUDeepResearchStatus/Dockerfile` — Fixed: python:3.9 base, match existing pattern
- MODIFIED: `SemanticGraphEU/deploy.sh` — Added shared/ copy-before-build step for Docker context
- MODIFIED: `SemanticGraphEU/provision_aws.sh` — Added Lambda creation + Function URL section
- CREATED: `SemanticGraphEU/deploy-new-lambdas.sh` — Full deployment: ECR + Lambda + Function URLs + env vars
- MODIFIED: `src/config/region-config.ts` (frontend) — Function URL env vars for deep research endpoints
- MODIFIED: `.env.example` (frontend) — Added DEEP_RESEARCH_URL, STATUS_URL, WEB_SEARCH_URL
**Key Decisions:**
- Function URLs with auth-type NONE + CORS (same pattern as compliance Lambdas)
- deploy.sh copies shared/ into each Lambda dir before build, cleans up after
- Frontend falls back to API Gateway base URL if Function URL env vars not set
- EUDeepResearch 900s timeout, 512MB memory
**Git:** 18a78e3 pushed to main (backend), ef9f998 pushed to main (frontend)
**Next:**
- [ ] Run deploy-new-lambdas.sh from AWS-CLI-enabled environment
- [ ] Set TAVILY_API_KEY on eu_web_search Lambda
- [ ] Update frontend .env.local with actual Function URLs
- [ ] E2E smoke test: web search + deep research flow
**Blockers:** AWS CLI not installed on dev machine — scripts must be run from WSL or CI/CD
