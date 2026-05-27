
---
title: Memory Management
impact: HIGH
impactDescription: Leaks, retention, object copying, and unbounded collections
tags: review, code-review, code-review-performance, perf-memory-management
---

## Rule

Review memory management concerns and flag concrete defects with evidence from the diff and any directly coupled files.

## What to Review

- Look for changes that worsen memory management or hide existing risk.
- Tie the finding to a concrete failure mode or maintenance cost implied by leaks, retention, object copying, and unbounded collections.
- Suggest the smallest concrete fix that addresses the issue without broad refactoring unless the trade-off is explicit.

## Incorrect

```text
A diff introduces memory management risk, but the review only says 'this feels wrong' without explaining the failure mode.
```

## Correct

```text
The review identifies the exact path, explains why memory management is risky here, and recommends a concrete fix.
```

## Notes

- Mark `cross_lens_candidate=true` when this issue should trigger another specialized review pass.
- Mark `tradeoff_required=true` only when the fix genuinely changes design, ergonomics, or runtime behavior.

