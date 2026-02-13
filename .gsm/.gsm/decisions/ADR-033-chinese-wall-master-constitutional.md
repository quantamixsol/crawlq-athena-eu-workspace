# ADR-033: Chinese Wall — Master Constitutional Decision

**Date:** 2026-02-13 | **Status:** MASTER CONSTITUTIONAL (supreme authority, cannot be overridden)

## Context

ADR-032 established repo isolation. This ADR elevates it to a **Chinese Wall** — a term from finance meaning an absolute information barrier that cannot be crossed under any circumstances. Previous sessions violated repo boundaries despite ADR-017, causing code drift and contamination. This must never happen again.

## Decision

### THE CHINESE WALL

There are two worlds. They do not touch.

**WORLD A — US CrawlQ (Reference Only)**
| Repo | Role | Access Level |
|------|------|-------------|
| `crawlq-ui` | US frontend product | **READ-ONLY** — learn from it, never modify |
| `crawlq-lambda` | US backend product | **READ-ONLY** — learn from it, never modify |

**WORLD B — Athena EU (Active Development)**
| Repo | Role | Access Level |
|------|------|-------------|
| `crawlq-chat-athena-eu-frontend` | EU frontend (all features except Canvas) | **READ-WRITE** — active development |
| `crawlq-athena-eu-backend` | EU Lambdas (25+ functions) | **READ-WRITE** — active development |
| `crawlq-athena-eu-canvas` | TRACE Canvas app (separate track) | **READ-WRITE** — active development |
| `crawlq-athena-eu-workspace` | GCC/GSM decisions, ADRs, session context | **READ-WRITE** — context management |

### ABSOLUTE RULES

1. **NEVER write to World A repos.** Not a single line. Not a comment. Not a config change. Read and learn only.

2. **ALL new code goes to World B repos.** Every feature, fix, refactor, test, config, deployment — all in World B.

3. **Shared patterns are COPIED, not linked.** If you need a pattern from crawlq-ui, copy the concept to the EU repo. Do not create cross-repo imports or dependencies.

4. **No "temporary" exceptions.** There is no "just this once" for writing to crawlq-ui or crawlq-lambda. The wall is absolute.

5. **Session start verification is MANDATORY:**
   ```
   BEFORE ANY WORK:
   1. Which repo am I in?
   2. Is this repo in World B?
   3. If World A → STOP. Switch to the correct World B repo.
   ```

6. **Violation protocol:**
   - STOP immediately
   - Revert all changes to World A repos
   - Migrate code to correct World B repo
   - Document the violation in GCC commit log
   - This is not optional

### DEPLOYMENT MAPPING

| App | Amplify ID | Repo (Source of Truth) | Branch |
|-----|-----------|----------------------|--------|
| Athena EU Frontend | d45bl3mgpjnhy | crawlq-chat-athena-eu-frontend | main |
| TRACE Canvas | TBD | crawlq-athena-eu-canvas | main |
| EU Lambdas | N/A (direct deploy) | crawlq-athena-eu-backend | main |

### WHY THIS EXISTS

1. **crawlq-ui is a production US product.** EU development must not risk US stability.
2. **Independent deployment cycles.** EU can ship without touching US. US can ship without touching EU.
3. **Clean audit trail.** Each repo has its own git history. No mixed commits.
4. **Session continuity.** Next AI session knows exactly where to work.
5. **Previous violations cost hours of migration work.** Prevention is cheaper than correction.

## Consequences

- **Positive:** Absolute clarity on where code belongs. No drift. No contamination. Independent deployments. Clean git history.
- **Negative:** Must maintain parallel codebases. Shared utilities duplicated. Slightly more repo management overhead.
- **Accepted trade-off:** Duplication is cheaper than confusion. 4 clean repos > 1 messy monorepo.

## Supersedes

- ADR-017 (Standalone Repo Extraction)
- ADR-032 (Repo Isolation Enforcement)

This ADR is the **supreme authority** on repo boundaries. All other ADRs defer to it.
