
---
title: Type Coverage
impact: HIGH
impactDescription: Typed public APIs, critical internals, and avoiding implicit any gaps
tags: review, code-review, code-review-type-safety, type-type-coverage
---

## Rule

Review type coverage concerns and flag concrete defects with evidence from the diff and any directly coupled files.

## What to Review

- Look for changes that worsen type coverage or hide existing risk.
- Tie the finding to a concrete failure mode or maintenance cost implied by typed public apis, critical internals, and avoiding implicit any gaps.
- Suggest the smallest concrete fix that addresses the issue without broad refactoring unless the trade-off is explicit.

## Incorrect

```text
A diff introduces type coverage risk, but the review only says 'this feels wrong' without explaining the failure mode.
```

## Correct

```text
The review identifies the exact path, explains why type coverage is risky here, and recommends a concrete fix.
```

## Notes

- Mark `cross_lens_candidate=true` when this issue should trigger another specialized review pass.
- Mark `tradeoff_required=true` only when the fix genuinely changes design, ergonomics, or runtime behavior.

