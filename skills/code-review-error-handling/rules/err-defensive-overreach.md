
---
title: Defensive-Programming Overreach
impact: LOW
impactDescription: Unnecessary guards and catches that add noise without resilience value
tags: review, code-review, code-review-error-handling, err-defensive-overreach
---

## Rule

Review defensive-programming overreach concerns and flag concrete defects with evidence from the diff and any directly coupled files.

## What to Review

- Look for changes that worsen defensive-programming overreach or hide existing risk.
- Tie the finding to a concrete failure mode or maintenance cost implied by unnecessary guards and catches that add noise without resilience value.
- Suggest the smallest concrete fix that addresses the issue without broad refactoring unless the trade-off is explicit.

## Incorrect

```text
A diff introduces defensive-programming overreach risk, but the review only says 'this feels wrong' without explaining the failure mode.
```

## Correct

```text
The review identifies the exact path, explains why defensive-programming overreach is risky here, and recommends a concrete fix.
```

## Notes

- Mark `cross_lens_candidate=true` when this issue should trigger another specialized review pass.
- Mark `tradeoff_required=true` only when the fix genuinely changes design, ergonomics, or runtime behavior.

