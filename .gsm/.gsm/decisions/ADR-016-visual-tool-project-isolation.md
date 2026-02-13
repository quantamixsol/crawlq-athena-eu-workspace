# ADR-016: Visual UI Test Tool — Project Isolation (CrawlQ vs FrictionMelt)

**Date:** 2026-02-11 | **Status:** ACCEPTED (HARD REQUIREMENT)

## Context

Both CrawlQ Athena EU and FrictionMelt projects use Playwright + Claude Vision visual audit tools for UI testing. The tools share a similar architecture (screenshot capture → AI analysis → report generation) but target completely different products with different branding, design systems, pages, and compliance requirements.

There is a risk of cross-contamination: using FrictionMelt brand rules when auditing CrawlQ UI, or vice versa. This would produce incorrect audit findings and potentially introduce wrong styling fixes.

## Decision

**The visual UI test tools for CrawlQ and FrictionMelt MUST be completely isolated.** Specifically:

### HARD RULES (Never Violate)

1. **Separate scripts:** CrawlQ uses `crawlq-ui/scripts/visual-audit-eu.mjs`. FrictionMelt uses its own `scripts/visual-audit.mjs`. These are DIFFERENT files in DIFFERENT repositories.

2. **Separate brand rules:** CrawlQ uses the TRACE EU Design System (glassmorphism, 5-tier confidence colors, EU compliance badges). FrictionMelt uses its own brand rules (#0E1935 headlines, #5485FE buttons, etc.). NEVER apply FrictionMelt brand rules to CrawlQ or vice versa.

3. **Separate AI backend:** CrawlQ uses **AWS Bedrock** (`eu.anthropic.claude-opus-4-6-v1` in eu-central-1). FrictionMelt uses direct Anthropic API. Different SDKs, different authentication, different regions.

4. **Separate page configurations:** CrawlQ audits `/guest-eu`, `/login`, `/sign-up`, `/chat-athena-eu`. FrictionMelt audits `/`, `/product`, `/pricing`. These page lists MUST NOT be mixed.

5. **Separate reports:** CrawlQ generates `VISUAL_AUDIT_REPORT_EU.md`. FrictionMelt generates `VISUAL_AUDIT_REPORT.md`. Different filenames, different output directories.

6. **No shared code:** The visual audit scripts do NOT import from each other. No shared utility modules. Each is self-contained.

### WHY This Matters

| Aspect | CrawlQ Athena EU | FrictionMelt |
|--------|-----------------|--------------|
| Design System | Glassmorphism 2.0, 5-tier confidence | Enterprise B2B, flat design |
| Typography | -apple-system, 9px-30px micro-typography | Inter/system, clamp() responsive |
| Colors | Emerald/Blue/Amber/Red/Rose tiers | Navy #0E1935 / Blue #5485FE |
| Compliance | EU AI Act Art. 13/14/50, GDPR | None |
| AI Backend | AWS Bedrock eu-central-1 | Direct Anthropic API |
| Auth | Cognito JWT (protected pages) | None (static site) |
| Dark Mode | Yes (full support) | No |
| Target | Enterprise compliance SaaS | Marketing/product site |

## Consequences

**Positive:**
- Zero risk of brand contamination between projects
- Each tool is optimized for its specific project's needs
- CrawlQ tool includes EU compliance checks that FrictionMelt doesn't need
- FrictionMelt tool includes SEO/marketing checks that CrawlQ doesn't need

**Negative:**
- Some code duplication (both use Playwright, both generate reports)
- Maintenance of two separate tools
- Accepted trade-off: isolation > DRY for cross-project testing tools

## Enforcement

- This ADR is logged as a HARD REQUIREMENT in GCC
- Any Claude Code instance working on either project MUST check this ADR before modifying visual audit tools
- The GCC `gcc-visual-audit` skill is configured per-project with project-specific parameters
