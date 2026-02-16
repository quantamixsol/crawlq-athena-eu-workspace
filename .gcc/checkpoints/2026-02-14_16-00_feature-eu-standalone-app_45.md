### COMMIT 45 — 2026-02-14T16:00:00Z
**Milestone:** COMMIT 9 complete — Subscription fix, KG source filtering, Audit Trail, Server-side query counter, Pallas 18/0/4
**State:** DONE
**Branch:** feature-eu-standalone-app
**Pallas Results:** 18 PASS / 0 FAIL / 4 WARN (regression, pallas-04)
**Build:** npx next build PASSED (chat-athena-eu: 669kB)
**ADR:** ADR-037 created — documents all 8 phases
**Files Changed:** 24 files (14 modified, 4 created, 6 test/config)
**Key Phases:**
- P1: Subscription data flow fix (type widening, EU normalization)
- P2: KG crash fix + ErrorBoundaryEU
- P3: Per-response KG + RAG sources panel
- P4: Mermaid 4th sanitization level
- P5: Search/TRACE mode indicator badges
- P6: KG source type filtering (doc/query/inferred)
- P7: Server-side query counter (hybrid localStorage + DynamoDB)
- P8: Audit Trail panel (Merkle chain verification, Business+ gated)
**Next:** Deploy to Amplify, re-run Pallas post-deploy
