# ADR-043: Canonical Lambda Update Deployment Method

**Date:** 2026-02-16 | **Status:** ACCEPTED
**Supersedes:** Extends ADR-004, ADR-014, ADR-021

---

## Context

After Phases 1-6 remediation, 19+ Lambda functions need redeployment. The existing deploy methods (ADR-004 ZIP, ADR-021 create-new) don't cover the most common case: **updating existing Lambdas** that already have bundled dependencies (neo4j, anthropic, etc.).

Key discovery on 2026-02-16:
- All EU Lambdas are `PackageType=Zip`, `Runtime=python3.10` (not Docker despite Dockerfiles existing)
- Each ZIP contains: source .py files + `shared/` directory + bundled pip dependencies (neo4j, etc.)
- Package sizes: 65KB-305KB (dependencies already bundled)
- No Lambda layers — everything is self-contained in the ZIP

## Decision

**The canonical method for updating existing Lambda source code is: Download → Overlay → Upload.**

### Method: Download-Overlay-Upload

```
1. DOWNLOAD: boto3.get_function() → download presigned URL → current.zip
2. OVERLAY:  Unzip into memory → replace handler.py, helpers.py, shared/* → re-zip
3. UPLOAD:   boto3.update_function_code(ZipFile=new_zip_bytes)
```

This preserves all bundled dependencies (neo4j, anthropic, PyJWT, etc.) while updating only the source code files we changed.

### When to Use Each Method

| Scenario | Method | Script |
|----------|--------|--------|
| **Update source code** (handler, shared) | Download-Overlay-Upload | `deploy_eu_lambdas.py` |
| **Add new dependency** (requirements.txt changed) | Full rebuild with pip install | `deploy_eu_lambdas.py --full-rebuild` |
| **Create new Lambda** | Full build + create_function | Per ADR-021 template |
| **CI/CD production** | Docker build + ECR push | `deploy.sh` |

### Script: `deploy_eu_lambdas.py`

The canonical deployment script lives at:
```
crawlq-athena-eu-backend/SemanticGraphEU/deploy_eu_lambdas.py
```

Features:
- `python deploy_eu_lambdas.py` — Deploy all modified Lambdas (overlay method)
- `python deploy_eu_lambdas.py --only eu_chat_athena_bot` — Deploy single Lambda
- `python deploy_eu_lambdas.py --full-rebuild` — Rebuild with pip install (for new deps)
- `python deploy_eu_lambdas.py --dry-run` — Show what would be deployed without uploading
- Always includes: handler.py, helpers.py, *.py, shared/, _jwt_lib/

### Lambda Registry

All 19 core Lambdas that use `shared/` modules:

| Function Name | Directory | Modified In |
|--------------|-----------|-------------|
| eu_get_deep_insights | EUGetDeepInsights | Phase 1 |
| eu_get_document_insights | EUGetDocumentInsights | Phase 1 |
| eu_upload_deep_document | EUUploadDeepDocument | Phase 1+3 |
| eu_subscription | EUSubscription | Phase 1 |
| eu_generate_deep_insights | EUGenerateDeepInsights | Phase 2 |
| eu_deep_graph_builder | EUGraphBuilder | Phase 2+3 |
| eu_conversation_memory | EUConversationMemory | Phase 3 |
| eu_response_kg_extractor | EUResponseKGExtractor | Phase 3 |
| eu_chat_athena_bot | EUChatAthenaBot | Phase 4 |
| eu_compliance_engine | EUComplianceEngine | Phase 6 |
| eu_onboard_user | EUOnboardUser | shared/ |
| eu_process_deep_document | EUProcessDeepDocument | shared/ |
| eu_deep_reasoner | EUReasoner | shared/ |
| eu_trace_explainer | EUTraceExplainer | shared/ |
| eu_audit_trail_store | EUAuditTrailStore | shared/ |
| eu_audit_trail_verify | EUAuditTrailVerify | shared/ |
| eu_consent_manager | EUConsentManager | shared/ |
| eu_get_chat_history | EUGetChatHistory | shared/ |
| eu_get_deep_documents | EUGetDeepDocuments | shared/ |

### File Inclusion Rules

For every Lambda deployment, the ZIP must contain:
1. **All .py files** from the Lambda directory (handler.py, helpers.py, chat_engine.py, etc.)
2. **shared/** directory (entire tree, excluding __pycache__)
3. **_jwt_lib/** directory (PyJWT library for jwt_auth.py)
4. **requirements.txt** from the Lambda directory
5. **All existing bundled dependencies** (preserved from current ZIP via overlay)

### Repo Isolation Rule (ADR-032/033)

**NEVER deploy to or modify:**
- `crawlq-lambda` repo (US backend)
- `crawlq-ui` repo (US frontend)

Only deploy from `crawlq-athena-eu-backend` to `eu-central-1` Lambdas.

## Consequences

### Positive
- **Safe**: Preserves existing dependencies — no broken imports
- **Fast**: ~5 seconds per Lambda (download + overlay + upload)
- **Idempotent**: Safe to re-run any number of times
- **No Docker/CLI required**: Pure Python + boto3
- **Deterministic**: Same overlay produces same result

### Negative
- **Can't add new pip dependencies** without --full-rebuild
- **ZIP size grows** over time if unused files accumulate
- **Must have AWS credentials** configured (via boto3 credential chain)

## Related
- ADR-004: ZIP-based Lambda deployment from Windows
- ADR-014: boto3 over AWS CLI
- ADR-021: Standard Lambda deployment tool template
- ADR-032: Repo isolation enforcement
