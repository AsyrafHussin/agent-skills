
---
title: Network I/O
impact: HIGH
impactDescription: Batching, pagination, parallelism, timeouts, and unnecessary round trips
tags: review, code-review, code-review-performance, perf-network-io
---

## Rule

Review network i/o concerns and flag concrete defects with evidence from the diff and any directly coupled files.

## What to Review

- Look for changes that worsen network i/o or hide existing risk.
- Tie the finding to a concrete failure mode or maintenance cost implied by batching, pagination, parallelism, timeouts, and unnecessary round trips.
- Suggest the smallest concrete fix that addresses the issue without broad refactoring unless the trade-off is explicit.

## Incorrect

```text
A diff introduces network i/o risk, but the review only says 'this feels wrong' without explaining the failure mode.
```

## Correct

```text
The review identifies the exact path, explains why network i/o is risky here, and recommends a concrete fix.
```

## Notes

- Mark `cross_lens_candidate=true` when this issue should trigger another specialized review pass.
- Mark `tradeoff_required=true` only when the fix genuinely changes design, ergonomics, or runtime behavior.

