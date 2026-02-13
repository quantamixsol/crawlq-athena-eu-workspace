# Checkpoint: feature-eu-chat-athena COMMIT 5
**Timestamp:** 2026-02-10T01:00:00Z
**Branch:** feature-eu-chat-athena
**State:** WORKING

## Summary
Fixed 3 root causes blocking real-world usage:
1. **Model:** Changed from Sonnet to Opus 4.6 (`eu.anthropic.claude-opus-4-6-v1`)
2. **URLs:** Replaced 3 placeholder URLs in region-config.ts with real Lambda Function URLs
3. **Auth:** Changed AuthType from AWS_IAM to NONE on 3 guest-facing Lambdas

## Deployments
- 7 Lambdas redeployed with updated shared/eu_config.py
- 17 Lambdas env var set to Opus 4.6
- Amplify Job 4: SUCCEED
- Git: c399e382 on feature/trace-eu-frontend

## Verified
- eu_chat_athena_bot confirmed using eu.anthropic.claude-opus-4-6-v1
- All 17 Lambda handlers load successfully
