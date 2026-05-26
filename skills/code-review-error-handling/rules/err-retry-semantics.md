
---
title: Retry Semantics
impact: HIGH
impactDescription: Max attempts, backoff, idempotency, and retryable error classes
tags: review, code-review, code-review-error-handling, err-retry-semantics
---

## Rule

Review retry semantics concerns and flag concrete defects with evidence from the diff and any directly coupled files.

## What to Review

- Look for changes that worsen retry semantics or hide existing risk.
- Tie the finding to a concrete failure mode or maintenance cost implied by max attempts, backoff, idempotency, and retryable error classes.
- Suggest the smallest concrete fix that addresses the issue without broad refactoring unless the trade-off is explicit.

## Incorrect

```text
A diff introduces retry semantics risk, but the review only says 'this feels wrong' without explaining the failure mode.
```

## Correct

```text
The review identifies the exact path, explains why retry semantics is risky here, and recommends a concrete fix.
```

## Notes

- Mark `cross_lens_candidate=true` when this issue should trigger another specialized review pass.
- Mark `tradeoff_required=true` only when the fix genuinely changes design, ergonomics, or runtime behavior.

