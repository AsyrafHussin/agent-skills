
---
title: Partial-Failure Handling
impact: HIGH
impactDescription: Batch safety, compensation, and caller-visible partial-success contracts
tags: review, code-review, code-review-error-handling, err-partial-failure
---

## Rule

Review partial-failure handling concerns and flag concrete defects with evidence from the diff and any directly coupled files.

## What to Review

- Look for changes that worsen partial-failure handling or hide existing risk.
- Tie the finding to a concrete failure mode or maintenance cost implied by batch safety, compensation, and caller-visible partial-success contracts.
- Suggest the smallest concrete fix that addresses the issue without broad refactoring unless the trade-off is explicit.

## Incorrect

```text
A diff introduces partial-failure handling risk, but the review only says 'this feels wrong' without explaining the failure mode.
```

## Correct

```text
The review identifies the exact path, explains why partial-failure handling is risky here, and recommends a concrete fix.
```

## Notes

- Mark `cross_lens_candidate=true` when this issue should trigger another specialized review pass.
- Mark `tradeoff_required=true` only when the fix genuinely changes design, ergonomics, or runtime behavior.

