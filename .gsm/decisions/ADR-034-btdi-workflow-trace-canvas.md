# ADR-034: Build-Test-Deploy-Iterate (BTDI) Standardized Workflow — TRACE Canvas

**Date:** 2026-02-14 | **Status:** ACCEPTED (LOCKED — Mandatory for all Canvas development)

## Context

The TRACE Canvas project has reached Build 14 on AWS Amplify with 14 components, 6 test files, and ~40% test coverage. Development has been productive but ad-hoc: builds were sometimes pushed without running the full test suite, user verification was informal, and GCC commits sometimes lagged. We need a standardized, repeatable workflow that any developer or Claude Code instance can follow to guarantee quality at every step.

## Decision

Every code change follows this mandatory 6-phase cycle. No phase may be skipped.

```
PREFLIGHT → CODE → BUILD → TEST → DEPLOY → USER VERIFY → GCC CHECKPOINT
```

---

## Phase 0: PREFLIGHT

| # | Check | Pass Criteria |
|---|-------|---------------|
| P1 | Correct repo (`crawlq-athena-eu-canvas/`) | Path verified |
| P2 | GCC context loaded | Know current COMMIT N, last milestone, next steps |
| P3 | Clean working tree (`git status`) | No unexpected unstaged changes |
| P4 | Last Amplify build green | Status: SUCCEED |
| P5 | Baseline tests pass (`npm test -- --bail`) | All existing tests pass |

## Phase 1: CODE

- One concern per cycle (never mix unrelated changes)
- ADR-028 compliance (isolated routes/components)
- Track all file changes (CREATED/MODIFIED/DELETED)
- GCC COMMIT every 30 minutes (state: WORKING)

## Phase 2: BUILD

- `npx next build` — exit code 0, 0 errors
- All 13+ pages compiled
- No new warnings introduced

## Phase 3: TEST

Execute in order:
1. **Unit tests:** `npm test -- __tests__/canvas/workflow __tests__/canvas/lib`
2. **Integration tests:** `npm test -- __tests__/canvas/integration`
3. **Regression tests:** `npm test -- --bail` (all tests)
4. **Coverage:** `npm run test:canvas:coverage` — target >= 80%

ALL must pass. New components MUST have corresponding test files.

## Phase 4: DEPLOY

1. `git add {specific files}` (never `git add .`)
2. `git diff --staged` — confirm only intended changes
3. `git commit -m "{descriptive message}"`
4. `git push origin main`
5. Wait for Amplify Build SUCCEED

## Phase 5: USER VERIFY

Core checklist on live app (`https://main.d1tnt2fg41rrrv.amplifyapp.com`):

| # | Test | Expected |
|---|------|----------|
| UV1 | Navigate to `/canvas` logged out | Redirects to `/login` |
| UV2 | Login | Canvas list loads |
| UV3 | Create workflow (`/canvas/new`) | Empty canvas with ToolLibrary |
| UV4 | Drag Input + LLM + Output | Nodes appear |
| UV5 | Connect nodes | Edges work |
| UV6 | Configure and Run | Results with TRACE score |
| UV7 | Save and reload | Canvas persists |

Plus feature-specific items for each sprint.

## Phase 6: GCC CHECKPOINT

Append to `.gcc/branches/feature-trace-canvas/commit.md`:

```
### COMMIT {N} — {timestamp}
**Milestone:** {description}
**State:** DONE
**BTDI:** Build PASS | Tests {X}/{X} PASS | Amplify Build {N} SUCCEED | User Verify {X}/{X} PASS
**Files Changed:** ...
**Next:** ...
```

## Consequences

- **Positive:** Every change verified across build, tests, deployment, and manual testing
- **Positive:** Any developer/Claude instance can follow without tribal knowledge
- **Positive:** Test coverage will increase from 40% toward 80%+
- **Negative:** ~15-20 min overhead per cycle (acceptable for production safety)
