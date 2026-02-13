# ADR-031: Master Delivery Priority Order — Athena EU Production Launch

**Date:** 2026-02-13 | **Status:** ACCEPTED (HARD REQUIREMENT)

## Context
Multiple workstreams are in progress: AI-first onboarding (replacing non-compliant guest flow), FrictionMelt integration, Canvas UI, and marketing/website. Without a locked execution order, there is risk of context-switching between features, deploying untested combinations, and shipping GDPR-non-compliant guest flows to production.

The guest flow was tested and **failed** — it allows users to upload documents and provide information without signing up, which violates GDPR requirements (no lawful basis for processing without consent + account).

## Decision

**STRICT sequential priority order. Each priority MUST be deployed and working before the next begins.**

### P1: Remove Guest Flow + AI-First Onboarding [BLOCKING]
- **Why first:** GDPR compliance risk. Guest flow is non-compliant.
- **Scope:** Remove guest-eu route, build GDPR consent page from home, smart signup flow, enterprise onboarding assessment, user archetype in DynamoDB, guided first upload
- **Definition of Done:** User clicks from home → GDPR consent → signup → onboarding assessment → workspace → first document upload. No guest/unsigned path exists.
- **GCC Branch:** `feature-onboarding-v2`
- **Blocks:** Everything else (no point deploying integrations on a non-compliant base)

### P2: FrictionMelt Integration [AFTER P1 deployed]
- **Why second:** Core product differentiator. Needs authenticated users (from P1) to function.
- **Scope:** Complete event emission, insights consumption, FrictionMelt UI panels in Athena EU
- **Definition of Done:** Authenticated user uploads doc → TRACE analysis → FrictionMelt patterns detected → insights displayed
- **GCC Branch:** `feature-frictionmelt-integration` (existing)
- **Depends on:** P1 (authenticated user flow)

### P3: Canvas UI Experience [AFTER P2 deployed]
- **Why third:** Premium feature layer. Needs TRACE data + FrictionMelt insights to visualize.
- **Scope:** Sync Canvas with main app, workflow chaining, template library, full persistence
- **Definition of Done:** User creates canvas → loads TRACE data → builds workflows → saves/loads
- **GCC Branch:** `feature-trace-canvas` (existing, Sprint 2 in progress)
- **Depends on:** P2 (FrictionMelt data feeds canvas)

### P4: Full End-to-End Testing [AFTER P3 deployed]
- **Why fourth:** Validates the complete integrated experience.
- **Scope:** Cognito auth flow, document upload, TRACE analysis, FrictionMelt insights, Canvas workflows, multi-session workspaces, mobile responsiveness
- **Test layers:** Smoke → Integration → E2E → Visual audit → Load test
- **Definition of Done:** All critical paths pass, 90%+ confidence score
- **Depends on:** P1 + P2 + P3 all deployed

### P5: Marketing, Website, Full Production [AFTER P4 passes]
- **Why last:** Don't market what isn't tested.
- **Scope:** crawlq.ai subdomain, SEO, landing pages, pricing, sales enablement
- **Definition of Done:** Public-facing site live, production domain configured, monitoring active
- **Depends on:** P4 (clean test results)

## Decision Tree (Gate Checks)

```
START
  │
  ├─ P1: Onboarding deployed?
  │   NO → Work on P1 ONLY
  │   YES ↓
  │
  ├─ P2: FrictionMelt deployed?
  │   NO → Work on P2 ONLY
  │   YES ↓
  │
  ├─ P3: Canvas deployed?
  │   NO → Work on P3 ONLY
  │   YES ↓
  │
  ├─ P4: E2E tests pass?
  │   NO → Fix failures, re-test
  │   YES ↓
  │
  └─ P5: Marketing & launch
```

## Consequences

**Positive:**
- GDPR compliance fixed FIRST (legal risk eliminated)
- Each layer builds on the previous (no broken integrations)
- Clear gate checks prevent premature feature shipping
- Testing happens on the complete product, not fragments

**Negative:**
- Sequential means later priorities wait (Canvas team blocked until P2 done)
- No parallel feature development across priorities
- Mitigation: Research/design for P2-P5 CAN happen in parallel, but DEPLOYMENT is sequential

## Anti-Patterns to Avoid
- DO NOT deploy FrictionMelt before onboarding is live
- DO NOT work on Canvas deployment while FrictionMelt is incomplete
- DO NOT start marketing before E2E testing passes
- DO NOT re-enable guest flow under any circumstances
