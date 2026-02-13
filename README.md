# Athena EU Workspace

Context management for the Athena EU project ecosystem using GCC (Global Context Controller) and GSM (Global Strategy Management).

## What This Repo Contains

- `.gcc/` — Global Context Controller (session history, branch commits, checkpoints)
- `.gsm/` — Global Strategy Management (architecture decisions, strategy summaries, external docs index)

## Repo Ecosystem

| Repo | Purpose | Access |
|------|---------|--------|
| **crawlq-chat-athena-eu-frontend** | Athena EU frontend app | Active development |
| **crawlq-athena-eu-backend** | Athena EU Lambda functions | Active development |
| **crawlq-athena-eu-canvas** | TRACE Canvas app | Active development |
| **crawlq-athena-eu-workspace** | This repo — decisions & context | Active development |
| crawlq-ui | US CrawlQ frontend | READ-ONLY (Chinese Wall — ADR-033) |
| crawlq-lambda | US CrawlQ backend | READ-ONLY (Chinese Wall — ADR-033) |

## Key ADRs

- **ADR-033**: Chinese Wall — Master Constitutional Decision (US repos = read-only)
- **ADR-032**: Repository Isolation Enforcement
- **ADR-031**: Master Priority Order
- **ADR-028**: Constitutional Non-Breaking (Canvas)

## How to Use

Every Claude Code session should:
1. Clone/pull this repo
2. Read `.gcc/main.md` (global roadmap)
3. Read `.gcc/registry.md` (active branches)
4. Read the active branch's `commit.md` (latest milestones)
5. Work in the appropriate feature repo (frontend/backend/canvas)
6. Commit context back to this workspace repo at session end

## GCC Protocol

See CLAUDE.md in parent directory or `.gcc/` files for the full GCC protocol documentation.
