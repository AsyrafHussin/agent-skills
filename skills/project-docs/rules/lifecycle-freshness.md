---
title: Freshness Dates on Architecture Docs
impact: MEDIUM
impactDescription: "Without a freshness signal, every reader has to guess if the doc is still accurate"
tags: lifecycle, freshness, verification
---

## Freshness Dates on Architecture Docs

**Impact: MEDIUM (Without a freshness signal, every reader has to guess if the doc is still accurate)**

An architecture doc from 2022 might still be accurate or might be wildly out of date — without an explicit "Last verified" line, the reader can't tell. Adding a freshness date converts a doc from "trust at your own risk" to "verified accurate as of YYYY-MM-DD".

## What needs a freshness date

| Doc type | Freshness needed? |
|---|---|
| `docs/architecture/*` | **Yes** — these describe how the system *currently* works |
| `docs/guides/*` | **Yes** — setup steps change |
| `docs/runbooks/*` | **Yes** — procedures must be verified |
| `docs/api/*` | If hand-written; not if auto-generated from code |
| `docs/adr/*` | **No** — ADRs are dated by design (Date: field); they describe a past decision |
| `docs/archive/*` | **No** — archived = frozen in time |
| `README.md` | The "Installation" section — yes |

## Correct

```markdown
# Architecture Overview

_Last verified on a fresh checkout: 2026-04-12 by @asyraf_

## System layout

The application is a Laravel monolith with an Inertia.js + React frontend …
```

Or as a table at the top:

```markdown
| Field | Value |
|---|---|
| Last verified | 2026-04-12 |
| Verified by   | @asyraf |
| Owner         | @org/platform |
```

## Incorrect

```markdown
❌ No freshness signal anywhere
# Architecture Overview

The application uses MySQL for orders and Redis for sessions.
…
```

(Was that true in 2022? Still true today? You'd have to ask.)

## When to refresh

Set a cadence per doc-type:

- **Architecture docs** — quarterly (or when a major change ships)
- **Runbooks** — verify by *executing* them quarterly (a "walk-through" exercise)
- **Setup / install guides** — verify by running them on a clean checkout quarterly
- **API references** — when they're hand-written, every time a public endpoint changes

The freshness date is part of the doc's content; updating it is part of the verification work, not separate paperwork.

## Detection

```bash
# Find architecture / guide / runbook docs without a "Last verified" line
for f in docs/architecture/*.md docs/guides/*.md docs/runbooks/*.md; do
  grep -qE '_?Last verified' "$f" || echo "NO FRESHNESS DATE: $f"
done

# Find docs whose "Last verified" date is > 12 months ago.
# Compute threshold in bash (BSD awk on macOS doesn't have strftime/systime).
THRESHOLD=$(date -v-12m +%F 2>/dev/null || date -d '12 months ago' +%F)
grep -rEoH 'Last verified[: ]+[0-9]{4}-[0-9]{2}-[0-9]{2}' docs/ \
  | awk -F: -v t="$THRESHOLD" '$NF < t { print "STALE: " $0 }'
```

## CI: prompt-rather-than-block

Don't block PRs on freshness dates — that creates noise. Instead, prompt with a weekly digest:

```yaml
# .github/workflows/docs-freshness.yml — runs weekly
- name: List stale docs
  run: |
    THRESHOLD=$(date -d '12 months ago' +%F)
    grep -rEoH 'Last verified[: ]+[0-9]{4}-[0-9]{2}-[0-9]{2}' docs/ \
      | awk -F'verified[: ]+' -v t="$THRESHOLD" '$2 < t { print }' \
      | tee stale-docs.txt
    # Open a GitHub issue listing stale docs (or post to Slack)
```

Reference: [Diátaxis — keeping documentation maintained](https://diataxis.fr/) · Internal: `docs-outdated-architecture` rule in the [technical-debt](../../technical-debt) skill covers detection of stale content
