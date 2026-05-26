
---
title: Type Correctness
impact: CRITICAL
impactDescription: Annotations, assertions, and guards that match real behavior
tags: review, code-review, code-review-type-safety, type-type-correctness
---

## Rule

Review type correctness concerns and flag concrete defects with evidence from the diff and any directly coupled files.

## What to Review

- Look for changes that worsen type correctness or hide existing risk.
- Tie the finding to a concrete failure mode or maintenance cost implied by annotations, assertions, and guards that match real behavior.
- Suggest the smallest concrete fix that addresses the issue without broad refactoring unless the trade-off is explicit.

## Incorrect

```text
A diff introduces type correctness risk, but the review only says 'this feels wrong' without explaining the failure mode.
```

## Correct

```text
The review identifies the exact path, explains why type correctness is risky here, and recommends a concrete fix.
```

## Notes

- Mark `cross_lens_candidate=true` when this issue should trigger another specialized review pass.
- Mark `tradeoff_required=true` only when the fix genuinely changes design, ergonomics, or runtime behavior.

