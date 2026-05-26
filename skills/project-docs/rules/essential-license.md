---
title: LICENSE — Required Legal Clarity
impact: CRITICAL
impactDescription: "Without a license, code is technically un-reusable; many companies' legal review blocks adoption outright"
tags: essential, license, legal
---

## LICENSE — Required Legal Clarity

**Impact: CRITICAL (Without a license, code is technically un-reusable; many companies' legal review blocks adoption outright)**

A repository without a LICENSE file means **all rights reserved** by default in most jurisdictions — nobody can legally use, copy, or modify the code, even if it's public. Every serious open-source project has one, and most companies' legal teams reject internal adoption of unlicensed dependencies.

## Required

```
LICENSE                       # at repo root, no extension by convention
```

Contents: the full text of the license you've chosen. For MIT, that's the standard ~20-line text with `<year>` and `<copyright holder>` filled in.

## Choosing a license

Use [choosealicense.com](https://choosealicense.com/) — it asks 2–3 questions and recommends one. Common picks:

| License | When | Notes |
|---|---|---|
| **MIT** | Permissive; you want maximum adoption | Short, well-known, allows commercial use |
| **Apache-2.0** | Permissive + explicit patent grant | Slightly longer; common for company-backed OSS |
| **AGPL-3.0** | Copyleft + network-use clause | SaaS deployments must share modifications |
| **BSL** / **Elastic 2.0** | Source-available, time-delayed open | "Open-ish" — verify it suits your goal |
| **Proprietary** | Closed-source | Add `LICENSE` saying "All rights reserved" + contact for commercial terms |

For internal-only repos, still include a `LICENSE` file stating the company holds copyright and the code is for internal use only. This removes ambiguity for departing employees and contractors.

## Incorrect

```
❌ No LICENSE file at all
.
├── README.md
├── CHANGELOG.md
└── src/...
```

```
❌ Wrong placement / naming
.
├── docs/license.txt          (should be at root, named LICENSE)
└── LICENSE.MD                (works but unconventional case)
```

```
❌ "MIT" mentioned only in README, no file
README:
> ## License
> MIT
```

(GitHub will not detect this as MIT licensed; license badges and tooling will fail.)

## Correct

```
✅ LICENSE at root
.
├── LICENSE                   ← the full license text
├── README.md
└── src/...
```

README links to it:

```markdown
## License

[MIT](LICENSE)
```

## License consistency in package manifests

Keep the manifest field in sync with the LICENSE file:

```json
// package.json
{
  "license": "MIT"
}
```

```json
// composer.json
{
  "license": "MIT"
}
```

Mismatch (LICENSE says MIT, `package.json` says Apache-2.0) trips up `npm audit`, license-scanning tools, and human reviewers.

## Detection

```bash
test -f LICENSE -o -f LICENSE.md -o -f LICENSE.txt || echo "MISSING LICENSE"

# Compare LICENSE to package.json (rough sanity check)
LICENSE_NAME=$(grep -oE '(MIT|Apache|BSD|GPL|ISC|MPL)' LICENSE | head -1)
PKG_LICENSE=$(node -e "console.log(require('./package.json').license)" 2>/dev/null)
echo "LICENSE file: $LICENSE_NAME, package.json: $PKG_LICENSE"
```

Reference: [Choose a License](https://choosealicense.com/) · [GitHub — Licensing a repository](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/licensing-a-repository) · [SPDX license list](https://spdx.org/licenses/)
