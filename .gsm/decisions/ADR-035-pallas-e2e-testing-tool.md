# ADR-035: Pallas is the E2E Testing Tool for All CrawlQ Projects
**Date:** 2026-02-14 | **Status:** ACCEPTED

**Context:** We need a standardized E2E visual testing tool for the TRACE Canvas app. Two candidates exist: Crucible (FrictionMelt's tool) and Pallas (Athena EU's tool). The user explicitly mandated Pallas, not Crucible.

**Decision:** Pallas (`crawlq-chat-athena-eu-frontend/scripts/pallas/`) is the mandatory E2E testing tool for all CrawlQ projects including TRACE Canvas. Crucible must NOT be used in any CrawlQ repository. Pallas will be extended with Canvas-specific test suites.

**Consequences:**
- (+) Single testing tool across all CrawlQ projects (Chat + Canvas)
- (+) Same Cognito pool (`eu-central-1_Z0rehiDtA`) — test users work on both apps
- (+) Project isolation from FrictionMelt (ADR-001, ADR-032, ADR-033 compliant)
- (-) Pallas needs Canvas-specific test methods (node drag/drop, health badge, coach, etc.)
- (-) Canvas Amplify URL different from Chat URL — config must support multiple targets
