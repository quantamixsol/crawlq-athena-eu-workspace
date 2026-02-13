# ADR-001: EU Lambda Function URLs use AuthType: NONE
**Date:** 2026-02-09 | **Status:** ACCEPTED
**Context:** EU Lambda functions need to be accessible from the Amplify-hosted frontend. Options: (1) AuthType: NONE with CORS, (2) AuthType: AWS_IAM with SigV4 signing.
**Decision:** AuthType: NONE with permissive CORS. JWT validation happens inside the Lambda handler via shared/jwt_validator.py, not at the Function URL layer.
**Consequences:**
- (+) Simpler frontend — no Authorization headers needed in fetch/axios calls
- (+) No IAM credential management on client side
- (-) Lambda must validate JWT tokens internally
- (-) Function URL is publicly accessible (but requires valid JWT in body)
