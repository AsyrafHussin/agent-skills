
---
title: Testing & Debugging Simplicity
impact: LOW
impactDescription: Hidden side effects, difficult debugging, and hard-to-test design
tags: review, code-review, code-review-simplicity, simp-testing-debugging
---

## Rule

Review testing & debugging simplicity concerns and flag concrete defects with evidence from the diff and any directly coupled files.

## What to Review

- Look for changes that worsen testing & debugging simplicity or hide existing risk.
- Tie the finding to a concrete failure mode or maintenance cost implied by hidden side effects, difficult debugging, and hard-to-test design.
- Suggest the smallest concrete fix that addresses the issue without broad refactoring unless the trade-off is explicit.

## Incorrect

```text
A diff introduces testing & debugging simplicity risk, but the review only says 'this feels wrong' without explaining the failure mode.
```

## Correct

```text
The review identifies the exact path, explains why testing & debugging simplicity is risky here, and recommends a concrete fix.
```

## Notes

- Mark `cross_lens_candidate=true` when this issue should trigger another specialized review pass.
- Mark `tradeoff_required=true` only when the fix genuinely changes design, ergonomics, or runtime behavior.

