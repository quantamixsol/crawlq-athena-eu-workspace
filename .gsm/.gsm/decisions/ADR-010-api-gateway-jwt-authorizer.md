# ADR-010: API Gateway HTTP API with Cognito JWT Authorizer
**Date:** 2026-02-10 | **Status:** ACCEPTED
**Supersedes:** ADR-009 (Guest-Facing EU Lambdas Use AuthType: NONE)

**Context:**
All 17 EU Lambda Function URLs return 403 Forbidden for unauthenticated requests, even when AuthType is set to NONE with a correct public resource policy. The AWS Lambda service in eu-central-1 appears to block public Function URL access at the account level (similar to S3 Block Public Access). Direct SDK invocation with SigV4 works, confirming Lambda code is functional. Additionally, exposing Lambda Function URLs directly to the public internet is not desirable from a security standpoint.

**Decision:** Replace direct Lambda Function URL calls from the browser with an **API Gateway HTTP API** in eu-central-1, using a **Cognito JWT Authorizer** on protected routes.

**Architecture:**
```
Browser → API Gateway HTTP API → Lambda (IAM-only invocation)
               │
               ├── Public routes (pre-auth): /register, /confirm, /resend-code, /upload
               └── Protected routes (JWT): /chat, /chat-history, /get-documents, /onboard
```

**Route Configuration:**
| Route | Lambda | Auth | Rationale |
|-------|--------|------|-----------|
| POST /register | eu_register | NONE | Users register before having a token |
| POST /confirm | eu_confirm_signup | NONE | Email confirmation is pre-login |
| POST /resend-code | eu_resend_code | NONE | Resend code is pre-login |
| POST /upload | eu_upload_deep_document | NONE | Guest upload flow (pre-auth) |
| POST /chat | eu_chat_athena_bot | JWT | Requires authenticated user |
| POST /chat-history | eu_get_chat_history | JWT | Requires authenticated user |
| POST /get-documents | eu_get_deep_documents | JWT | Requires authenticated user |
| POST /onboard | eu_onboard_user | JWT | Requires authenticated user |

**JWT Authorizer:**
- Type: Cognito User Pool
- Issuer: `https://cognito-idp.eu-central-1.amazonaws.com/eu-central-1_Z0rehiDtA`
- Audience: `7d4487490ur1tpai0fuh4qle0b`
- Token source: `$request.header.Authorization`

**Lambda Function URLs:**
- ALL 17 EU Lambdas keep AuthType: **AWS_IAM** (never publicly accessible)
- API Gateway invokes Lambdas via IAM role, not Function URLs
- Internal Lambda-to-Lambda calls continue to use SDK invoke()

**Frontend Changes:**
- `region-config.ts`: EU endpoints point to API Gateway base URL + route paths
- `useEUSendMessage.ts`: Add Authorization header (Cognito ID token) to chat requests
- `useEUChatHistoryQuery.ts`: Add Authorization header to history requests

**Consequences:**
- (+) Lambda functions are never publicly exposed
- (+) Cognito JWT validation happens at the API Gateway level (before Lambda executes)
- (+) Built-in rate limiting and throttling via API Gateway
- (+) Proper CORS handling at API Gateway level
- (+) Single base URL for all EU API endpoints (clean architecture)
- (+) CloudWatch metrics and access logging at the API level
- (-) Additional infrastructure component (API Gateway)
- (-) Small additional latency (~10-30ms per request)
- (-) API Gateway cost ($1 per million requests — negligible)
