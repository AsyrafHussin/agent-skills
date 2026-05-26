---
title: CONTRIBUTING — When and What
impact: MEDIUM
impactDescription: "Sets the rules of engagement; without it, every PR is a guess"
tags: essential, contributing, process
---

## CONTRIBUTING — When and What

**Impact: MEDIUM (Sets the rules of engagement; without it, every PR is a guess)**

A `CONTRIBUTING.md` tells outside (or new-to-the-team) contributors how to participate: which branch to target, commit message style, test expectations, and how PRs get reviewed. GitHub auto-links to it whenever someone opens an issue or PR.

## When required

- **Open-source projects** — required from day one
- **Internal projects with external collaborators** (consultants, contractors) — recommended
- **Solo or single-team internal projects** — optional; an `.md` heading inside the README's "Development" section is fine for small teams

If the project is closed-source and only your team contributes, skip `CONTRIBUTING.md` and put the same information in the README.

## Recommended structure

```markdown
# Contributing

Thanks for considering a contribution!

## Branching

- `main` is protected; PRs only
- Feature branches: `feat/<short-desc>`
- Bug fixes: `fix/<short-desc>`

## Commit messages

We follow [Conventional Commits](https://www.conventionalcommits.org/):

```
feat(auth): add MFA enrollment endpoint
fix(checkout): handle expired payment intent
docs(readme): clarify install steps
```

## Development setup

```bash
git clone …
composer install && npm install
cp .env.example .env
php artisan migrate
```

## Running tests

```bash
php artisan test         # PHP / Laravel
npm test                 # JS / TS
```

All tests must pass; new features need new tests.

## Pull request checklist

- [ ] CHANGELOG.md updated (under `[Unreleased]`)
- [ ] Tests added / updated
- [ ] `composer.lock` / `package-lock.json` committed if deps changed
- [ ] Linked to a GitHub issue (if applicable)

## Review process

- One maintainer approval required
- CI must pass (`tests`, `lint`, `markdown`)
- Squash-merge — keep `main` history linear

## Code of Conduct

This project follows the [Contributor Covenant](CODE_OF_CONDUCT.md).
```

## Incorrect

```markdown
❌ Vague CONTRIBUTING that says "just open a PR"

# Contributing

Open a PR. We'll review.
```

**Problems:**
- No branch policy → reviewer has to comment on every PR
- No test expectation → contributors merge without running them
- No commit style → CHANGELOG generation is harder, history is noisy

## Detection

```bash
# Does CONTRIBUTING exist where needed?
if [ -f "package.json" ] && grep -q '"private"' package.json; then
  : # internal, optional
else
  test -f CONTRIBUTING.md || echo "MISSING CONTRIBUTING.md (recommended for non-private projects)"
fi
```

Reference: [GitHub — Setting guidelines for repository contributors](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/setting-guidelines-for-repository-contributors) · [Conventional Commits](https://www.conventionalcommits.org/)
