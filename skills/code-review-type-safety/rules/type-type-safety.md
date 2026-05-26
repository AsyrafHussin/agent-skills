
---
title: Type Safety
impact: CRITICAL
impactDescription: Unsafe casts, unknown data, discriminated unions, and runtime checks
tags: review, code-review, code-review-type-safety, type-type-safety
---

## Rule

Review type safety concerns and flag concrete defects with evidence from the diff and any directly coupled files.

## What to Review

- Look for changes that worsen type safety or hide existing risk.
- Tie the finding to a concrete failure mode or maintenance cost implied by unsafe casts, unknown data, discriminated unions, and runtime checks.
- Suggest the smallest concrete fix that addresses the issue without broad refactoring unless the trade-off is explicit.

## Incorrect

```text
A diff introduces type safety risk, but the review only says 'this feels wrong' without explaining the failure mode.
```

## Correct

```text
The review identifies the exact path, explains why type safety is risky here, and recommends a concrete fix.
```

## Notes

- Mark `cross_lens_candidate=true` when this issue should trigger another specialized review pass.
- Mark `tradeoff_required=true` only when the fix genuinely changes design, ergonomics, or runtime behavior.
- Include one unexpected input shape and explain what runtime validation or the type system does with it.
