# CrawlQ EU Chat Athena — Global Roadmap

## Project Goal
Deploy a fully functional EU-region GDPR-compliant AI chat system (Athena EU) with TRACE compliance protocol, document analysis, semantic graph building, and guest-to-authenticated user flow. All data stays in eu-central-1.

## Architecture (Post ADR-032 Repo Isolation)
- **Frontend:** Next.js 14 (SSR via AWS Amplify) — `crawlq-chat-athena-eu-frontend/`
- **Backend:** 25+ AWS Lambda functions (Python, ZIP deploy) — `crawlq-athena-eu-backend/SemanticGraphEU/`
- **Canvas:** TRACE Canvas app (separate track) — `crawlq-athena-eu-canvas/`
- **US App:** `crawlq-ui/` — READ-ONLY for EU work (ADR-032)
- **US Backend:** `crawlq-lambda/` — READ-ONLY for EU work (ADR-032)
- **AI Models:** Anthropic Claude (primary), Gemini (fallback), OpenAI (fallback)
- **Storage:** DynamoDB (eu-central-1), S3 (eu-central-1), Neo4j (EU instance)
- **Auth:** AWS Cognito (EU user pool: eu-central-1_Z0rehiDtA)
- **Region:** eu-central-1 only, isolated from US flows

## Milestones
- [x] Phase 1: Deploy 3 chat Lambda functions + DynamoDB tables (fixes 403)
- [x] Phase 2: Fix frontend chat integration (streaming, cancel, history)
- [x] Phase 3: EU guest document analysis flow (guest-to-auth transition)
- [x] Phase 4: Enhanced chat UI (sidebar, toolbar, code blocks, TRACE cards)
- [x] Phase 5: CI/CD infrastructure updates (deploy.sh, Dockerfiles, GitHub Actions)
- [x] Phase 6: Fix 6 broken Lambda functions (missing dependencies)
- [x] Phase 7: Comprehensive gap analysis + Phase 1 implementation plan (COMMIT 9-10)
- [x] Phase 8: Port US TRACE UI — ALL 5 SPRINTS COMPLETE (59 files, ~18,110 LoC, 208+ tests)
- [x] Phase 9: KG Exploration UI + TRACE Governance Runtime — ADR-023 (per-response KG, session KG, governance gate, circuit breaker, Merkle audit)
- [x] Phase 10: World-Class UI Revamp — ADR-024 "Trust by Design" (export PDF/DOCX/MD, artifact panel, command palette, suggested actions, profile/theme, search/feedback)
- [x] Phase 11: Production fixes — API Gateway routing (Function URLs 403), mermaid validation, 13 markdown capabilities, E2E testing (ADR-025, 87% confidence)
- [x] Phase 12: Production hardening — COMPLETE (8/8 items: CHAT-02 async mode ✓, AUTH-02 validation ✓, mobile UX ✓, CloudWatch alarms ✓, visual audit ✓, COMP-04 reasoner deployed without LangChain ✓, WEB-01 working as designed ✓, SNS notifications ✓) — Production readiness: 90%+
- [x] Phase 13: TRACE Canvas Sprint 1 MVP — COMPLETE (Workflow builder with React Flow, 3 node types, execution engine with topological sort, Zustand state management, DynamoDB persistence with multi-tenant isolation, EU Cognito authentication, 85%+ test coverage, zero breaking changes per ADR-028) — Ready for private beta
## Master Priority Order (ADR-031 — HARD REQUIREMENT)

**STRICT sequential. Each must be deployed before next begins.**

| Priority | Phase | Branch | Status | Gate Check |
|----------|-------|--------|--------|------------|
| **P1** | Phase 14: Remove Guest Flow + AI-First Onboarding | `feature-eu-standalone-app` | **DONE** (35/35 tests) | GDPR compliant ✓ Auth-only ✓ |
| **P2** | Phase 15: FrictionMelt Integration + Deploy | `feature-frictionmelt-integration` | **DONE** (MERGED) — Live E2E verified, 34/34 tests | Events emitting ✓ Insights displaying ✓ |
| **P3** | Phase 16: Canvas UI Synced + Deployed | `feature-trace-canvas` | **UNBLOCKED** — ready | Canvas loads TRACE + FM data? |
| **P4** | Phase 17: Full E2E Testing | — | BLOCKED by P3 | All critical paths pass? 90%+ confidence? |
| **P5** | Phase 18: Marketing, Website, Production Launch | — | BLOCKED by P4 | Tests green? Domain live? |

**Decision tree:** P1 deployed? → P2 deployed? → P3 deployed? → P4 passes? → P5 launch
**Anti-pattern:** DO NOT deploy later priority before earlier is complete + tested

## Key URLs
- **Athena EU Frontend:** https://main.d45bl3mgpjnhy.amplifyapp.com
- Amplify App ID: d45bl3mgpjnhy → repo: `crawlq-chat-athena-eu-frontend` (branch: main)
- Canvas: Not yet Amplify-deployed → repo: `crawlq-athena-eu-canvas`

## Key Decisions
- EU Lambda Function URLs use AuthType: NONE (no Authorization headers needed)
- Memory is opt-in by default (GDPR privacy-by-default)
- google-genai made conditional import (Gemini = optional fallback)
- python-magic replaced with mimetypes stdlib fallback
