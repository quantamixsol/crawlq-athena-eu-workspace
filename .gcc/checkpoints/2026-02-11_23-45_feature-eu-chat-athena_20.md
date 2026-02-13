### COMMIT 20 — 2026-02-11T23:45:00Z ★ LOCKED TOOL
**Milestone:** Visual UI Test Tool created — Playwright + AWS Bedrock Opus 4.6 Vision Analysis (LOCKED FOR ALWAYS USE)
**State:** DONE
**Files Changed:**
- CREATED: `crawlq-ui/scripts/visual-audit-eu.mjs` — Full visual audit tool
- CREATED: `.gsm/decisions/ADR-016-visual-tool-project-isolation.md` — HARD REQUIREMENT
- MODIFIED: `.gsm/index.md` — Added ADR-014, ADR-015, ADR-016
- MODIFIED: `crawlq-ui/package.json` — Added playwright, @playwright/test, @aws-sdk/client-bedrock-runtime
**Key Decisions:**
- ADR-016 (HARD REQUIREMENT): CrawlQ and FrictionMelt visual tools COMPLETELY ISOLATED
- AWS Bedrock eu.anthropic.claude-opus-4-6-v1 in eu-central-1
- Tool LOCKED for permanent use — must run before every deployment
