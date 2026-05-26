---
title: Outdated Architecture Documentation
impact: MEDIUM
impactDescription: "Onboarding and incident response collapse without accurate maps"
tags: documentation, architecture, readme
---

## Outdated Architecture Documentation

**Impact: MEDIUM (Onboarding and incident response collapse without accurate maps)**

Architecture docs that don't match the running system cause the worst kind of mistake: confident wrong decisions. The cost shows up at the worst moments — onboarding a new engineer or debugging a 2am incident.

## How to Detect

For each architecture doc in the repo:
- **README "Getting Started":** can a brand-new engineer follow it end-to-end *today* and end with a working dev environment?
- **Architecture diagrams:** do all named services still exist with the same names?
- **Database schema diagrams:** do all tables and columns exist?
- **Sequence diagrams:** do the request paths still match the code?

```bash
# Find architecture docs and check last-modified vs the code they describe
find . \( -name 'ARCHITECTURE.md' -o -name 'README.md' -o -path '*/docs/*' \) | \
  while read doc; do
    DOC_DATE=$(git log -1 --format=%ct -- "$doc")
    SRC_DATE=$(git log -1 --format=%ct -- src/ app/)
    if [ $((SRC_DATE - DOC_DATE)) -gt 7776000 ]; then       # 90 days
      echo "STALE: $doc (last touched $(date -r $DOC_DATE +%Y-%m-%d))"
    fi
  done
```

## Incorrect

```markdown
<!-- ❌ README from 18 months ago -->
# CheckoutService

## Architecture
- Node.js 14 (LTS)
- MongoDB for orders
- Redis for sessions
- Stripe for payments

## Getting Started
1. Clone the repo
2. Copy `.env.example` to `.env` and fill in the DB credentials
3. `composer install && npm install`
4. `php artisan migrate && php artisan serve`

## Services
The service is split into:
- /api    — REST endpoints
- /jobs   — background workers
- /web    — admin dashboard
```

Reality:
- Migrated to Node 22 last year
- Moved from MongoDB to MySQL 8 months ago
- Stripe was replaced with Adyen
- `.env.example` no longer exists; uses Doppler
- `/web` was extracted to its own repo 6 months ago

**Problems:**
- New engineer spends 2 days debugging "why won't MongoDB connect"
- Incident responder pages the wrong on-call (the old `/web` team)
- Architectural decisions get made against an imaginary system

## Correct

```markdown
<!-- ✅ README that matches reality, with a freshness signal -->
# CheckoutService

_Last verified against running system: 2026-05-10_

## Architecture
- Node.js 22 (LTS)
- MySQL 8.4 for orders
- Redis for sessions
- Adyen for payments (migrated from Stripe — see `docs/adr/0007-adyen.md`)

## Getting Started
1. `gh repo clone org/checkout && cd checkout`
2. `cp .env.example .env`; fill in DB and Adyen credentials (or pull via `doppler setup`)
3. `composer install && npm install`
4. `php artisan migrate --seed && php artisan serve` (in another shell: `npm run dev`)

## Services
- `/api`  — REST endpoints
- `/jobs` — background workers
- (Admin dashboard lives in `org/checkout-admin`)
```

**Benefits:**
- Onboarding works end-to-end without hidden tribal knowledge
- Incident responders trust the doc to find the right team
- Each ADR captures the *why* for architectural changes

## Remediation Strategy

- **Effort:** S–M per doc
- **When to pay down:**
  - Whenever a PR changes architecture (new service, new datastore, new dependency): **update the doc in the same PR**
  - Quarterly: "doc walk" — pair an engineer with a brand-new hire, follow the README, fix what breaks
- **Tip:** add a `Last verified` date to architecture docs; treat doc-aging like dep-aging

Reference: [Architecture Decision Records (ADR)](https://adr.github.io/) · [The Diátaxis Framework](https://diataxis.fr/)
