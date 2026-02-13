# ADR-021: Standard Lambda Deployment Tool (boto3 + ZIP + Function URLs)

**Date:** 2026-02-12 | **Status:** ACCEPTED
**Decision Makers:** Claude Code Agent

---

## Context

Deploying new Lambda functions to the CrawlQ Athena EU stack requires:
1. Creating ZIP deployment packages (handler + shared/ modules + pip dependencies)
2. Creating/updating Lambda functions in eu-central-1
3. Configuring Function URLs with CORS for frontend access
4. Setting environment variables (DynamoDB tables, API keys, Bedrock model IDs)
5. Ensuring DynamoDB tables exist
6. Outputting `.env.local` snippets for frontend integration

Previous deployments used a mix of bash scripts (deploy.sh, provision_aws.sh) and boto3 scripts (deploy_tier3_zip.py). Per ADR-014, boto3 is preferred for programmatic deployment.

---

## Decision

**Standardize all new Lambda deployments using a reusable boto3 script pattern.**

### Standard Script Template

Every new Lambda deployment script MUST follow this structure:

```python
#!/usr/bin/env python3
"""
{Description} Lambda Deployment Script (boto3, ZIP-based)
Standard tool — ADR-014 (boto3), ADR-018 (Function URLs), ADR-021 (template)
"""

# 1. Configuration block: REGION, ACCOUNT_ID, EXECUTION_ROLE, BACKEND_ROOT
# 2. LAMBDAS dict: path, handler, memory, timeout, runtime, needs_function_url, env_vars
# 3. boto3 clients: lambda_client, iam, dynamodb
# 4. Reusable functions:
#    - get_execution_role_arn()     — discover from existing Lambda or named role
#    - ensure_dynamodb_table()      — idempotent table creation with TTL
#    - create_deployment_package()  — ZIP with shared/ modules + pip deps
#    - create_or_update_lambda()    — idempotent create/update with waiters
#    - create_function_url()        — Function URL + CORS + public permission
# 5. main() — orchestrates Steps 1-4, prints summary + .env.local snippet
```

### Key Patterns

| Pattern | Implementation |
|---------|---------------|
| **Role discovery** | Try named role first, then probe existing Lambdas |
| **ZIP packaging** | `pip install --platform manylinux2014_x86_64` with fallback to generic |
| **shared/ modules** | Copy entire `shared/` directory into ZIP |
| **Idempotent create** | Try `get_function`, create if not found, update if exists |
| **Function URLs** | `AuthType=NONE`, `Cors.AllowMethods=["*"]`, public `InvokeFunctionUrl` |
| **Waiters** | `function_active` for create, `function_updated` for update |
| **Output** | JSON file + `.env.local` snippet + warnings for missing env vars |

### Naming Convention

```
deploy_{feature_name}_lambdas.py
```

Examples:
- `deploy_tier3_zip.py` — Tier 3 async job queue (3 Lambdas)
- `deploy_deep_research_lambdas.py` — Deep research pipeline (3 Lambdas)

### CLI Flags

| Flag | Purpose |
|------|---------|
| `--only {name}` | Deploy single Lambda |
| `--skip-package` | Skip ZIP creation (reuse existing) |

---

## Existing Deployment Scripts

| Script | Lambdas | Status |
|--------|---------|--------|
| `deploy_tier3_zip.py` | eu_chat_job_queue, eu_chat_job_status, eu_chat_job_worker | DEPLOYED |
| `deploy_deep_research_lambdas.py` | eu_web_search, eu_deep_research, eu_deep_research_status | DEPLOYED |
| `deploy.sh` (bash) | All 23 Lambdas (Docker-based, for CI/CD) | REFERENCE |
| `provision_aws.sh` (bash) | DynamoDB, S3, ECR, Lambda+FuncURL | REFERENCE |

---

## Consequences

### Positive
1. **Fully autonomous** — agent deploys without user running manual commands
2. **Idempotent** — safe to re-run (creates if missing, updates if existing)
3. **Cross-platform** — works on Windows, Linux, macOS (no Docker/AWS CLI required)
4. **Fast** — ZIP deploys in ~60s total for 3 Lambdas
5. **Auto-configures frontend** — outputs `.env.local` snippet with Function URLs

### Negative
1. **ZIP size limit** — 50MB direct upload, 250MB unzipped (S3 needed for larger)
2. **No Docker** — can't test container-based Lambdas locally
3. **Cross-compilation** — some C-extension packages need fallback pip install

### Trade-offs
- ZIP (fast, simple) vs Docker (production-grade, larger packages)
- Use ZIP for development iteration, Docker for CI/CD production deploys

---

## Related Decisions

- **ADR-014**: boto3 over AWS CLI for automation
- **ADR-018**: Function URLs for long-running Lambdas
- **ADR-004**: ZIP-based deployment from Windows
- **ADR-001**: Lambda Function URLs AuthType: NONE

---

## Deployed Function URLs (2026-02-12)

```
eu_web_search:              https://szwe24pakrrtpojpbfv5lqdlxu0xqqnu.lambda-url.eu-central-1.on.aws/
eu_deep_research:           https://xcw7giwpn2bpv7rsd4xjcl4aci0rssop.lambda-url.eu-central-1.on.aws/
eu_deep_research_status:    https://kyylsjckef4ektconmdp5bphjy0tqjye.lambda-url.eu-central-1.on.aws/
```
