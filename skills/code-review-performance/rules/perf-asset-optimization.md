
---
title: Asset & Payload Optimization
impact: LOW
impactDescription: Bundles, images, compression, and lazy loading opportunities
tags: review, code-review, code-review-performance, perf-asset-optimization
---

## Rule

Review asset & payload optimization concerns and flag concrete defects with evidence from the diff and any directly coupled files.

## What to Review

- Look for changes that worsen asset & payload optimization or hide existing risk.
- Tie the finding to a concrete failure mode or maintenance cost implied by bundles, images, compression, and lazy loading opportunities.
- Suggest the smallest concrete fix that addresses the issue without broad refactoring unless the trade-off is explicit.

## Incorrect

```text
A diff introduces asset & payload optimization risk, but the review only says 'this feels wrong' without explaining the failure mode.
```

## Correct

```text
The review identifies the exact path, explains why asset & payload optimization is risky here, and recommends a concrete fix.
```

## Notes

- Mark `cross_lens_candidate=true` when this issue should trigger another specialized review pass.
- Mark `tradeoff_required=true` only when the fix genuinely changes design, ergonomics, or runtime behavior.

