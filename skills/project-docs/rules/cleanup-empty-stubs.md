---
title: Empty Stubs and Placeholder Files
impact: MEDIUM
impactDescription: "Stubs make the docs tree look complete when it isn't — false confidence"
tags: cleanup, stubs, placeholder, tbd
---

## Empty Stubs and Placeholder Files

**Impact: MEDIUM (Stubs make the docs tree look complete when it isn't — false confidence)**

An empty stub is a markdown file that exists but contains nothing useful: just a heading, just "TBD", just a placeholder paragraph. Stubs are dangerous because they make the docs index look populated while delivering nothing — readers click expecting content and get an empty page, eroding trust in the entire docs.

## How to recognize stubs

### 1. Files with only a heading

```markdown
# Deployment Guide

```

(One line, then nothing. 100% useless.)

### 2. Files with "TBD" / "Coming soon" / placeholder content

```markdown
# API Reference

TBD — will document this later.
```

```markdown
# Architecture Overview

This document describes the architecture of the system.

(coming soon)
```

### 3. Files with only Lorem ipsum / template content

```markdown
# Title

Lorem ipsum dolor sit amet, consectetur adipiscing elit...
```

### 4. Files with the rule-template unchanged

(A bigger problem in skill repos like this one) — an `.md` file containing the literal `## Rule Title Here` template heading that was never edited.

### 5. Tiny files (< 30 lines, almost no information)

A 12-line `architecture.md` that just says "We use Laravel and React" is a stub even if it doesn't say "TBD".

## Incorrect

```markdown
❌ A stub that pretends to be documentation
docs/guides/deployment.md (the entire file):

# Deployment

TBD
```

**Problems:**
- A reader clicking this link gets nothing
- The file's existence implies the topic is documented when it isn't
- The "TBD" was written 18 months ago and forgotten

## Correct — three options

1. **Fill it in** — if you're going to write the doc, just write it (or at least cover the basics in 50+ lines). Don't create an empty file as a "reminder".

2. **Delete it** — if there's no plan to fill it in, delete. Reduce false expectations.

3. **Promote to an issue** — if you genuinely want to track "we need to write this someday", open a GitHub issue. The issue is for tracking; the markdown file is for content. Don't conflate them.

```markdown
✅ Replace with content OR delete
docs/guides/deployment.md (replacement):

# Deployment

Production deploys happen automatically when a tag is pushed to `main`:

1. Run tests locally: `php artisan test`
2. Bump the version in `composer.json`
3. Update CHANGELOG.md under `[Unreleased]`
4. Tag: `git tag v1.2.3 && git push --tags`
5. CI builds, tests, and deploys to production within 10 minutes
…etc, with real content.
```

## Detection

```bash
# Files under 30 lines (likely stubs)
find docs/ -name '*.md' -type f -exec wc -l {} \; | \
  awk '$1 < 30 { print }' | sort -n

# Files containing "TBD" / "Coming soon" / "Lorem"
grep -rlEi '\b(TBD|Coming soon|Lorem ipsum|placeholder|to be (done|written|filled))\b' \
  --include='*.md' docs/

# Files matching the rule-template heading (in this skills repo)
grep -rln '^## Rule Title Here$' --include='*.md' rules/

# Files with only a title and no content
for f in $(find docs/ -name '*.md'); do
  CONTENT_LINES=$(grep -cvE '^(#|$)' "$f")
  [ "$CONTENT_LINES" -lt 3 ] && echo "NEAR-EMPTY: $f"
done
```

## Prevention: don't create empty files

A common anti-pattern is creating the docs/ skeleton up-front: `touch docs/architecture.md docs/deployment.md docs/api.md`. This produces nothing but stubs.

Instead: create each doc file *when you have content for it*. The folder structure can be empty until then.

Reference: [Diátaxis — "documentation is not a TODO list"](https://diataxis.fr/)
