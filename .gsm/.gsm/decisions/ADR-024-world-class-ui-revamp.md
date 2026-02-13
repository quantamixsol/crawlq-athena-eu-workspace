# ADR-024: World-Class UI Revamp — "Trust by Design"
**Date:** 2026-02-12 | **Status:** ACCEPTED

**Context:** Athena EU backend is production-ready (23 Lambdas, 7 DynamoDB tables, full TRACE protocol, KG exploration, governance runtime — all smoke-tested). The frontend is ~70% complete with strong foundations but missing the productivity layer that makes a B2B enterprise product feel world-class. Enterprise compliance officers need document exports, artifact panels, keyboard-first workflows, and self-guided UX.

**Decision:** Implement a 6-sprint UI revamp under the design philosophy "Trust by Design" — every pixel communicates trustworthiness, auditability, and intelligence. The UI is self-guided — users discover features through context, not manuals.

**Sprints:**
1. **Export Intelligence** — PDF/DOCX/MD export with branded templates, TRACE compliance summary, governance metadata
2. **Artifact Panel** — Claude-style resizable right panel for long responses, auto-generated TOC, section-level navigation
3. **Command Palette + Keyboard Shortcuts** — Ctrl+K fuzzy search, keyboard-first workflows (Ctrl+N, Ctrl+E, ?, etc.)
4. **Smart Response Actions** — Context-aware follow-up suggestion chips after each response, keyword extraction from compliance topics
5. **Profile Header + Theme Toggle** — User presence, theme switching (light/dark/system), logout, enterprise account display
6. **Conversation Intelligence** — Workspace search, response feedback (thumbs up/down + comment), quality tracking

**New Dependencies:** `docx` (DOCX generation), `file-saver` (blob downloads)

**Files Created:** 9 new components + 1 utility
**Files Modified:** ChatMessageBubble.tsx, ChatSidebar.tsx, page.tsx, ChatInput.tsx, ChatContainer.tsx

**Consequences:**
- **Positive:** Enterprise-grade UX, self-guided discovery, keyboard-first power-user workflows, exportable compliance documents, competitive with Claude/Notion/Linear
- **Positive:** Each sprint independently deployable — incremental value delivery
- **Negative:** 9 new files increase bundle size (mitigated: all components use dynamic imports and lazy loading)
- **Negative:** `docx` dependency adds ~150KB gzipped to client bundle (mitigated: only loaded on export action)
