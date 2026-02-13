# ADR-003: Chat Memory Default to Opt-In (false)
**Date:** 2026-02-09 | **Status:** ACCEPTED
**Context:** EU Chat Athena stores conversation memory (summaries, topics) in DynamoDB. GDPR Article 25 requires "data protection by design and by default" — meaning the least privacy-invasive option must be the default.
**Decision:** Changed `memoryEnabled` default from `true` to `false` in `useChatEUStore.ts`. Users must explicitly opt in via the ChatMemoryIndicator toggle or the consent banner.
**Consequences:**
- (+) GDPR Article 25 compliant — privacy by default
- (+) Users explicitly consent before memory is stored
- (-) New users don't get memory features until they opt in
- The consent banner (`ChatConsentBanner`) appears for new users to guide them
