# GSM Document Index — CrawlQ EU Chat Athena

## Architecture Decision Records

| ID | Title | Type | Tags | Status |
|----|-------|------|------|--------|
| ADR-001 | EU Lambda Function URLs AuthType: NONE | decision | lambda, security, cors | ACCEPTED |
| ADR-002 | google-genai Made Optional Import | decision | lambda, dependencies, gemini | ACCEPTED |
| ADR-003 | Chat Memory Default to Opt-In | decision | gdpr, privacy, frontend | ACCEPTED |
| ADR-004 | ZIP-Based Lambda Deployment from Windows | decision | lambda, deploy, ci-cd | ACCEPTED |
| ADR-005 | EU Region Isolation Strategy | decision | architecture, gdpr, isolation | ACCEPTED |
| ADR-006 | TRACE Compliance Protocol for EU AI Act | decision | compliance, eu-ai-act, trace | ACCEPTED |
| ADR-007 | LLM Fallback Chain (Anthropic -> Gemini -> OpenAI) | decision | llm, availability, fallback | ACCEPTED |
| ADR-008 | Claude Opus 4.6 as Primary EU Bedrock Model | decision | bedrock, model, opus | ACCEPTED |
| ADR-009 | Guest-Facing EU Lambdas Use AuthType: NONE | decision | lambda, security, guest | SUPERSEDED by ADR-010 |
| ADR-010 | API Gateway HTTP API + Cognito JWT Authorizer | decision | api-gateway, security, cognito, jwt | ACCEPTED |
| ADR-011 | API Gateway 30s Timeout Handling Strategy | decision | api-gateway, timeout, streaming, lambda | ACCEPTED |
| ADR-012 | Tier 3 Enterprise Async Architecture + Intelligent Markdown | decision | architecture, async, sqs, markdown, rendering | ACCEPTED (separate branch) |
| ADR-013 | US Region Non-Interference Policy | decision | architecture, policy, us-region, isolation | ACCEPTED (HARD REQUIREMENT) |
| ADR-014 | Use boto3 for Lambda Deployment Over AWS CLI | decision | deployment, boto3, aws-cli, automation | ACCEPTED |
| ADR-014 | EU Knowledge Graph Enhancements | decision | knowledge-graph, visualization, eu | ACCEPTED |
| ADR-015 | EU Document Analysis + Guest Flow Enhancements | decision | document-analysis, guest-flow, eu | ACCEPTED |
| ADR-016 | Visual UI Test Tool — Project Isolation (CrawlQ vs FrictionMelt) | decision | testing, visual-audit, isolation, playwright, bedrock | ACCEPTED (HARD REQUIREMENT) |
| ADR-017 | EU Standalone App Extraction Strategy | decision | architecture, extraction, repo-separation, eu | ACCEPTED |
| ADR-018 | Lambda Function URLs for Long-Running Operations | decision | lambda, function-url, timeout, bedrock | ACCEPTED |
| ADR-019 | EU LLM Fallback Strategy | decision | llm, bedrock, fallback, eu | ACCEPTED |
| ADR-020 | Rolling Context Window Analysis | decision | context, analysis, rolling | ACCEPTED |
| ADR-021 | Standard Lambda Deployment Tool (boto3 + ZIP + Function URLs) | decision | deployment, boto3, lambda, function-url, standard-tool | ACCEPTED |
| ADR-022 | Chat Response Mode Rules (Web Search / TRACE / Combined) | decision | chat, web-search, trace, modes, ux | ACCEPTED |
| ADR-023 | KG Exploration UI + TRACE Governance Runtime | decision | knowledge-graph, trace, governance, circuit-breaker, merkle, lineage | ACCEPTED |
| ADR-024 | World-Class UI Revamp — Trust by Design | decision | ui, ux, export, artifact-panel, command-palette, keyboard-shortcuts, profile, feedback | ACCEPTED |
| ADR-025 | Production Testing Strategy — 5-Layer Pyramid | decision | testing, smoke-test, e2e, visual-audit, cost-control | ACCEPTED |
| ADR-026 | FrictionMelt × Athena EU Integration Architecture | decision | frictionmelt, integration, closed-loop, data-moat, revenue | PLANNED (research-frictionmelt-integration branch) |
| ADR-029 | Async Chat Result Format: JSON Wrapper for Markdown + Metadata | decision | async-chat, json, trace, metadata, s3 | ACCEPTED |
| ADR-030 | EU Reasoner: LangChain Dependency Removed (Rule-Based Python) | decision | lambda, reasoner, langchain, dependencies, rule-based, deployment | ACCEPTED |
| ADR-031 | Master Delivery Priority Order — P1:Onboarding → P2:FrictionMelt → P3:Canvas → P4:E2E → P5:Launch | decision | priority, governance, delivery-order, gdpr, compliance | ACCEPTED (HARD REQUIREMENT) |
| ADR-034 | BTDI Workflow Standardization for Canvas | decision | canvas, workflow, btdi, process | ACCEPTED |
| ADR-035 | Pallas E2E as Sole Testing Tool (Not Crucible) | decision | testing, pallas, e2e, playwright | ACCEPTED |
| ADR-036 | Canvas Integration into Athena EU Main App | decision | canvas, monorepo, tier-gating, integration | ACCEPTED |
| ADR-037 | Strategic Gap Analysis — EU Launch Readiness | audit + living-doc | launch, gaps, compliance, trace, canvas, testing | ACTIVE (Living Document) |
| ADR-039 | OWL/SHACL Foundation — Python-Side Ontology Validation | decision | ontology, owl, shacl, neo4j, validation | ACCEPTED |
| ADR-040 | Explainability Metrics Pipeline | decision | explainability, metrics, trace, scoring | ACCEPTED |
| ADR-041 | Cache Invalidation Deployment Rule | decision + enforcement | cache, cloudfront, deployment, amplify, browser-cache | ACCEPTED (HARD REQUIREMENT) |
| ADR-042 | EU System E2E Inventory & Configuration Audit | audit + living-doc | lambda, api-gateway, amplify, endpoints, inventory, testing | ACCEPTED (Living Document) |

## External Documents

| Filename | Type | Added | Summary |
|----------|------|-------|---------|
| Enterprise AI Implementation Playbook_v1_29012026.pdf | strategy | 2026-01-29 | TRACE framework specification and enterprise AI implementation guide |
| CrawlQ_Messaging_Platform_Summary.md | strategy | 2026-02-12 | Brand voice, typography (Plus Jakarta Sans), color system (Navy/Blue), copy guidelines, design tokens |
| FrictionMelt-AthenaEU-Integration-Strategy.md.pdf | strategy + architecture + revenue | 2026-02-12 | Closed-loop data flywheel, dual-product competitive moat, 7 revenue streams ($2.4M ARR Year 1), API contracts, sales playbooks |
| TRACE Friction Framework.pdf | research + framework | 2026-02-12 | 58 friction patterns mapped to TRACE components, 8-layer taxonomy, 7.4x ROI prevention vs. remediation |
| 2026-02-15_Strategic-Gap-Analysis-Launch-Readiness.md | audit + living-doc | 2026-02-15 | 12-gap launch readiness matrix, Pallas 228/0/32 baseline, 82% current score, 7-day sprint plan |
| 2026-02-16_EU-System-E2E-Inventory.md | architecture + living-doc | 2026-02-16 | 36 Lambda inventory, 30 frontend endpoints, API Gateway routes, Amplify env vars, test infrastructure, inter-invocation map |
| 2026-02-16_EU-E2E-Workflow-Audit-Report.md | audit + action-plan | 2026-02-16 | 4-workflow parallel audit: 17 CRITICAL + 17 HIGH + 29 MEDIUM issues, 6-phase action plan (283h), security/data/compliance/gating findings |

## Summaries
| Filename | Source | Tags |
|----------|--------|------|
| CrawlQ_Messaging_Platform_Summary.summary.md | .gsm/external/CrawlQ_Messaging_Platform_Summary.md | branding, voice, typography, design-system |
| FrictionMelt-AthenaEU-Integration-Strategy.summary.md | .gsm/external/FrictionMelt-AthenaEU-Integration-Strategy.md.pdf | frictionmelt, athena-eu, integration, closed-loop, revenue-model, data-moat, trace-friction-framework |
| TRACE-Friction-Framework.summary.md | .gsm/external/TRACE Friction Framework.pdf | trace, friction, research-paper, 58-patterns, adoption, compliance, explainability |
| EU-System-E2E-Inventory.summary.md | .gsm/external/2026-02-16_EU-System-E2E-Inventory.md | lambda, api-gateway, amplify, inventory, endpoints, testing, e2e |
