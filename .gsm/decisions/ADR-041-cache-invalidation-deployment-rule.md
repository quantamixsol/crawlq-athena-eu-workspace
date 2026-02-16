# ADR-041: Cache Invalidation Deployment Rule

**Date:** 2026-02-15 | **Status:** ACCEPTED | **Priority:** CRITICAL

## Context

Recurring production issue: code changes are pushed to git, Amplify builds succeed, but users (and developers) see stale content because:

1. **CloudFront CDN cache** (Amplify uses CloudFront) serves cached HTML/JS for up to 24h by default
2. **Browser cache** serves locally cached pages without checking for updates
3. **No cache-control headers** were configured in next.config.mjs or amplify.yml
4. **No build ID versioning** — no way to verify which build is live
5. **.next/cache was cached between builds** in amplify.yml — could serve stale SSR content

This caused repeated "ghost deployments" where builds show GREEN but users see old UI.

## Decision

### Rule 1: Cache-Control Headers (MANDATORY)

Every Next.js deployment MUST have these cache-control headers:

| Path Pattern | Cache-Control | Reason |
|-------------|---------------|--------|
| `/_next/static/**` | `public, max-age=31536000, immutable` | Content-hashed, safe to cache forever |
| `/**/*.html`, `/` | `public, max-age=0, must-revalidate` | HTML pages must always check for new version |
| `/api/**` | `no-store, no-cache, must-revalidate` | API responses must never be cached |

### Rule 2: Unique Build ID (MANDATORY)

Every build MUST generate a unique `BUILD_ID` via `generateBuildId()` in next.config.mjs:
```javascript
generateBuildId: async () => `build-${Date.now()}-${crypto.randomBytes(4).toString("hex")}`
```
This ensures `/_next/static/{BUILD_ID}/` paths are unique per deployment, busting CloudFront cache for JS/CSS bundles.

### Rule 3: No SSR Cache Between Builds (MANDATORY)

The amplify.yml cache section MUST NOT include `.next/cache/**/*`:
```yaml
cache:
  paths:
    - node_modules/**/*    # OK — dependencies don't change often
    # NEVER cache .next/cache — causes stale SSR pages
```

### Rule 4: Post-Deploy Verification (MANDATORY)

After every Amplify build succeeds, verify the live site serves the new build:

```bash
# Check live BUILD_ID matches deployed BUILD_ID
curl -s https://main.d45bl3mgpjnhy.amplifyapp.com/_next/static/BUILD_ID 2>/dev/null
# Or check response headers for X-Build-Timestamp
curl -sI https://main.d45bl3mgpjnhy.amplifyapp.com/ | grep -i cache-control
```

### Rule 5: Force Invalidation (EMERGENCY)

If stale content persists after build succeeds:
```bash
# Amplify automatically invalidates CloudFront on deploy
# If it doesn't, use AWS CLI:
aws cloudfront create-invalidation \
  --distribution-id <DIST_ID> \
  --paths "/*" \
  --region eu-central-1
```

### Rule 6: CloudFront Error Pattern Detection (CRITICAL)

**Known failure pattern (recurring):**
```
x-cache: Error from cloudfront          ← CloudFront is erroring
Last-Modified: <OLD timestamp>          ← Stale cached version
Content-Type: text/html                 ← Wrong MIME (should be application/javascript)
```

**Root cause:** Amplify's CloudFront distribution caches old deployment artifacts. When new build deploys, CloudFront sometimes:
1. Returns stale cached HTML/JS with wrong MIME types
2. Shows `x-cache: Error from cloudfront` — indicating CDN-level error
3. Serves `Last-Modified` timestamps from BEFORE the new deployment

**Detection:** After every deploy, check response headers:
```bash
curl -sI https://main.d45bl3mgpjnhy.amplifyapp.com/ | grep -E "x-cache|last-modified|content-type|cache-control"
```

**If x-cache shows Error:** Force CloudFront invalidation immediately via Amplify Console > App settings > General > Manage Production branch > Redeploy this version.

### Rule 7: Browser Hard Refresh (USER SIDE)

Document for users: after deployment, if old UI persists:
- **Chrome/Edge:** Ctrl+Shift+R (hard refresh)
- **Firefox:** Ctrl+F5
- **Safari:** Cmd+Option+R

## Enforcement

### GCC Deployment Checklist (EVERY DEPLOY)

```
PRE-DEPLOY:
  [ ] next.config.mjs has generateBuildId()
  [ ] next.config.mjs has headers() with cache-control
  [ ] amplify.yml does NOT cache .next/cache
  [ ] amplify.yml has customHeaders section

POST-DEPLOY:
  [ ] Amplify build shows SUCCEED
  [ ] curl live URL — verify Cache-Control: public, max-age=0, must-revalidate
  [ ] Load site in incognito — verify new changes visible
  [ ] If stale: force CloudFront invalidation
```

### Pre-Commit Hook (RECOMMENDED)

Add to `.husky/pre-commit` or CI:
```bash
# Verify cache-busting config exists
grep -q "generateBuildId" next.config.mjs || (echo "ERROR: Missing generateBuildId in next.config.mjs (ADR-041)" && exit 1)
grep -q "must-revalidate" next.config.mjs || (echo "ERROR: Missing cache-control headers in next.config.mjs (ADR-041)" && exit 1)
```

## Consequences

- (+) Every deployment immediately visible to all users
- (+) Static assets still cached efficiently (content-hashed, immutable)
- (+) Build ID enables verification that correct version is live
- (+) No more "ghost deployments"
- (-) HTML pages always revalidated — slightly slower first load (but correct content)
- (-) Amplify postBuild step adds ~5s to build pipeline

## Files Modified

- `crawlq-chat-athena-eu-frontend/next.config.mjs` — Added generateBuildId + headers()
- `crawlq-chat-athena-eu-frontend/amplify.yml` — Added customHeaders, removed .next/cache from cache, added postBuild verification
