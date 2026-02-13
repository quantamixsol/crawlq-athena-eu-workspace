# ADR-004: ZIP-Based Lambda Deployment from Windows
**Date:** 2026-02-09 | **Status:** ACCEPTED
**Context:** 17 EU Lambda functions need to be deployed from a Windows dev machine. Options: (1) Docker-based deploy via deploy.sh (requires Docker Desktop), (2) ZIP-based deploy via Python boto3 script with cross-platform pip.
**Decision:** Use ZIP-based deployment with `pip install --platform manylinux2014_x86_64 --only-binary=:all:` to get Linux-compatible wheels from Windows. Fallback to generic pip install for packages without prebuilt Linux wheels.
**Consequences:**
- (+) No Docker dependency on dev machine
- (+) Fast iteration (30s per Lambda vs 5min Docker build)
- (-) Some packages with C extensions (google-genai/pydantic_core) can't cross-compile — made optional
- (-) CI/CD pipeline (GitHub Actions) should still use Docker for production deploys
