---
title: Heading Hierarchy — H1 Once, No Skipped Levels
impact: HIGH
impactDescription: "Broken heading hierarchy breaks readability, accessibility, and auto-generated TOCs"
tags: quality, headings, structure, accessibility
---

## Heading Hierarchy — H1 Once, No Skipped Levels

**Impact: HIGH (Broken heading hierarchy breaks readability, accessibility, and auto-generated TOCs)**

Each markdown file has exactly one H1 (the document title), and lower levels go in sequence (H2 → H3 → H4) without skipping. Auto-generated TOCs, screen readers, and GitHub's outline view all rely on this. A file with three H1s or with H1 → H3 jumps reads as broken to humans and as malformed to tools.

## Rules

1. **Exactly one H1 per file** — it's the title
2. **No skipping levels** — H2 can be followed by H2 or H3, but not H4
3. **Don't use bold instead of a heading** — `**Important:**` doesn't show up in TOCs
4. **Don't use H1 inside a doc** — once you've used `#`, use only `##` and below
5. **Headings should be descriptive, not generic** — `## Configuration` not `## Section 2`

## Incorrect

### Multiple H1s

```markdown
# Deployment Guide

## Overview

Some overview content.

# Configuration

Some configuration content.       ← second H1; should be ##

# Troubleshooting

…                                  ← third H1
```

The doc has three "top-level" sections in markdown's eyes; GitHub will treat each H1 as a candidate document title. The TOC will be flat and confused.

### Skipped levels

```markdown
# Architecture Overview

## Components                       ← H2

#### Authentication                 ← H4 (skipped H3)

Some content about auth.

#### Authorization                  ← H4 still

#### Sessions                       ← H4 still
```

The reader expects "Components" to have direct sub-sections; instead it has sub-sub-sections. Auto-generated outlines look broken.

### Bold-as-heading

````markdown
## Setup

**Prerequisites:**                  ← bold, not heading

- Node 22
- MySQL 8

**Install:**                        ← bold, not heading

```bash
npm install
```
````

"Prerequisites" and "Install" don't appear in the TOC; readers scanning headings miss them.

## Correct

```markdown
# Deployment Guide                  ← exactly one H1, matching the file's purpose

## Overview

Brief overview content.

## Configuration

### Environment variables           ← H3 under H2

### Secrets storage

## Troubleshooting

### Build failures                  ← H3 under H2

### Deploy timeouts
```

Sequential, predictable, scannable.

## Heading content

### Use sentence case

```
✓ ## Setting up a development environment
✗ ## Setting Up A Development Environment   (title case is awkward to read)
✗ ## SETTING UP A DEVELOPMENT ENVIRONMENT   (shouting)
```

### Be specific

```
✓ ## Resetting a user's password from the admin panel
✗ ## Password reset                          (which kind? from where?)
```

### Match how readers search

The TOC of a 1000-line doc is its index. Headings should answer "what would I search for?", not be cute.

## Detection

```bash
# Files with more than one H1 (set -e safe — uses if/then instead of && chain)
find docs/ -name '*.md' -print0 | while IFS= read -r -d '' f; do
  H1_COUNT=$(grep -cE '^# [^#]' "$f" || true)
  if [ "$H1_COUNT" -gt 1 ]; then
    echo "MULTIPLE H1s ($H1_COUNT): $f"
  fi
done

# Skipped heading levels (markdownlint rule MD001)
npx markdownlint-cli2 'docs/**/*.md' 'README.md'
# (Configure .markdownlint.json: { "MD001": true })
```

`markdownlint`'s rule MD001 (`heading-increment`) catches skipped levels automatically; MD025 (`single-h1`) catches multiple H1s. Enable both.

Reference: [Markdownlint rules MD001 / MD025](https://github.com/DavidAnson/markdownlint/blob/main/doc/Rules.md) · [WAI-ARIA Heading Levels](https://www.w3.org/WAI/tutorials/page-structure/headings/)
