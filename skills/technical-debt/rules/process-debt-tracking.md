---
title: Untracked Technical Debt
impact: MEDIUM
impactDescription: "Debt that isn't tracked can't be prioritized or budgeted"
tags: debt-register, process, prioritization
---

## Untracked Technical Debt

**Impact: MEDIUM (Debt that isn't tracked can't be prioritized or budgeted)**

Debt that exists only in engineers' heads (or in scattered TODOs and Slack messages) competes with product features by stealth — engineers slow down, but leadership can't see why. A debt register makes the cost visible, so it can be funded properly instead of being paid in invisible overtime.

## How to Detect

Check the project for an explicit debt-tracking mechanism:

- **Issue tracker label** (e.g., GitHub `tech-debt`, Linear `Debt` project)
- **Dedicated debt board** (Trello, Notion, or a `DEBT.md` in the repo)
- **ADRs** for major debt accumulation decisions
- **Quarterly debt-paydown allocation** (e.g., 20% capacity)

```bash
# Quick repo check
ls DEBT.md TECH_DEBT.md docs/debt/ 2>/dev/null
gh issue list --label "tech-debt" --state=all
grep -rE 'tech.?debt' .github/ docs/
```

If none of these exist, the project has **process debt about technical debt** — meta-debt.

## Incorrect

```
❌ Debt situation in a typical repo:
- 47 TODO comments, no tickets
- 12 known "we should rewrite that" conversations on Slack
- 3 engineers each have a mental list of "things that scare me"
- Last debt-paydown sprint: never
- When asked "what's our biggest debt?": four engineers give four different answers
```

**Problems:**
- Debt accrues invisibly — leadership only sees velocity drop
- Same problem gets discovered repeatedly by new hires
- No paydown budget because no list to justify the budget

## Correct

Pick **one** lightweight mechanism and use it consistently:

```markdown
<!-- ✅ Option A: DEBT.md in the repo (low ceremony) -->
# Technical Debt Register

| # | Category | Item | Effort | Impact | Owner | Linked |
|---|----------|------|--------|--------|-------|--------|
| 1 | deps     | guzzle 6.x (5y behind, blocks PHP 8.4)  | M | HIGH     | @asyraf | #1842 |
| 2 | code     | OrderService god class (820 LoC)        | L | HIGH     | @team-orders | #1844 |
| 3 | test     | Checkout flow has no integration test   | M | CRITICAL | @asyraf | #1845 |

Last reviewed: 2026-05-01.  Next review: 2026-08-01.
```

```yaml
# ✅ Option B: GitHub label + saved search
# Label every debt-related issue with 'tech-debt'
# Saved search: https://github.com/org/repo/issues?q=is%3Aopen+label%3Atech-debt+sort%3Areactions-%2B1-desc
```

**Allocate budget:** dedicate a consistent fraction of every sprint to debt paydown (commonly 15–25%). Without an allocation, debt always loses to features.

**Benefits:**
- Debt is visible to product and leadership
- Prioritization is principled (effort × impact), not loudest-engineer
- Paydown velocity is measurable

## Remediation Strategy

- **Effort:** S to start (one register + a label), ongoing M to maintain
- **When to pay down:**
  - **Now:** start the register with the top 10 items from your last audit
  - **Quarterly:** review and reprioritize; close completed entries
  - **Per PR:** if a PR introduces accepted debt (shortcut, missing test), add a register entry as part of merge

**Anti-patterns:**
- Register that nobody owns → goes stale, becomes worse than nothing
- Register with 200 entries → useless; cap at top 20–30 active items
- Debt sprints disconnected from a register → effort goes to whatever's annoying that week, not what matters

Reference: [Martin Fowler — Technical Debt Quadrant](https://martinfowler.com/bliki/TechnicalDebtQuadrant.html) · [ThoughtWorks Tech Radar — Debt Register](https://www.thoughtworks.com/radar)
