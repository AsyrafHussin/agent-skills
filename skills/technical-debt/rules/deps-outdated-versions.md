---
title: Outdated Dependency Versions
impact: HIGH
impactDescription: "Each major version skipped exponentially raises upgrade cost"
tags: dependencies, upgrades, versioning
---

## Outdated Dependency Versions

**Impact: HIGH (Each major version skipped exponentially raises upgrade cost)**

Skipping major versions doesn't save effort — it just defers and compounds it. Two majors behind is roughly 4× the upgrade work of one major behind, because deprecations from intermediate versions stack.

## How to Detect

```bash
# Node / TypeScript
npm outdated                       # shows current vs wanted vs latest
npx npm-check-updates              # interactive upgrade tool

# PHP / Laravel
composer outdated --direct         # direct deps only
composer outdated --direct --major-only
```

Threshold: any dependency **more than 2 major versions behind** OR **more than 18 months behind on minor/patch**.

## Incorrect

```json
// ❌ package.json — multiple dependencies 3+ majors behind
{
  "dependencies": {
    "react": "^16.8.0",           // current major: 19
    "express": "^4.17.0",         // current major: 5
    "webpack": "^4.46.0",         // current major: 5
    "jest": "^26.6.3",            // current major: 29
    "@types/node": "^14.0.0"      // current: 22
  }
}
```

**Problems:**
- React 16 → 19 means a full upgrade path through legacy mode, automatic batching, new JSX transform, etc.
- Webpack 4 → 5 requires polyfill changes, ESM handling, persistent caching adoption
- Jest 26 → 29 changes test environment and ESM behaviour
- Each upgrade *separately* is now too big to fit in one sprint

## Correct

```json
// ✅ Upgraded incrementally; pin policies documented
{
  "dependencies": {
    "react": "^19.0.0",
    "express": "^5.0.0",
    "webpack": "^5.95.0",
    "jest": "^29.7.0",
    "@types/node": "^22.0.0"
  }
}
```

Establish a **monthly upgrade rhythm** rather than letting deps drift for a year.

**Benefits:**
- Each upgrade fits in a small PR
- Security patches and deprecation warnings land while context is fresh
- Avoids the "we can't upgrade React because of 5 transitive blockers" situation

## Remediation Strategy

- **Effort:** S per minor upgrade, M–L per major upgrade
- **When to pay down:**
  - **Patch/minor:** weekly or biweekly automated PRs (Renovate, Dependabot)
  - **Major:** scheduled, one dep at a time, with a release plan
- **Order of operations:** upgrade dev tools (TypeScript, ESLint, Jest) before frameworks; framework before app code

Reference: [Renovate Bot](https://docs.renovatebot.com/) · [Dependabot](https://docs.github.com/en/code-security/dependabot)
