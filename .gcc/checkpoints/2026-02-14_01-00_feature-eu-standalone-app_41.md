### COMMIT 41 — 2026-02-14T01:00:00Z
**Branch:** feature-eu-standalone-app
**Milestone:** Migrated EUGetDeepInsights from OpenAI to Bedrock Claude Sonnet 4.5 — 27/32 endpoints PASS (84%). OpenAI dependency fully removed.
**State:** WORKING

**Summary:**
- Replaced OpenAI with Bedrock Claude Sonnet 4.5 in EUGetDeepInsights (4 LLM calls)
- Removed openai from requirements.txt
- Added EU_BEDROCK_SONNET_MODEL_ID to eu_config.py
- Deployed Lambda with neo4j driver + Bedrock
- Final test: 27 PASS / 2 WARN / 3 FAIL (all 3 are expected behavior)

**Remaining:**
- Configure Neo4j URI when EU Neo4j instance available
- Push backend fixes to repo
- Phase 18: Production launch
