### COMMIT 20 — 2026-02-12T15:15:00Z
**Milestone:** Autonomous Lambda deployment — 3 Lambdas live with Function URLs + standard tool ADR
**State:** DONE
**Files Changed:**
- CREATED: `deploy_deep_research_lambdas.py` — boto3 ZIP deployment script (standard tool)
- CREATED: `.env.local` (frontend) — Function URLs auto-populated
- CREATED: `deep_research_function_urls.json` — Deployed URL reference
- CREATED: `.gsm/decisions/ADR-021-standard-lambda-deploy-tool.md` — Standard deployment tool pattern
- MODIFIED: `.gsm/index.md` — Added ADR-018 through ADR-021
**Key Decisions:**
- boto3 + ZIP over bash + Docker (per ADR-014) — autonomous, no user intervention
- CORS AllowMethods=["*"] (not individual methods — AWS API 6-char limit on individual values)
- Standard deployment script template codified in ADR-021
**AWS Resources Created:**
- DynamoDB: eu-deep-research-jobs (PK=job_id, TTL on 'ttl')
- Lambda: eu_web_search (512MB, 30s) → https://szwe24pakrrtpojpbfv5lqdlxu0xqqnu.lambda-url.eu-central-1.on.aws/
- Lambda: eu_deep_research (512MB, 900s) → https://xcw7giwpn2bpv7rsd4xjcl4aci0rssop.lambda-url.eu-central-1.on.aws/
- Lambda: eu_deep_research_status (256MB, 10s) → https://kyylsjckef4ektconmdp5bphjy0tqjye.lambda-url.eu-central-1.on.aws/
- Function URLs: 3x auth-type NONE with CORS and public invoke permission
**Next:**
- [ ] Set TAVILY_API_KEY on eu_web_search (user must provide key)
- [ ] E2E smoke test: deep research + web search flow
- [ ] Visual audit at 375px, 768px, 1440px
- [ ] Deploy existing Lambdas that also need web_search support (EUChatAthenaBot update)
**Blockers:** TAVILY_API_KEY not set — eu_web_search returns 503 until key is provided
