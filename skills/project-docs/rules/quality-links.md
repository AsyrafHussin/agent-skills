---
title: Links — Descriptive Text and No Broken URLs
impact: MEDIUM
impactDescription: "'Click here' is unscannable and inaccessible; broken links erode trust in the whole doc set"
tags: quality, links, accessibility, broken-links
---

## Links — Descriptive Text and No Broken URLs

**Impact: MEDIUM ("Click here" is unscannable and inaccessible; broken links erode trust in the whole doc set)**

Two failure modes degrade docs quickly: link text that doesn't describe where the link goes ("click here", "this link") and links that 404 because the target moved. Both are easy to detect and fix; left unchecked, both compound until readers stop trusting the docs.

## Rule 1 — Descriptive link text

```markdown
❌ For deployment instructions, click [here](docs/guides/deployment.md).
❌ See [this page](docs/guides/deployment.md) for deployment.
❌ Read more about deployment [here](docs/guides/deployment.md).

✅ See the [deployment guide](docs/guides/deployment.md).
✅ Deployment instructions are in [docs/guides/deployment.md](docs/guides/deployment.md).
✅ For details, see the [deployment guide](docs/guides/deployment.md).
```

Why descriptive text matters:

1. **Scannability** — a reader skimming the doc sees "deployment guide", not "click here"
2. **Accessibility** — screen readers can announce just the link text in isolation; "click here" out of context is meaningless
3. **Search** — "deployment guide" is a useful keyword; "click here" is not

### Quick test

Read just the link text out loud (ignore the surrounding sentence). Does it tell you where you'd land? If yes, it's good. If "click here" / "this link" / "more", rewrite.

## Rule 2 — Use relative paths for in-repo links

```markdown
❌ See https://github.com/your-org/your-repo/blob/main/docs/architecture/overview.md
✅ See [docs/architecture/overview.md](docs/architecture/overview.md)
```

Relative paths:
- Work on any fork or mirror
- Don't break when the repo is renamed or moved
- Resolve correctly when docs are served via MkDocs / GitHub Pages

Use absolute URLs only for **off-repo** targets (external sites, other repos).

## Rule 3 — Check links don't 404

Broken links happen when files are renamed, moved, or deleted without updating their referrers. The fix is automation: a link checker in CI.

```yaml
# .github/workflows/check-links.yml
name: Check links
on:
  pull_request:
    paths: ['**/*.md']
  schedule: [{ cron: '0 9 * * MON' }]   # weekly catches link-rot

jobs:
  lychee:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: lycheeverse/lychee-action@v2
        with:
          args: --no-progress --exclude-mail './**/*.md' README.md
          fail: true
```

[Lychee](https://github.com/lycheeverse/lychee) is the recommended tool: fast (Rust), supports config files, exits non-zero on broken links so CI fails the PR.

For per-file checks during local development:

```bash
npx markdown-link-check docs/guides/deployment.md
```

## Rule 4 — Don't link to the current file

```markdown
❌ See [this section](#installation) for installation.   (when "this section" IS the installation section)
```

Self-references are usually filler. The reader is already there.

## Rule 5 — Use reference-style links for repeated targets

When the same URL appears 3+ times in a doc, use reference-style:

```markdown
Some prose linking to the [Laravel docs][laravel].

Some more prose also linking to the [Laravel docs][laravel] later on.

Yet more prose, third time linking to the [Laravel docs][laravel].

[laravel]: https://laravel.com/docs/11.x
```

The doc stays readable in source form, and updating the URL in one place updates all occurrences.

## Detection

```bash
# Non-descriptive link text
grep -rEnH '\[(click here|here|this|this link|read more|more|link)\]\(' \
  --include='*.md' docs/ README.md

# Absolute URLs to your own repo (should be relative)
# EDIT: replace with your actual GitHub org/repo before running.
ORG_REPO="your-org/your-repo"
grep -rEnH "github\.com/$ORG_REPO/(blob|tree)/" --include='*.md' .

# Run link checker locally
npx lychee --no-progress './**/*.md' README.md
```

Reference: [Lychee](https://github.com/lycheeverse/lychee) · [markdown-link-check](https://github.com/tcort/markdown-link-check) · [WCAG 2.4.4 — Link Purpose](https://www.w3.org/WAI/WCAG21/Understanding/link-purpose-in-context.html)
