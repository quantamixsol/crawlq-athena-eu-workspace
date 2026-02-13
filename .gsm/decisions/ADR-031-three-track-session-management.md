# ADR-031: Three-Track Session Management Protocol

**Date:** 2026-02-13 | **Status:** ACCEPTED

---

## Context

The CrawlQ EU Athena project has grown into 3 parallel workstreams running simultaneously:

1. **Athena Main** — The core EU Chat Athena standalone app (frontend + backend + deployment)
2. **TRACE Canvas** — Workflow builder with React Flow, execution engine, DynamoDB persistence
3. **FrictionMelt Integration** — Event emission, 91-pattern taxonomy, insights dashboard

Multiple Claude Code sessions work on these tracks concurrently. Without clear rules, sessions bleed across tracks — work gets lost, context gets confused, and the user (Haris) can't tell what's happening where. This ADR establishes the management protocol.

## Decision

### 1. Track Definitions

| Track | GCC Branch | Session Name | Owner | Status |
|-------|-----------|-------------|-------|--------|
| **Athena Main** | `feature-eu-standalone-app` | "Athena Main" | Any session | DEPLOYED, polish phase |
| **TRACE Canvas** | `feature-trace-canvas` | "Canvas" | Any session | Sprint 2 active |
| **FrictionMelt** | `feature-frictionmelt-integration` | "FrictionMelt" | Any session | BLOCKED until Feb 19 |
| **Master** | None (read-only) | "Master" | Coordination only | Always available |

### 2. Session Start Protocol (Enhanced)

Every Claude Code session MUST begin with:

```
Step 0: Declare track — "I'm working on [Athena Main / Canvas / FrictionMelt]"
Step 1: Read .gcc/main.md (global roadmap)
Step 2: Read .gcc/registry.md (all branches)
Step 3: Read ONLY the declared track's commit.md
Step 4: Announce: "Resuming COMMIT {N} on {branch}. Last: {X}. Next: {Y}"
Step 5: DO NOT read or modify other tracks' branches
```

**Master sessions** (this one) read ALL branches but write to NONE. They are for:
- Status dashboards
- Cross-track coordination
- Priority decisions
- ADR creation

### 3. One Session = One Track (Strict Isolation)

| Rule | Description |
|------|-------------|
| **No cross-track work** | A "Canvas" session MUST NOT touch FrictionMelt files or Athena Main files outside `crawlq-ui/src/components/canvas/*` and `crawlq-ui/src/app/(protected)/canvas/*` |
| **No cross-track commits** | A "Canvas" session commits ONLY to `feature-trace-canvas` branch |
| **Shared files need coordination** | If Canvas needs to modify `ChatSidebar.tsx` (shared), do it via a sub-branch or coordinate in a Master session |
| **Exceptions documented** | If a session MUST touch another track, log it in the commit with `CROSS-TRACK: {reason}` |

### 4. Shared File Registry

These files are touched by multiple tracks. Changes require extra care:

| File | Tracks | Rule |
|------|--------|------|
| `ChatSidebar.tsx` | Athena Main + FrictionMelt | Additive only — append nav links, never restructure |
| `package.json` | All 3 | Additive only — add deps, never remove |
| `.env.local` | All 3 | Additive only — add env vars with track prefix |
| `middleware.ts` | Athena Main + Canvas | Canvas uses dev bypass; Athena Main owns prod config |
| `tailwind.config.ts` | Athena Main + Canvas | Extend only, never override existing theme tokens |
| `globals.css` | Athena Main + Canvas | Append sections with `/* TRACK: Canvas */` comments |

### 5. Merge Order

Tracks merge into `feature-eu-standalone-app` in this order:

```
1. feature-trace-canvas         → feature-eu-standalone-app  (when Sprint 2 verified)
2. feature-frictionmelt-integration → feature-eu-standalone-app  (after Feb 19 + E2E pass)
3. feature-eu-standalone-app    → feature-eu-chat-athena      (release candidate)
4. feature-eu-chat-athena       → main                        (production release)
```

**Merge prerequisites:**
- All tests pass on source branch
- No BLOCKED commits
- Shared files reviewed for conflicts
- GCC COMMIT with state: DONE on source branch

### 6. Priority Matrix

| Priority | Track | Why | Action |
|----------|-------|-----|--------|
| **P1** | Canvas Sprint 2 | Active development, unblocked, closest to user value | Work on this NOW |
| **P2** | Athena Main polish | Deployed but needs domain + visual audit for launch | Work when Canvas is blocked or between sprints |
| **P3** | FrictionMelt | Hard-blocked on external team (Feb 19 ETA) | No action until API is live |

### 7. Daily Check-In Template

At the start of each day, run a Master session with this checklist:

```markdown
## Daily Check-In — {YYYY-MM-DD}

### Athena Main
- Last commit: {N} — {summary}
- Blockers: {any}
- Today's priority: {what to do}

### Canvas
- Last commit: {N} — {summary}
- Sprint 2 progress: {X/Y items done}
- Blockers: {any}
- Today's priority: {what to do}

### FrictionMelt
- Last commit: {N} — {summary}
- External blocker: {status — still waiting / resolved}
- Today's priority: {what to do}

### Decision needed: {any cross-track decisions}
```

### 8. When to Open a Master Session

- Start of day (daily check-in)
- When confused about overall status
- When a track needs to touch shared files
- When priorities need to change
- When an external blocker resolves (e.g., FrictionMelt API goes live)
- Weekly planning

## Consequences

**Positive:**
- Clear ownership: every session knows exactly which branch to read/write
- No accidental cross-track contamination
- User (Haris) can ask "what's happening on Canvas?" and get a clean answer
- Merge conflicts minimized by shared file registry
- Priority is explicit — no wasted cycles on blocked tracks

**Negative:**
- Slight overhead: sessions must declare track at start
- Shared file changes require coordination (but this prevents bugs)
- Master sessions are read-only (can't do quick fixes across tracks)

**Mitigations:**
- Track declaration is 1 line, trivial overhead
- Shared file registry is small (6 files) and stable
- For urgent cross-track fixes, use `CROSS-TRACK:` annotation in commits
