# Checkpoint: research-frictionmelt-integration COMMIT 3 (FINAL)

**Timestamp:** 2026-02-12T21:00:00Z
**Branch:** research-frictionmelt-integration
**Parent:** feature-eu-standalone-app
**Status:** ✅ **APPROVED** (Conditional on ADR-026 strict isolation)

## Milestone

Plan APPROVED with critical clarification: **Strict Product Isolation** enforced via ADR-026 (HARD REQUIREMENT).

## User Feedback Incorporated

**Critical Clarification Received**:
1. **Both products are different** — FrictionMelt and Athena EU must remain separate
2. **Data structure understanding needed** — Clear documentation of what each product sends/receives
3. **Build isolation rule** — Do NOT build FrictionMelt components in Athena EU codebase
4. **Separate instructions** — Provide Claude Code-based files for FrictionMelt work separately
5. **ADR requirement** — Record isolation rule as decision and follow strictly

## Response: Files Created

### **1. ADR-026: Strict Product Isolation (HARD REQUIREMENT)**

**File**: [.gsm/decisions/ADR-026-frictionmelt-athena-integration-architecture.md](.gsm/decisions/ADR-026-frictionmelt-athena-integration-architecture.md)

**Key Principles**:
- ✅ **Athena EU codebase (THIS repo)**: Event emission, insights consumption, UI integration
- ❌ **Athena EU codebase NEVER contains**: FrictionMelt ingestion API, analytics engine, DynamoDB schemas
- ✅ **FrictionMelt codebase (SEPARATE repo)**: Ingestion API, analytics engine, insights generation
- 🔒 **Enforcement**: Same level as ADR-013 (US Non-Interference), ADR-016 (Visual Tool Isolation)

**Interface Contracts** (versioned JSON schemas in THIS repo):
- `shared/schemas/v1/friction-event-schema.json` — Athena EU → FrictionMelt
- `shared/schemas/v1/friction-enrichment-schema.json` — FrictionMelt → Athena EU (enrichment response)
- `shared/schemas/v1/friction-insights-schema.json` — FrictionMelt → Athena EU (insights API)

**Data Structure Documentation**:
- **Current LLM pipeline format**: Minimal (eventId, timestamp, traceComponent, eventType, orgId)
- **Enhanced format**: Richer (+ context, frictionSignals, metadata)
- **Evolution strategy**: Start with current, incrementally add fields. FrictionMelt supports both (backward compatibility).

### **2. FrictionMelt Build Instructions (SEPARATE)**

**File**: [docs/frictionmelt-integration/FRICTIONMELT-BUILD-INSTRUCTIONS.md](docs/frictionmelt-integration/FRICTIONMELT-BUILD-INSTRUCTIONS.md)

**8,000 words** of detailed instructions for FrictionMelt team:
- Component 1: Athena EU Connector Ingestion Lambda (`POST /ingest`)
- Component 2: Friction Analytics Engine (ARIMA forecasting, XGBoost classification)
- Component 3: Athena EU Insights API Lambda (`GET /insights/{orgId}`)
- Component 4: Industry Benchmark Aggregator (cross-org anonymization)
- DynamoDB schemas: `fm-friction-records`, `fm-org-insights`, `fm-org-patterns`, `fm-industry-benchmarks`
- Testing strategy, deployment, monitoring, rollout plan

**CRITICAL**: These components are **NOT built in THIS codebase**. Separate Claude Code session for FrictionMelt team.

## Implementation Boundaries (Clear Separation)

### **Athena EU Responsibilities (THIS REPO)**

| Component | File | Purpose |
|-----------|------|---------|
| Event Emitter | `SemanticGraphEU/shared/friction_event_emitter.py` | Sends events TO FrictionMelt |
| Event Batcher | `SemanticGraphEU/EUFrictionEventBatcher/handler.py` | EventBridge cron every 5min, batches + POSTs to FrictionMelt API |
| Insights Poller | `SemanticGraphEU/EUFrictionInsightsPoller/handler.py` | EventBridge cron every 6hr, GETs from FrictionMelt API, stores in `eu-friction-insights` |
| TRACE Dashboard | `crawlq-chat-athena-eu-frontend/src/components/friction/TRACEEffectivenessDashboard.tsx` | Displays FrictionMelt insights |
| DynamoDB Tables | `eu-friction-events`, `eu-friction-insights` | Athena EU owns these |

### **FrictionMelt Responsibilities (SEPARATE REPO)**

| Component | File | Purpose |
|-----------|------|---------|
| Ingestion API | `frictionmelt/lambdas/athena_eu_connector_ingest/handler.py` | POST /v1/connectors/athena-eu/ingest |
| Analytics Engine | `frictionmelt/lambdas/friction_analytics_engine/handler.py` | ARIMA+XGBoost forecasting, pattern matching |
| Insights API | `frictionmelt/lambdas/athena_eu_insights_api/handler.py` | GET /v1/connectors/athena-eu/insights/{orgId} |
| Benchmark Aggregator | `frictionmelt/lambdas/industry_benchmark_aggregator/handler.py` | Cross-org anonymization, differential privacy |
| DynamoDB Tables | `fm-friction-records`, `fm-org-insights`, `fm-org-patterns`, `fm-industry-benchmarks` | FrictionMelt owns these |

## Deployment Independence

**Athena EU Deployment** (THIS repo):
- Can deploy event emission changes WITHOUT FrictionMelt changes ✅
- Can deploy UI updates independently ✅
- DynamoDB tables managed by Athena EU team ✅

**FrictionMelt Deployment** (SEPARATE repo):
- Can deploy ingestion API changes WITHOUT Athena EU changes ✅
- Can deploy analytics engine updates independently ✅
- API versioning allows backward compatibility ✅

**Interface Versioning**:
- JSON schemas versioned in THIS repo (`shared/schemas/v1/`, `v2/`)
- Both products support current + N-1 version (rolling upgrade)
- Breaking changes require coordinated release (rare, documented)

## Code Review Enforcement Checklist

**Before merging ANY PR to Athena EU**:
- [ ] Does this PR add FrictionMelt ingestion logic to Athena EU? ❌ **REJECT**
- [ ] Does this PR add FrictionMelt DynamoDB schemas to Athena EU? ❌ **REJECT**
- [ ] Does this PR modify FrictionMelt API implementation in Athena EU? ❌ **REJECT**
- [ ] Does this PR only emit events or consume insights? ✅ **APPROVE**
- [ ] Does this PR update interface contract schemas? ✅ **APPROVE** (notify FrictionMelt team)

## Next Steps (Phase 1 Sprint 1 — Ready to Start)

### **Week 1 Actions (Athena EU Team — THIS REPO)**

**Day 1-2: Event Emission Infrastructure**
- [ ] Create DynamoDB table `eu-friction-events` (schema: eventId, timestamp, orgId, event_data, ttl)
- [ ] Create `SemanticGraphEU/shared/friction_event_emitter.py` (FrictionEventEmitter class)
- [ ] Create `SemanticGraphEU/EUFrictionEventBatcher/handler.py` (EventBridge cron every 5min)
- [ ] Create JSON schemas in `shared/schemas/v1/friction-*.json`

**Day 3-4: Event Trigger Points**
- [ ] Instrument `eu_chat_athena_bot/handler.py` — emit "user_override" event
- [ ] Instrument `ResponseFeedback.tsx` — emit "feedback" event (thumbs down)
- [ ] Instrument `TraceDashboardEU.tsx` — emit "challenge" event ("Challenge This" clicked)
- [ ] Instrument `eu_response_kg_extractor` — emit "kg_low_confidence" event

**Day 5: Testing**
- [ ] Unit test: FrictionEventEmitter formats JSON correctly
- [ ] Integration test: Event appears in `eu-friction-events` table
- [ ] Mock FrictionMelt API: Use `responses` library to mock `/ingest` endpoint

### **Week 1 Actions (FrictionMelt Team — SEPARATE REPO)**

**Use instructions from**: [docs/frictionmelt-integration/FRICTIONMELT-BUILD-INSTRUCTIONS.md](docs/frictionmelt-integration/FRICTIONMELT-BUILD-INSTRUCTIONS.md)

- [ ] Create DynamoDB table `fm-friction-records`
- [ ] Create Lambda `fm_athena_eu_connector_ingest` (POST /v1/connectors/athena-eu/ingest)
- [ ] Implement JSON schema validation
- [ ] Implement friction pattern classification (map traceComponent + eventType → friction pattern)
- [ ] Return enrichment response (pattern match, suggested resolution)

### **Week 1 Coordination**

- [ ] **Athena EU team**: Provide test events to FrictionMelt team (sample JSON)
- [ ] **FrictionMelt team**: Provide staging API endpoint + test API key
- [ ] **Both teams**: Run contract tests (Pact or similar) to verify API compatibility

## Success Metrics (Week 4 Targets)

| Metric | Target |
|--------|--------|
| Events streamed to FrictionMelt | 100/day |
| Enrichment API latency (p95) | <2s |
| Event batching reliability | 90% |
| TRACE event instrumentation coverage | 60% (6+ event types) |

## Files Changed (This Session)

- CREATED: `.gsm/decisions/ADR-026-frictionmelt-athena-integration-architecture.md`
- CREATED: `docs/frictionmelt-integration/FRICTIONMELT-BUILD-INSTRUCTIONS.md`
- MODIFIED: `.gcc/branches/research-frictionmelt-integration/commit.md` (reflect conditional approval)

## Approval Status

✅ **APPROVED by user** with conditions:
1. ADR-026 strict isolation enforced ✅ (HARD REQUIREMENT created)
2. Data structure understanding documented ✅ (JSON schemas + current/enhanced formats)
3. Build isolation maintained ✅ (separate instructions for FrictionMelt)
4. Separation of responsibilities clear ✅ (Athena EU emits/consumes, FrictionMelt ingests/analyzes)

**Ready to proceed with Phase 1 Sprint 1 implementation.**

## Blockers

None. All research questions can be answered during Phase 1 implementation (FrictionMelt hosting location, API availability, etc. will be discovered when FrictionMelt team starts build).

---

**Session Status**: COMPLETE
**Branch Status**: APPROVED
**Next Action**: Start Phase 1 Sprint 1 — Event emission infrastructure (Athena EU side)
