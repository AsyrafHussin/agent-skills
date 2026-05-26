---
title: Dependency Creep — New Library When Existing One Suffices
impact: HIGH
impactDescription: "Inflates bundle/install size, adds CVE surface, signals model picked training-data favourites"
tags: over-engineering, dependencies, dependency-creep, ai-fingerprint
---

## Dependency Creep — New Library When Existing One Suffices

**Impact: HIGH (Inflates bundle/install size, adds CVE surface, signals model picked training-data favourites)**

AI tends to introduce a new dependency whenever a problem matches a library it has seen in training, even when the project already includes a dependency that solves the same problem. Example signals:

- Adding `date-fns` to a project that already uses `dayjs`
- Adding `axios` to a project that already imports `fetch` everywhere
- Adding `lodash` to a project that already has `lodash-es` (or uses native ES methods)
- Adding `uuid` when the project already uses `nanoid`
- Adding `winston` when the app uses `pino`
- Adding `joi` / `yup` to a TS project that already uses `zod`
- Adding `bcrypt` (the npm package) in a Laravel project where `Hash::make` handles it
- Adding `moment` in a project that just removed `moment` last quarter

Each new dependency:
- Adds to install / bundle size
- Adds a new CVE surface (every `npm audit` lights up)
- Forces the team to maintain TWO libraries for the same concern, indefinitely
- Drifts as the LLM picks whichever was popular in the training-data slice it sampled from

## Incorrect

```json
// ❌ package.json — two libs doing the same job
{
  "dependencies": {
    "dayjs": "^1.11.10",            // already in use
    "date-fns": "^3.0.0",           // ADDED — same purpose
    "axios": "^1.6.0",              // ADDED — but app uses fetch everywhere
    "lodash": "^4.17.21",           // ADDED — lodash-es already present
    "lodash-es": "^4.17.21"
  }
}
```

```php
// ❌ composer.json — two HTTP clients
{
  "require": {
    "guzzlehttp/guzzle": "^7.8",
    "symfony/http-client": "^7.0"          // ADDED — same purpose
  }
}
```

**Why it's slop:**
- Both libraries get pulled into every install
- New `dayjs` code still gets written alongside the new `date-fns` code — drift forever
- Two CVE feeds to track
- Reviewer didn't notice because each PR seems reasonable in isolation

## Correct

```bash
# ✅ Use the existing dep; reject the PR adding the new one with a note

# Before merging a PR that adds a new dependency, ask:
#  1. Does an existing dependency in package.json/composer.json already do this?
#  2. Does a native browser/Node/PHP API already do this?
#  3. Is the new dep > 50KB gzipped (or > 1MB unpacked) for a single function call?
#  4. Does it have CVEs / abandoned upstream?
#
# If any answer is "yes" or "maybe", reject and use the existing tool.
```

Common collisions and the canonical choice:

| Concern | Pick one |
|---|---|
| Date / time | `dayjs` OR `date-fns` (not both) |
| HTTP client | `fetch` OR `axios` (not both) |
| UUID | `uuid` OR `nanoid` (not both) |
| Schema validation (TS) | `zod` (preferred) — reject `joi`/`yup`/`ajv` additions |
| HTTP client (PHP) | `guzzlehttp/guzzle` (Laravel default) — reject `symfony/http-client` unless project-wide |
| Logging | `monolog` (Laravel default) for PHP; pick one of `pino`/`winston` for Node |
| Form validation (Laravel) | Built-in `FormRequest` — reject standalone validation libs |

## Detection

```bash
# Node: dependency-list audit
node -e "
const pkg = require('./package.json');
const all = { ...pkg.dependencies, ...pkg.devDependencies };
const dupes = [
  ['dayjs', 'date-fns', 'moment', 'luxon'],
  ['axios', 'node-fetch', 'got', 'superagent'],
  ['uuid', 'nanoid', 'cuid'],
  ['lodash', 'lodash-es', 'ramda'],
  ['joi', 'yup', 'zod', 'ajv'],
];
for (const group of dupes) {
  const found = group.filter(p => all[p]);
  if (found.length > 1) console.log('OVERLAP:', found.join(', '));
}
"

# PHP: check for HTTP-client overlap
grep -E '\"(guzzlehttp/guzzle|symfony/http-client|kriswallsmith/buzz)\"' composer.json
```

When a new dep lands in a PR, the PR author should justify why the existing options don't fit. "AI suggested this" is not a justification.

Reference: [`technical-debt`'s `deps-unused-deps`](../technical-debt/rules/deps-unused-deps.md) (the broader audit) · [BundlePhobia](https://bundlephobia.com/) (size impact for npm packages)
