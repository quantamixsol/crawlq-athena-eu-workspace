# Checkpoint: feature-tier3-async-markdown — COMMIT 3
**Date:** 2026-02-11 14:23 UTC
**Branch:** feature-tier3-async-markdown
**Commit:** 3
**State:** DONE

## Session Summary
Successfully deployed all 3 Tier 3 Lambda functions using boto3 (not AWS CLI). Fixed 7 deployment issues including permissions, CORS, Decimal serialization, composite keys, and query string parameters. Created ADR-014 documenting boto3 deployment decision.

## Lambda Functions Deployed
1. **eu_chat_job_queue** — POST endpoint for job submission
   - URL: https://msby2wga4iovicryrw7swa2euy0kiaai.lambda-url.eu-central-1.on.aws/
   - Status: ✅ Deployed & Tested

2. **eu_chat_job_status** — GET endpoint for status polling
   - URL: https://d3fjrowuuyxxu2qoika22x2n6y0bqyzq.lambda-url.eu-central-1.on.aws/
   - Status: ✅ Deployed & Tested

3. **eu_chat_job_worker** — SQS-triggered async processor
   - Trigger: eu-chat-jobs queue
   - Status: ✅ Deployed (needs API key for testing)

## Issues Fixed
1. 403 Forbidden → Added lambda:InvokeFunction permission
2. CORS error → Changed to AllowMethods: ['*']
3. AWS_REGION reserved → Removed from env vars
4. Float not supported → Added Decimal conversion
5. Decimal not JSON serializable → Convert back to float for SQS
6. Missing path parameter → Changed to query string parameter
7. Composite key error → Changed GetItem to Query

## Files Created
- deploy_tier3_zip.py (350 lines)
- test_tier3_e2e.py (300 lines)
- ADR-014-boto3-deployment-over-aws-cli.md (200 lines)
- tier3_function_urls.json

## Files Modified
- crawlq-ui/src/hooks/useJobPolling.ts (Function URLs)
- crawlq-lambda/SemanticGraphEU/EUChatJobQueue/handler.py (Decimal fix)
- crawlq-lambda/SemanticGraphEU/EUChatJobStatus/handler.py (Query fix)
- .gsm/index.md (ADR-014 entry)

## Next Steps
- Add Anthropic API key to SSM
- Run full end-to-end test
- Verify markdown quality
- Update ChatContainer component
- Monitor CloudWatch logs

## Key Decision
**ADR-014:** Use boto3 for Lambda deployment instead of AWS CLI. Rationale: boto3 is more reliable, already available, better error handling, cross-platform compatible.

## Testing
- ✅ Queue endpoint tested (200 OK, returns job_id)
- ✅ Status endpoint tested (200 OK, returns status)
- ✅ SQS trigger configured
- ⏸️ Full e2e processing (needs API key)

## Implementation Complete: 95%
Only missing: Anthropic API key for worker processing.
All infrastructure deployed, all endpoints working.
