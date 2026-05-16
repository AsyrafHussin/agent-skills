---
title: README — Required Content
impact: CRITICAL
impactDescription: "First impression for every reader; missing or stale README is the #1 onboarding blocker"
tags: essential, readme, onboarding
---

## README — Required Content

**Impact: CRITICAL (First impression for every reader; missing or stale README is the #1 onboarding blocker)**

The README is the front door. Every new contributor, every dependency review, every "is this project maintained?" check starts here. A weak README means each onboarding repeats the same questions in Slack.

## Required sections (in order)

````markdown
# Project Name

One-sentence description: what does this do, for whom.

[Badges optional — build, version, license]

## Overview

2–3 sentences expanding on the one-liner. Who uses this, what problem it solves.

## Requirements

- PHP 8.3+ / Node 22+
- MySQL 8.0+ / Redis (if used)
- Any other prerequisites

## Installation

```bash
git clone …
composer install
npm install
cp .env.example .env
php artisan key:generate
php artisan migrate
```

## Usage

How to run it locally:

```bash
php artisan serve            # backend
npm run dev                  # frontend
```

How to run tests:

```bash
php artisan test
npm test
```

## Documentation

- [Architecture overview](docs/architecture/overview.md)
- [Deployment guide](docs/guides/deployment.md)
- [API reference](docs/api/openapi.yaml)

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

[MIT](LICENSE) (or whichever)
````

## Incorrect

```markdown
❌ Sparse README missing essentials
# my-app

A web app.

## Setup

Run npm install. Should work.
```

**Problems:**
- No description of *what* the app does or *for whom*
- No requirements (which Node version? what database?)
- "Should work" — when it doesn't, the reader has no context
- No path to architecture docs, deployment, contributing
- No license — many companies' legal review will block adoption

## What to include vs leave out

**Include:**
- One-sentence description
- Requirements (with versions)
- Install / run / test commands
- Links to deeper docs (`docs/`)
- License

**Don't include:**
- A full API reference (link to `docs/api/`)
- Detailed architecture (link to `docs/architecture/overview.md`)
- Release history (that's `CHANGELOG.md`)
- Long FAQs (`docs/guides/troubleshooting.md`)

A README that's longer than one screen is doing too much. Split it.

## Verification

Add a `Last verified: YYYY-MM-DD` line at the top of installation. Once a quarter, walk through the steps from scratch on a clean checkout and update the date.

```markdown
## Installation

_Last verified on a clean checkout: 2026-04-12_

git clone …
…
```

## Detection

```bash
# Does README exist and have minimum required sections?
test -f README.md || echo "MISSING README.md"
for h in 'Overview' 'Requirements' 'Installation' 'Usage' 'License'; do
  grep -q "^## $h" README.md || echo "MISSING SECTION: ## $h"
done
```

Reference: [Make a README](https://www.makeareadme.com/) · [GitHub — About READMEs](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-readmes)
