
---
title: Generic Discipline
impact: MEDIUM
impactDescription: Constraints, parameter bounds, and avoiding generic overuse
tags: review, code-review, code-review-type-safety, type-generic-discipline
---

## Rule

Review generic discipline concerns and flag concrete defects with evidence from the diff and any directly coupled files.

## What to Review

- Look for changes that worsen generic discipline or hide existing risk.
- Tie the finding to a concrete failure mode or maintenance cost implied by constraints, parameter bounds, and avoiding generic overuse.
- Suggest the smallest concrete fix that addresses the issue without broad refactoring unless the trade-off is explicit.

## Incorrect

```text
A diff introduces generic discipline risk, but the review only says 'this feels wrong' without explaining the failure mode.
```

## Correct

```text
The review identifies the exact path, explains why generic discipline is risky here, and recommends a concrete fix.
```

## Notes

- Mark `cross_lens_candidate=true` when this issue should trigger another specialized review pass.
- Mark `tradeoff_required=true` only when the fix genuinely changes design, ergonomics, or runtime behavior.

