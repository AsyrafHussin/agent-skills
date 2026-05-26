
---
title: Resource Cleanup on Error Paths
impact: HIGH
impactDescription: Scoped release patterns for files, locks, sockets, and transactions
tags: review, code-review, code-review-error-handling, err-resource-cleanup
---

## Rule

Review resource cleanup on error paths concerns and flag concrete defects with evidence from the diff and any directly coupled files.

## What to Review

- Look for changes that worsen resource cleanup on error paths or hide existing risk.
- Tie the finding to a concrete failure mode or maintenance cost implied by scoped release patterns for files, locks, sockets, and transactions.
- Suggest the smallest concrete fix that addresses the issue without broad refactoring unless the trade-off is explicit.

## Incorrect

```text
A diff introduces resource cleanup on error paths risk, but the review only says 'this feels wrong' without explaining the failure mode.
```

## Correct

```text
The review identifies the exact path, explains why resource cleanup on error paths is risky here, and recommends a concrete fix.
```

## Notes

- Mark `cross_lens_candidate=true` when this issue should trigger another specialized review pass.
- Mark `tradeoff_required=true` only when the fix genuinely changes design, ergonomics, or runtime behavior.

