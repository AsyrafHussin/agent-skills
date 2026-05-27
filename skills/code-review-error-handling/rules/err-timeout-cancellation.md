
---
title: Timeout & Cancellation Discipline
impact: CRITICAL
impactDescription: Explicit timeouts, cancellation propagation, and bounded waits
tags: review, code-review, code-review-error-handling, err-timeout-cancellation
---

## Rule

Review timeout & cancellation discipline concerns and flag concrete defects with evidence from the diff and any directly coupled files.

## What to Review

- Look for changes that worsen timeout & cancellation discipline or hide existing risk.
- Tie the finding to a concrete failure mode or maintenance cost implied by explicit timeouts, cancellation propagation, and bounded waits.
- Suggest the smallest concrete fix that addresses the issue without broad refactoring unless the trade-off is explicit.

## Incorrect

```text
A diff introduces timeout & cancellation discipline risk, but the review only says 'this feels wrong' without explaining the failure mode.
```

## Correct

```text
The review identifies the exact path, explains why timeout & cancellation discipline is risky here, and recommends a concrete fix.
```

## Notes

- Mark `cross_lens_candidate=true` when this issue should trigger another specialized review pass.
- Mark `tradeoff_required=true` only when the fix genuinely changes design, ergonomics, or runtime behavior.
- Name the timeout or cancellation boundary explicitly and describe what the caller observes when it expires.
