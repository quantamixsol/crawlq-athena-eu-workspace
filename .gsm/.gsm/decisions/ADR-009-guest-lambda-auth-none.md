# ADR-009: Guest-Facing EU Lambdas Use AuthType: NONE
**Date:** 2026-02-10 | **Status:** SUPERSEDED by ADR-010
**Context:** 14 of 17 EU Lambda Function URLs had AuthType: AWS_IAM, requiring SigV4 request signing. This blocked browser-based calls (guest upload, onboard, get documents) since the frontend doesn't have IAM credentials.
**Decision:** Changed AuthType from AWS_IAM to NONE for 3 guest-facing Lambdas:
- `eu_upload_deep_document` — needed for guest document upload
- `eu_onboard_user` — needed for post-login guest data transfer
- `eu_get_deep_documents` — needed for document listing
All other 11 Lambdas (internal: graph builder, process doc, compliance, audit, etc.) retain AWS_IAM auth as they are only invoked by other Lambdas, not the browser.
**Consequences:**
- (+) Guest upload flow now works from browser
- (+) Matches US architecture pattern (US endpoints also use NONE)
- (+) JWT validation still happens inside Lambda handlers
- (-) Function URLs are publicly reachable (mitigated by handler-level auth)
- (-) Rate limiting should be added (future work)
