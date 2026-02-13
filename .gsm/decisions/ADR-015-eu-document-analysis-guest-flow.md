# ADR-015: EU Document Analysis & Guest Flow Enhancements
**Date:** 2026-02-11 | **Status:** ACCEPTED

## Context
Sprint 3 and Sprint 4 implement document analysis and guest flow components for the EU region, replicating and enhancing the US approach with full TRACE protocol compliance and GDPR-aware consent management.

## Decision

### Document Analysis (Sprint 3)
1. **5-level severity** (CRITICAL/HIGH/MEDIUM/LOW/INFO) vs US 3-level (HIGH/MEDIUM/LOW)
2. **Compliance checkpoints with status badges** (PASS/FAIL/WARNING/PENDING) vs US plain text list
3. **TraceDashboardEU with regulation subtitles** on every TRACE pillar panel
4. **EU-specific prompt builder** with Art. 13/14/50 + GDPR 15/22 + 5-tier confidence + 7-year retention
5. **Sort + Filter capabilities** on insight list (US has none)
6. **Animated ScoreCardEU** with count animation and 5-tier tier badges
7. **SHA-256 audit hash** field in auditability panel

### Guest Flow (Sprint 4)
1. **GDPR consent step** (3 checkboxes: data processing, AI analysis, temporary storage) before upload
2. **EU-specific processing phrases** with TRACE protocol step indicators
3. **Locked insights CTA** with visible count and unlock button
4. **GuestConversionEU** with 6 benefits highlighting TRACE features
5. **Session-only storage** messaging throughout
6. **Data residency notice** (eu-central-1) on all screens

## Consequences
- **Positive:** Full GDPR compliance, superior UX, complete TRACE protocol integration
- **Positive:** No US code modified (ADR-013 compliant)
- **Negative:** 21 additional files increase bundle size (~estimated 50KB gzipped)
