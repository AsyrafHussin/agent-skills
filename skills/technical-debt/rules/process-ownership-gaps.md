---
title: Code Without Owners
impact: MEDIUM
impactDescription: "Orphaned code = nobody reviews, nobody maintains, nobody knows"
tags: ownership, codeowners, process
---

## Code Without Owners

**Impact: MEDIUM (Orphaned code = nobody reviews, nobody maintains, nobody knows)**

When code has no clear owner, two things happen: PRs touching it stall (nobody knows who should approve), and when it breaks at 2am, the on-call rotation finds out the hard way. Orphaned code accumulates as the "bus-factor of one" engineer who wrote it changes teams or leaves.

## How to Detect

```bash
# Files not covered by CODEOWNERS
gh api repos/:owner/:repo/contents | jq -r '.[].path' > all-files.txt
# Compare against CODEOWNERS patterns (manual or via `git check-attr` for path-attribute alternative)

# "Author concentration" — files where one person wrote >70% and they're gone
git ls-files | while read f; do
  TOP=$(git log --format='%ae' -- "$f" | sort | uniq -c | sort -rn | head -1)
  echo "$TOP $f"
done | sort -rn | head -30

# High-churn, low-author-count files (bus-factor risk)
git log --since='12 months ago' --name-only --format='COMMIT %ae' | \
  awk '/^COMMIT/{a=$2; next} {print a, $0}' | \
  sort -k2 | uniq -c | sort -rn | head

# Repos lacking a CODEOWNERS file at all
ls .github/CODEOWNERS docs/CODEOWNERS CODEOWNERS 2>/dev/null
```

## Incorrect

```
❌ A typical legacy repo:

- No .github/CODEOWNERS file
- 40% of files last touched by engineers who left 2+ years ago
- Critical billing module written by one engineer, no shared knowledge
- Incident response for `/api/legacy/*` routes pages a randomly-selected on-call
  who has never seen the code
- PRs touching low-traffic areas wait 2 weeks for a review because nobody owns them
```

**Problems:**
- Knowledge debt is invisible until incident — then it's catastrophic
- Code reviews degrade to rubber-stamps because nobody has context
- Refactoring is risky — "is anyone using this?" has no quick answer

## Correct

```
# ✅ .github/CODEOWNERS — every path has an owner team

# Default owners for everything not matched below
*                               @org/platform

# Domain owners
/app/Billing/                   @org/billing-team
/app/Auth/                      @org/identity-team
/app/Notifications/             @org/messaging-team
/resources/js/Pages/Admin/      @org/admin-frontend

# Infrastructure / DevEx
/.github/workflows/             @org/devex
/deploy/                        @org/devex

# Database
/database/migrations/           @org/data-engineering @org/platform
```

Add a CI step to validate the file and check coverage:

```yaml
- name: CODEOWNERS lint + coverage
  uses: mszostok/codeowners-validator@v0.7.4
  with:
    checks: "files,owners,duppatterns,syntax"
    experimental_checks: "notowned"
    github_access_token: ${{ secrets.GITHUB_TOKEN }}
```

The `notowned` experimental check flags files in the repo not matched by any CODEOWNERS pattern — the right tool for "is everything covered?".

**Benefits:**
- GitHub auto-requests reviews from owners — no more "who should review this?"
- Incident response routes to the right team
- New engineers can find a domain expert by path
- Refactor decisions get the input they need

## Remediation Strategy

- **Effort:**
  - **S** — bootstrap a CODEOWNERS with broad team-level patterns
  - **M** — refine ownership as teams form; add path-specific owners
  - **L** — rehome orphaned code (find a new owner team, document context, transfer ownership)
- **When to pay down:**
  - **NOW:** if you've ever asked "who owns this code?" and the answer was unclear
  - **As a project:** quarterly ownership review — re-attest that owners listed are still active
- **Anti-patterns:**
  - **Individual owners** for production code (bus-factor of 1) — prefer team owners
  - **Stale CODEOWNERS** referencing teams or people that no longer exist (CI lint catches this)
  - **Owning everything** by one team — fragment by domain so PRs route fast

**Tip:** when no team wants to own a piece of code, that's a strong signal to delete it (if unused), extract it (if shared), or fold it into a new team's charter (if business-critical).

Reference: [GitHub — CODEOWNERS](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners) · [GitLab — Code Owners](https://docs.gitlab.com/ee/user/project/codeowners/)
