---
title: Unused Dependencies
impact: MEDIUM
impactDescription: "Inflate install size, supply-chain surface, and audit noise"
tags: dependencies, unused, cleanup
---

## Unused Dependencies

**Impact: MEDIUM (Inflate install size, supply-chain surface, and audit noise)**

A dependency you don't use is one you still ship, audit, and trust. Each unused dep is a potential supply-chain footgun (compromised maintainer, malicious post-install script) for zero benefit.

## How to Detect

```bash
# Node / TypeScript
npx depcheck                         # unused + missing deps
npx knip                             # also finds unused files and exports

# PHP / Composer
composer-unused                      # https://github.com/composer-unused/composer-unused
vendor/bin/composer-unused
```

## Incorrect

```json
// ❌ package.json declares deps no longer imported
{
  "dependencies": {
    "lodash": "^4.17.21",        // grep shows zero `from 'lodash'` imports
    "axios": "^1.6.0",           // migrated to fetch 6 months ago
    "moment": "^2.29.4",         // migrated to date-fns; one stale import left
    "node-fetch": "^3.3.0"       // only used in a deleted script
  }
}
```

**Problems:**
- Each `npm install` downloads code that does nothing
- Each `npm audit` reports advisories you can't act on (you don't even use the affected code paths)
- New engineers see them and assume they're load-bearing

## Correct

```bash
# ✅ Remove unused deps
$ npx depcheck
Unused dependencies: lodash, axios, moment, node-fetch
$ npm uninstall lodash axios moment node-fetch
$ npm audit            # quieter report
```

```yaml
# Add to CI to keep it clean
- run: npx depcheck --ignores="@types/*,eslint-*"
```

**Benefits:**
- Smaller `node_modules`, faster installs, faster CI
- Audit reports are signal, not noise
- Reduced supply-chain attack surface

## Remediation Strategy

- **Effort:** S (almost always)
- **When to pay down:** Immediately on detection. Add a depcheck/composer-unused step to CI to prevent regression.

**Watch out for:**
- **Transitive usage only:** some deps are loaded by tooling (e.g., babel plugins listed in `babel.config.js`). Verify before removing.
- **Type-only packages:** `@types/*` packages are used by the compiler but invisible to import scanners — configure your tool to ignore them.

Reference: [depcheck](https://github.com/depcheck/depcheck) · [composer-unused](https://github.com/composer-unused/composer-unused)
