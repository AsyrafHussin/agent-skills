
---
title: Cognitive Load
impact: HIGH
impactDescription: Parameter count, responsibility load, inheritance depth, and mental model cost
tags: review, code-review, code-review-simplicity, simp-cognitive-load
---

## Rule

Review cognitive load concerns and flag concrete defects with evidence from the diff and any directly coupled files.

## What to Review

- Look for changes that worsen cognitive load or hide existing risk.
- Tie the finding to a concrete failure mode or maintenance cost implied by parameter count, responsibility load, inheritance depth, and mental model cost.
- Suggest the smallest concrete fix that addresses the issue without broad refactoring unless the trade-off is explicit.

## Incorrect

```text
A diff introduces cognitive load risk, but the review only says 'this feels wrong' without explaining the failure mode.
```

## Correct

```text
The review identifies the exact path, explains why cognitive load is risky here, and recommends a concrete fix.
```

## Notes

- Mark `cross_lens_candidate=true` when this issue should trigger another specialized review pass.
- Mark `tradeoff_required=true` only when the fix genuinely changes design, ergonomics, or runtime behavior.

