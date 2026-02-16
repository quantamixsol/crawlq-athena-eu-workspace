### COMMIT 15 — 2026-02-14T01:45:00Z (Canvas Session Part 2)
**Milestone:** Canvas app LIVE on Amplify — Env vars fixed, IAM service role, build SUCCEED

**State:** DONE

**Key Results:**
- Amplify env vars fixed: NEXT_PUBLIC_REGION=eu, correct Cognito client ID, DynamoDB/LLM config
- IAM Role `AmplifyCanvasServiceRole` created with DynamoDB access to trace-canvas-documents
- Build 11 SUCCEED — app live at https://main.d1tnt2fg41rrrv.amplifyapp.com
- All routes verified: /login (200), /canvas (307→login), /sign-up (200)
- Cognito Pool ID correctly embedded in login page

**Next:** User tests login, verify DynamoDB operations on deployed app
