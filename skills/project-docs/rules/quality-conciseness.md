---
title: Conciseness — Cut Bloat
impact: HIGH
impactDescription: "Long docs aren't more informative — they're more skipped. The real risk is signal-to-noise."
tags: quality, conciseness, brevity, readability
---

## Conciseness — Cut Bloat

**Impact: HIGH (Long docs aren't more informative — they're more skipped. The real risk is signal-to-noise.)**

A README that's 400 lines doesn't tell the reader more — it tells them less, because they stop reading after the first screen. Good docs say what's needed and stop. Length is a cost, not a virtue.

## Bloat patterns to cut

### 1. Opening boilerplate

```markdown
❌ # Deployment Guide

   This document describes the deployment process for our application.
   It is intended to be read by engineers who are responsible for
   deploying the application to production. Before reading this document,
   you should be familiar with the basics of the application architecture.

   Without further ado, let's dive into the deployment process.

   ## Overview
   ...
```

```markdown
✅ # Deployment Guide

   ## Overview
   ...
```

The reader knows it's a deployment guide — the H1 says so. They know it's for engineers — they're reading the engineering docs. Five paragraphs of "this document describes" is five paragraphs of throat-clearing.

### 2. Restating what code or commands already say

````markdown
❌ ## Installation

   To install the application, you first need to clone the repository
   from GitHub. After cloning the repository, navigate into the
   newly-created directory. Then, install the PHP dependencies by
   running composer install. Once that completes, install the
   JavaScript dependencies by running npm install. Finally, copy the
   example environment file to a new .env file.

   ```bash
   git clone …
   cd …
   composer install
   npm install
   cp .env.example .env
   ```
````

````markdown
✅ ## Installation

   ```bash
   git clone …
   cd …
   composer install
   npm install
   cp .env.example .env
   ```
````

The commands are self-explanatory. Don't narrate them.

### 3. Repetition across sections

A README that says the same thing in the "Overview", then again in "Description", then again in "About this project". Pick one.

### 4. "As mentioned above" / "as we'll see later"

If something needs cross-referencing, link to it. Inline meta-commentary about the document's structure is noise.

### 5. Over-explaining basics

```markdown
❌ Run `composer install`. Composer is a dependency manager for PHP
   that allows you to declare the libraries your project depends on
   and manages (install/update) them for you. To learn more about
   Composer, visit https://getcomposer.org.
```

If the reader is on the install page of your project, they know what Composer is (or they have one click to learn). Don't teach the basics inline.

## Rough length targets

These are heuristics — focused docs can be longer if they have to be, and short docs can be too long if they're padded.

| Doc | Target |
|---|---|
| README | < 200 lines (link to deeper docs for detail) |
| Single guide / runbook | < 300 lines |
| Single ADR | < 150 lines (decision + context + consequences) |
| Architecture overview | < 500 lines (split into multiple docs if longer) |

If you hit these and the content is genuinely necessary, split into multiple focused docs rather than letting one file balloon.

## The "delete a paragraph" exercise

Before publishing a doc, try this: pick any paragraph and ask, "if I deleted this, would a reader miss anything important?" If the answer is no — or "they'd just need to read the next paragraph more carefully" — delete it.

Most docs improve after a 20–30% cut.

## Detection

```bash
# Long markdown files — review for bloat
find docs/ -name '*.md' -exec wc -l {} \; | sort -rn | head -20

# Docs with low signal: many lines containing "this document", "you should", "let's"
# Note: grep -c counts matching LINES (one filler line = 1, even with multiple phrases).
grep -rEcH "(this document|you should|let'?s (dive|look|start))" --include='*.md' docs/ \
  | awk -F: '{ if ($2 > 5) print "BLOATY: " $1 " (" $2 " matching lines)" }'
```

Reference: [Strunk & White — Omit Needless Words](https://en.wikipedia.org/wiki/The_Elements_of_Style) · [Diátaxis — "less is more"](https://diataxis.fr/) · Internal: [`quality-ai-slop`](quality-ai-slop.md) (AI-style filler patterns)
