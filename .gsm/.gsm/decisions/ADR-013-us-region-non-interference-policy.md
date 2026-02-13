# ADR-013: US Region Non-Interference Policy

**Date:** 2026-02-11
**Status:** ACCEPTED (HARD REQUIREMENT)
**Context:** Multi-region architecture with separate US and EU implementations

---

## Problem Statement

CrawlQ operates two parallel AI chat implementations:
1. **US Region** (us-east-1) — Existing production system with 21 TRACE UI components
2. **EU Region** (eu-central-1) — New GDPR-compliant system with 17 Lambda functions

During EU development, there is a risk of accidentally modifying US code when:
- Porting TRACE UI components from US to EU
- Aligning architectures between regions
- Fixing bugs discovered during gap analysis
- Refactoring shared utilities

**User Requirement:**
> "You should not change anything in the US work and keep only checking it to align and fix the gap and continue to evolve the EU version directly. You must not change anything in US this is must follow rule."

This is a **HARD REQUIREMENT** — any violation would break production and disrupt ongoing US operations.

---

## Decision

### MANDATORY POLICY: US Region is READ-ONLY

**All US region code, configuration, and infrastructure is strictly READ-ONLY during EU development.**

### Permitted US Interactions
✅ **ALLOWED:**
- **READ** US code for reference (component structure, API patterns, utilities)
- **ANALYZE** US implementation to understand architecture
- **COMPARE** US vs EU to identify gaps
- **DOCUMENT** findings in gap analysis reports
- **COPY** US code patterns into NEW EU files (never modify originals)

❌ **PROHIBITED:**
- **MODIFY** any US source code files
- **DELETE** US files or components
- **REFACTOR** shared utilities that US depends on
- **FIX** bugs in US code (document them instead)
- **UPDATE** US Lambda functions, configurations, or infrastructure
- **MERGE** changes into US git branches

### File Path Rules

**US Region Files (READ-ONLY):**
```
crawlq-ui/src/app/(protected)/chat-athena/          ← US chat page
crawlq-ui/src/components/chat/                      ← US chat components (21 TRACE components)
crawlq-ui/src/store/useChatStore.ts                 ← US store
crawlq-ui/src/queries/chat/                         ← US React Query hooks
crawlq-ui/src/config/aws-exports-us.ts              ← US AWS config
crawlq-lambda/USLambdas/                            ← US Lambda functions (10 functions)
```

**EU Region Files (WRITABLE):**
```
crawlq-ui/src/app/(protected)/chat-athena-eu/       ← EU chat page
crawlq-ui/src/components/chat-eu/                   ← EU chat components (port here)
crawlq-ui/src/store/chat-eu/useChatEUStore.ts       ← EU store
crawlq-ui/src/queries/chat-eu/                      ← EU React Query hooks
crawlq-ui/src/config/region-config.ts               ← EU config (separate)
crawlq-lambda/SemanticGraphEU/                      ← EU Lambda functions (17 functions)
```

**Shared Files (CAUTION):**
```
crawlq-ui/src/types/                                ← Type definitions (safe to add, not modify)
crawlq-ui/src/utils/                                ← Utilities (create EU-specific versions)
crawlq-ui/src/hooks/                                ← Hooks (create EU-specific versions if needed)
```

### Porting Strategy

When porting US TRACE UI components to EU:

**Step 1: COPY (don't modify original)**
```bash
# ❌ WRONG: Modify US file
Edit crawlq-ui/src/components/chat/ChatSidebar.tsx

# ✅ CORRECT: Copy to EU, then modify
Copy crawlq-ui/src/components/chat/ChatSidebar.tsx
  → crawlq-ui/src/components/chat-eu/ChatSidebar.tsx
Edit crawlq-ui/src/components/chat-eu/ChatSidebar.tsx (EU version only)
```

**Step 2: ADAPT imports and dependencies**
```typescript
// US version (read-only, don't touch)
import { useChatStore } from '@/store/useChatStore';
import { fetchChatHistory } from '@/queries/chat/useChatHistoryQuery';

// EU version (new file, safe to modify)
import { useChatEUStore } from '@/store/chat-eu/useChatEUStore';
import { useEUChatHistoryQuery } from '@/queries/chat-eu/useEUChatHistoryQuery';
```

**Step 3: RENAME EU-specific components**
```typescript
// US (don't rename)
export function ChatSidebar() { ... }

// EU (add EU suffix to avoid confusion)
export function ChatSidebarEU() { ... }
// OR keep same name if in chat-eu/ folder (isolation by folder)
```

### Gap Analysis Protocol

When conducting US vs EU gap analysis:

1. **READ** US component to understand features
2. **DOCUMENT** findings in `.gcc/branches/feature-eu-chat-athena/gap-analysis.md`
3. **CREATE** corresponding EU component in `chat-eu/` folder
4. **TEST** EU component independently
5. **NEVER** modify US component

### Branch Strategy

**US Branches (DO NOT MERGE INTO):**
- `main` (may have US changes)
- `feature/chat-athena` (US-specific features)
- `feature/trace-us-*` (US TRACE work)

**EU Branches (SAFE TO MERGE):**
- `feature/trace-eu-frontend` (EU Amplify branch)
- `feature/trace-eu-enterprise` (EU Lambda branch)
- `feature-eu-chat-athena` (EU main feature branch, this branch)
- `feature-tier3-async-markdown` (EU Tier 3 architecture, ADR-012, separate work)

### Conflict Resolution with ADR-012

**ADR-012 Branch:** `feature-tier3-async-markdown`
- **Parent:** `feature-eu-chat-athena` (this branch)
- **Purpose:** Tier 3 async job queue + intelligent markdown processor
- **Status:** WORKING (in progress by separate developer/session)

**Coordination Rules:**
1. This branch (`feature-eu-chat-athena`) focuses on **Phase 1: TRACE UI porting**
2. ADR-012 branch focuses on **Tier 3 backend architecture + markdown rendering**
3. **No code conflicts expected** (different files: UI vs backend)
4. **Merge strategy:** ADR-012 merges back into `feature-eu-chat-athena` when complete
5. **Communication:** Document any shared file changes in commit logs

**Potential Overlap:**
- `ChatMarkdownRenderer.tsx` — This branch has basic version, ADR-012 has enterprise version
- **Resolution:** ADR-012 will replace basic renderer with enterprise version on merge
- This branch should NOT modify markdown renderer (let ADR-012 own it)

---

## Rationale

### Why This Policy Exists

1. **Production Stability:** US system is in production, serving real users
2. **Parallel Development:** Two teams/sessions may work simultaneously on US and EU
3. **Risk Mitigation:** Accidental US changes could break production
4. **Clean Separation:** EU should be fully isolated (different region, GDPR, compliance)
5. **Merge Safety:** When ADR-012 merges back, no US conflicts should exist

### Why US Must Remain Untouched

- **US has working production system** → Any change is high risk
- **EU is new greenfield implementation** → Safe to iterate
- **Gap analysis purpose** is to understand differences, NOT to fix US
- **US may have different requirements** (no GDPR, different compliance)
- **EU has superior compliance infrastructure** but lacks UI → port UI from US

### Benefits of READ-ONLY US Policy

✅ **Zero risk** of breaking US production
✅ **Clear ownership** (US team owns US, EU team owns EU)
✅ **Parallel development** without conflicts
✅ **Easy rollback** (EU can always revert without affecting US)
✅ **Clean git history** (no accidental US changes in EU branches)

---

## Implementation Checklist

### Before Every Edit
- [ ] Check file path — is it in `chat-eu/` or `SemanticGraphEU/`?
- [ ] If in US folder, create EU equivalent instead
- [ ] Never modify files in `chat/` or `USLambdas/`

### During Development
- [ ] Use `chat-eu/` prefix for all EU components
- [ ] Import from `@/store/chat-eu/` not `@/store/`
- [ ] Use EU-specific hooks and utilities
- [ ] Test EU implementation independently

### Before Commit
- [ ] `git status` — confirm no US files modified
- [ ] `git diff` — review changes, ensure all in EU folders
- [ ] Commit message — mention "EU only" if porting from US

### During Code Review
- [ ] Reviewer checks: no US file paths in diff
- [ ] Reviewer verifies: new EU files, not modified US files

---

## Consequences

### Positive
- **Production safety:** US system remains stable
- **Clean architecture:** EU fully isolated from US
- **Parallel workflows:** Multiple developers/sessions can work simultaneously
- **Easy merging:** ADR-012 branch merges cleanly into EU branch
- **Clear ownership:** Each region has distinct codebase

### Negative
- **Code duplication:** Same component exists in both `chat/` and `chat-eu/`
- **Maintenance burden:** Bug fixes must be applied to both regions separately
- **Divergence risk:** US and EU implementations may drift over time

### Mitigation for Negatives
- **Short-term:** Acceptable during initial EU buildout (Phase 1-3, 12 weeks)
- **Long-term:** Create shared component library (Phase 4, Week 13+)
- **Documentation:** Gap analysis tracks divergence
- **Refactor later:** Once EU reaches parity, extract shared utilities into `@/shared/`

---

## Monitoring and Enforcement

### Git Pre-Commit Hook (Recommended)
```bash
#!/bin/bash
# .git/hooks/pre-commit

# Check for US file modifications in EU branches
if git branch --show-current | grep -E 'feature-eu|feature-tier3'; then
    US_FILES=$(git diff --cached --name-only | grep -E '/(chat|USLambdas)/')
    if [ -n "$US_FILES" ]; then
        echo "❌ ERROR: EU branch cannot modify US files"
        echo "Modified US files:"
        echo "$US_FILES"
        echo ""
        echo "Policy: ADR-013 (US Region Non-Interference)"
        exit 1
    fi
fi
```

### Manual Verification
```bash
# Before commit, check for US file changes
git diff --name-only | grep -E '/(chat|USLambdas)/'

# Should return empty (no US files modified)
```

### CI/CD Check (GitHub Actions)
```yaml
# .github/workflows/enforce-adr-013.yml
name: Enforce ADR-013 (US Non-Interference)
on: [pull_request]
jobs:
  check-us-files:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0
      - name: Check for US file modifications
        run: |
          if [[ "${{ github.head_ref }}" == feature-eu-* ]] || [[ "${{ github.head_ref }}" == feature-tier3-* ]]; then
            US_FILES=$(git diff --name-only origin/${{ github.base_ref }}...${{ github.sha }} | grep -E '/(chat|USLambdas)/' || true)
            if [ -n "$US_FILES" ]; then
              echo "::error::ADR-013 violation: EU branch modified US files"
              exit 1
            fi
          fi
```

---

## Related ADRs

- **ADR-005:** EU Region Isolation Strategy (architectural separation)
- **ADR-012:** Tier 3 Async Architecture (separate EU branch work, no US interaction)
- **Gap Analysis:** `.gcc/branches/feature-eu-chat-athena/gap-analysis.md` (US vs EU comparison)

---

## Review and Updates

- **Review Frequency:** Every merge from `feature-tier3-async-markdown` back to `feature-eu-chat-athena`
- **Update Trigger:** If shared utilities are created in Phase 4
- **Sunset Date:** When unified shared component library is created (estimated Week 13+)

---

## Approval

**Decision Maker:** User (Haris)
**Enforcement:** Claude Code Agent (this session and all future EU sessions)
**Policy Type:** HARD REQUIREMENT (non-negotiable)

**Violation Response:**
- Session ends with error
- Changes rolled back
- ADR-013 re-read before continuing

---

**Status:** ACCEPTED
**Enforcement Date:** 2026-02-11
**Last Updated:** 2026-02-11
