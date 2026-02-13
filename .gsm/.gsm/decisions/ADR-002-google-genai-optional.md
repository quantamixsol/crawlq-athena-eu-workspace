# ADR-002: google-genai Made Optional Import
**Date:** 2026-02-09 | **Status:** ACCEPTED
**Context:** `google-genai` package has a complex transitive dependency tree (pydantic, pydantic_core with C extensions) that cannot be reliably cross-compiled from Windows for Lambda Linux x86_64. Installing it hangs or produces incompatible binaries.
**Decision:** Made `from google import genai` a conditional import with `try/except ImportError`. GeminiClient raises an exception when called without the library. Anthropic Claude is primary LLM, OpenAI is secondary fallback. Gemini is tertiary and optional.
**Consequences:**
- (+) eu_deep_graph_builder and eu_generate_deep_insights deploy and run successfully
- (+) Anthropic + OpenAI provide full LLM coverage
- (-) Gemini fallback unavailable until Docker-based deployment is set up
- Future: Docker-based CI/CD (deploy.sh) will install google-genai natively on Linux
