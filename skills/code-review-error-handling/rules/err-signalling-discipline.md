
---
title: Error Signalling Discipline
impact: CRITICAL
impactDescription: Ignored return codes, unchecked fallible operations, and silent partial-success paths
tags: review, code-review, code-review-error-handling, err-signalling-discipline
---

## Rule

Review error signalling discipline concerns and flag concrete defects with evidence from the diff and any directly coupled files.

## What to Review

- Look for changes that worsen error signalling discipline or hide existing risk.
- Tie the finding to a concrete failure mode or maintenance cost implied by ignored return codes, unchecked fallible operations, and silent partial-success paths.
- Suggest the smallest concrete fix that addresses the issue without broad refactoring unless the trade-off is explicit.

## Incorrect

```text
A diff introduces error signalling discipline risk, but the review only says 'this feels wrong' without explaining the failure mode.
```

## Correct

```text
The review identifies the exact path, explains why error signalling discipline is risky here, and recommends a concrete fix.
```

## Notes

- Mark `cross_lens_candidate=true` when this issue should trigger another specialized review pass.
- Mark `tradeoff_required=true` only when the fix genuinely changes design, ergonomics, or runtime behavior.

