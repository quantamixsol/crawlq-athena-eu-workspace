# ADR-037: Strategic Gap Analysis — EU Launch Readiness Assessment

**Date:** 2026-02-15 | **Status:** ACTIVE (Living Document)
**Context:** Comprehensive audit of TRACE + FrictionMelt + Canvas integration readiness for EU product launch (target: Feb 22, 2026)
**Decision:** Systematic gap closure with continuous Pallas testing until 100% launch readiness

---

## 1. AUDIT METHODOLOGY

- **GCC Context:** All 14 branches read, 46+ commits analyzed, 80+ checkpoints reviewed
- **GSM Strategy Docs:** TRACE Friction Framework, FrictionMelt Integration Strategy, CrawlQ Messaging Platform — all 3 summaries cross-referenced
- **Pallas E2E Testing:** 10 test users, 260 assertions, 12 test executions
- **Git Hygiene:** 5 repos audited for uncommitted work
- **Build Verification:** Frontend build confirmed 0 errors, 13 routes

---

## 2. PALLAS TEST FLEET RESULTS (2026-02-15)

| Metric | Value |
|--------|-------|
| Total Test Users | 10 |
| Total Assertions | 260 |
| PASS | 228 (87.7%) |
| WARN | 32 (12.3%) |
| FAIL | 0 (0%) |

### Per-User Scorecard

| User | Focus | Pass | Warn | Fail |
|------|-------|------|------|------|
| Aria Testos | Auth/Login | 22 | 4 | 0 |
| Bruno Verity | Workspace CRUD | 23 | 3 | 0 |
| Clara Insights | Chat/Streaming | 23 | 3 | 0 |
| Damon Trace | TRACE/Audit | 22 | 4 | 0 |
| Elena Graph | KG/Neo4j | 23 | 3 | 0 |
| Felix Export | PDF/DOCX/MD | 23 | 3 | 0 |
| Greta Comply | GDPR/EU AI Act | 23 | 3 | 0 |
| Hugo Upload | Doc Upload | 23 | 3 | 0 |
| Iris Session | Sessions | 23 | 3 | 0 |
| Jules Mobile | Mobile/Responsive | 23 | 3 | 0 |

### Systemic Issues (100% of users)

1. **KG Source Filter** — 0/6 checks, component not rendering
2. **Query Count** — not displaying, `/query-usage` CORS blocked
3. **Auto-naming** — partial/delayed session names
4. **Canvas list API** — 500 error (graceful fallback masks it)

---

## 3. STRATEGIC GAP MATRIX

### A. TRACE Compliance Protocol (ADR-006)

| Pillar | Target | Current | Gap | Severity | Fix |
|--------|--------|---------|-----|----------|-----|
| T — Transparency | AI confidence, data sources, model shown | Streaming + async return TRACE dimensions | Verify display in production | MEDIUM | Test |
| R — Reasoning | Decision chain, "Ask Why", KG traversal | KG Source Filter broken (0/6 Pallas) | **Component not rendering** | HIGH | FIX #7 |
| A — Auditability | Merkle audit trail, decision lineage DAG | Audit trail UI renders, tier-gated | Working as designed | LOW | None |
| C — Compliance | GDPR/EU AI Act embedded | Consent in profile settings | **Consent not persistent** | HIGH | FIX #8 |
| E — Explainability | Step-by-step breakdown, suggested actions | SuggestedActions component working | Working | LOW | None |

### B. FrictionMelt Integration (ADR-026)

| Component | Target | Current | Gap | Status |
|-----------|--------|---------|-----|--------|
| Event Emission | 10 event types → DynamoDB | FrictionEventEmitter deployed, v2.0 events | None | DONE |
| Event Batcher | 5-min EventBridge → FM API | Lambda deployed, real API live | None | DONE |
| Insights Dashboard | TRACE Effectiveness per pillar | Mock+live fallback | Verify live data | MONITOR |
| Developer Hub | 6-tab wiki | Complete, in sidebar | None | DONE |
| 91-Pattern Taxonomy | Full classification | JSON sent, rule-based confirmed | None | DONE |

### C. Canvas Integration (ADR-036)

| Component | Target | Current | Gap | Severity | Fix |
|-----------|--------|---------|-----|----------|-----|
| Code Merge | 44 files → main app | COMMIT 24 complete | None | — | DONE |
| Tier Gating | Explorer: 1 canvas, 3 runs | eu-plans.ts updated | Not enforced at API | LOW | Phase 2 |
| Sidebar Nav | "Canvas" link in main app | Cross-app nav works | **No persistent sidebar link** | HIGH | FIX #10 |
| Real LLM | EU Chat Lambda | CANVAS_MOCK_LLM=true | **Still in mock mode** | HIGH | FIX #5 |
| Canvas List API | Load user canvases | **500 error on main app** | **DynamoDB client issue** | CRITICAL | FIX #2 |

### D. EU AI Act Compliance

| Requirement | Article | Current | Gap | Severity | Fix |
|-------------|---------|---------|-----|----------|-----|
| Human Oversight | Art. 14 | human_review flag in JSON | **No UI indicator** | HIGH | FIX #6 |
| Transparency | Art. 13 | TRACE card shows model + confidence | Working | LOW | None |
| Data Governance | Art. 10 | EU-only data residency (eu-central-1) | Working | LOW | None |
| Risk Assessment | Art. 9 | TRACE compliance scoring | Working | LOW | None |

### E. Infrastructure & Git Hygiene

| Issue | Current | Gap | Severity | Fix |
|-------|---------|-----|----------|-----|
| Backend git | 5 modified files uncommitted | **Deployed code ≠ git** | CRITICAL | FIX #3 |
| Lambda git | 52+ untracked files | **Entire subsystems unversioned** | CRITICAL | FIX #3 |
| CORS | /query-usage missing headers | **100% users affected** | HIGH | FIX #1 |
| Session Lambdas | Built but deployment unverified | **Unknown state** | HIGH | FIX #4 |

---

## 4. PRIORITIZED FIX LIST

| # | Fix | Category | Est. | Impact | Launch-Block? |
|---|-----|----------|------|--------|---------------|
| 1 | `/query-usage` CORS on API Gateway | Infrastructure | 30 min | 100% users | YES |
| 2 | Canvas list API 500 on main app | Canvas | 1-2 hr | Canvas broken | YES |
| 3 | Git commit all uncommitted backend code | Infrastructure | 1 hr | Disaster risk | YES |
| 4 | Verify session/archetype Lambda deployments | Onboarding | 1 hr | Onboarding broken? | YES |
| 5 | Set CANVAS_MOCK_LLM=false on Amplify | Canvas | 15 min | Canvas value prop | YES |
| 6 | Human Review UI indicator (EU AI Act Art. 14) | Compliance | 2-3 hr | Legal compliance | YES |
| 7 | KG Source Filter fix (0/6 Pallas) | TRACE | 2-4 hr | Reasoning pillar | YES |
| 8 | GDPR consent persistence to DynamoDB | Compliance | 2-3 hr | GDPR audit | YES |
| 9 | Session auto-naming fix | UX | 1-2 hr | UX quality | NO |
| 10 | Canvas sidebar nav link in main app | Canvas | 1 hr | Discoverability | NO |
| 11 | Workspace list race condition (20% users) | UX | 1-2 hr | UX reliability | NO |
| 12 | FrictionMelt insights endpoint verification | FrictionMelt | 30 min | Flywheel proof | NO |

---

## 5. LAUNCH READINESS SCORECARD

| Category | Current | Target | Status |
|----------|---------|--------|--------|
| Authentication | 95% | 95% | PASS |
| Core Chat | 90% | 90% | PASS |
| TRACE Protocol | 75% | 90% | GAP — Fix #7 |
| Canvas | 70% | 85% | GAP — Fix #2, #5, #10 |
| FrictionMelt | 85% | 80% | PASS |
| GDPR/EU AI Act | 70% | 95% | GAP — Fix #6, #8 |
| Infrastructure | 80% | 95% | GAP — Fix #1, #3 |
| UX/Mobile | 85% | 85% | PASS |
| Monitoring | 90% | 85% | PASS |
| **Overall** | **82%** | **100%** | **18% gap** |

---

## 6. PROGRESS TRACKER (Updated After Each Fix)

| Fix # | Started | Completed | Verified | Pallas Retest |
|-------|---------|-----------|----------|---------------|
| 1 | | | | |
| 2 | | | | |
| 3 | | | | |
| 4 | | | | |
| 5 | | | | |
| 6 | | | | |
| 7 | | | | |
| 8 | | | | |
| 9 | | | | |
| 10 | | | | |
| 11 | | | | |
| 12 | | | | |

---

## 7. TESTING CADENCE

- After each fix: Targeted Pallas test (affected user + 1 regression user)
- After every 3 fixes: Full 10-user Pallas regression
- Final gate: 10-user regression with 95%+ pass rate (0 FAIL, <5% WARN)
- Launch gate: 98%+ pass rate

---

## Consequences

**Positive:**
- Systematic, measurable path to launch
- Every gap has owner, timeline, and verification method
- Continuous testing prevents regression

**Negative:**
- 12 fixes in 7 days requires sustained focus
- Some fixes (consent persistence, human review UI) require new code
- Git hygiene fix requires careful staging to avoid breaking deployed code

**Risk Mitigation:**
- Fix launch-blockers first (Days 1-3), UX polish after (Days 4-5)
- Each fix gets its own Pallas verification before moving to next
- GCC COMMIT after every fix for crash recovery

---

## 8. HARD RULES (ADR-032 + ADR-013 Enforced)

1. **crawlq-ui** and **crawlq-lambda** are **READ-ONLY**. NEVER modify, commit, or push to these repos.
2. All fixes apply ONLY to the 3 EU repos: `crawlq-chat-athena-eu-frontend`, `crawlq-athena-eu-backend`, `crawlq-athena-eu-canvas`
3. If EU code was developed in crawlq-ui/crawlq-lambda but not yet migrated, EXTRACT it into the correct EU repo. Never modify the source.
4. FIX #3 scope: ONLY `crawlq-athena-eu-backend` (5 EU-specific modified files). The 52+ untracked files in crawlq-lambda are US-side work and must not be touched.
