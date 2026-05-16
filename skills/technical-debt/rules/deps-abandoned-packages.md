---
title: Abandoned and Unmaintained Packages
impact: HIGH
impactDescription: "No upstream fixes for bugs, CVEs, or runtime upgrades"
tags: dependencies, abandoned, maintenance
---

## Abandoned and Unmaintained Packages

**Impact: HIGH (No upstream fixes for bugs, CVEs, or runtime upgrades)**

A package without a release in 24+ months, with open critical issues, or with the maintainer publicly stepping away is **abandonware**. The next CVE, the next Node/PHP version, or the next breaking dep change becomes *your* problem to fix.

## How to Detect

Indicators to check on each direct dependency:

- **Last release date** (`npm view <pkg> time.modified`, `composer info <pkg>`)
- **Open issues vs closed** ratio (high open count, no recent triage)
- **Maintainer activity** (last commit > 2 years ago)
- **Explicit deprecation** (`npm view <pkg> deprecated`)
- **Known alternatives community has migrated to**

```bash
# Node
npm view <pkg> time.modified deprecated
npx npm-check                       # flags deprecated packages
npx snyk test                       # warns on unmaintained packages

# PHP
composer info <pkg>                 # shows abandoned status from packagist
composer audit                      # also reports abandonment
```

## Incorrect

```json
// ❌ Depending on packages flagged as abandoned or deprecated
{
  "dependencies": {
    "request": "^2.88.0",            // deprecated 2020 by maintainer
    "node-uuid": "^1.4.8",           // replaced by `uuid` years ago
    "moment": "^2.29.0",             // maintenance-only since 2020, deprecated by author
    "babel-eslint": "^10.1.0"        // replaced by @babel/eslint-parser
  }
}
```

**Problems:**
- `request` has an open CVE with no upstream fix coming
- `moment` ships ~290KB of timezone data — `date-fns` or `dayjs` do it in 10KB
- Future engineer cannot tell whether these are "trusted core deps" or graveyard residents

## Correct

```json
// ✅ Migrated to maintained alternatives
{
  "dependencies": {
    "undici": "^6.0.0",              // replaces `request`
    "uuid": "^9.0.0",                // replaces `node-uuid`
    "date-fns": "^3.0.0",            // replaces `moment`
    "@babel/eslint-parser": "^7.23.0"
  }
}
```

**Benefits:**
- CVEs in maintained packages get upstream fixes — you only patch
- Bundle size and runtime characteristics improve
- New engineers don't waste time on packages they "shouldn't have learned"

## Remediation Strategy

- **Effort:** S–M per package (depends on API surface used)
- **When to pay down:**
  1. **Now:** any abandoned dep with a known CVE
  2. **This quarter:** any abandoned dep blocking a runtime upgrade
  3. **Opportunistically:** the rest, when you're already touching that code path

**Tip:** When forced to keep an abandoned dep temporarily, lock the version exactly, document why in a comment in the manifest, and create a tracking issue.

Reference: [npm Deprecation Policy](https://docs.npmjs.com/policies/deprecation) · [Packagist Abandoned Packages](https://packagist.org/about#abandoning-a-package)
