---
title: kebab-case for Files Under docs/
impact: CRITICAL
impactDescription: "URLs, search, and grep all behave better with lowercase-hyphenated names"
tags: naming, docs-folder, kebab-case
---

## kebab-case for Files Under docs/

**Impact: CRITICAL (URLs, search, and grep all behave better with lowercase-hyphenated names)**

Files under `docs/` get linked, served as URLs by static-site generators, and grepped daily. `kebab-case` (lowercase, hyphen-separated) avoids case-sensitivity bugs, generates clean URLs (`docs/deployment-guide.md` → `/deployment-guide`), and is the dominant convention across documentation sites.

## Incorrect

```
❌ Mixed cases, spaces, underscores, PascalCase
docs/
├── DeploymentGuide.md          # PascalCase
├── deployment_guide.md         # snake_case
├── Deployment Guide.md         # spaces — break URLs
├── Architecture Overview.md    # spaces + Title Case
├── api-Reference.md            # mixed case
├── data_model.MD               # uppercase extension
└── Onboarding.md               # initial capital
```

**Problems:**
- Mixed conventions force readers to guess each filename
- Spaces in filenames produce `%20`-encoded URLs that are hard to type and look broken
- Case mismatches between branches cause "file not found" on Linux CI while working on macOS/Windows
- Static-site generators usually lowercase URLs anyway, so `DeploymentGuide.md` and `deployment-guide.md` collide

## Correct

```
✅ Consistent kebab-case throughout docs/
docs/
├── architecture/
│   ├── overview.md
│   ├── data-model.md
│   └── service-boundaries.md
├── guides/
│   ├── deployment-guide.md
│   ├── local-development.md
│   ├── getting-started.md
│   └── api-reference.md
└── runbooks/
    ├── incident-response.md
    └── deploy-production.md
```

**Benefits:**
- Predictable: anyone can guess the filename from the topic
- URL-friendly: `/docs/guides/deployment-guide` reads naturally
- Case-safe: lowercase eliminates case-sensitivity differences across OSes
- Greppable: `grep -rn 'deployment-guide' docs/` always finds the file

## Conventions

- **All lowercase** — `deployment-guide.md`, not `Deployment-Guide.md`
- **Hyphens, not underscores** — `data-model.md`, not `data_model.md`
- **No spaces** — ever
- **`.md` extension lowercase** — `.md`, not `.MD`
- **Keep names short** — 2–4 words ideal; if you need 6, the doc may be doing too much
- **Use nouns or noun-phrases** — `deployment-guide.md`, not `how-to-deploy.md` (the folder structure already conveys the verb)

## Detection

```bash
# Find files in docs/ with bad casing or characters
find docs/ -name '*.md' | grep -E '[A-Z]|[[:space:]]|_'
```

Any hit is a candidate for renaming via `git mv`. Add a markdownlint rule or a CI grep to prevent regression.

Reference: [Diátaxis — naming](https://diataxis.fr/) · [Markdownlint](https://github.com/DavidAnson/markdownlint)
