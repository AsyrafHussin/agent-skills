
---
title: Transaction Boundary Invariants
impact: CRITICAL
impactDescription: Transactional writes versus external side effects and lifecycle-hook safety
tags: review, code-review, code-review-architecture, arch-transaction-boundaries
---

## Rule

Review transaction boundary invariants concerns and flag concrete defects with evidence from the diff and any directly coupled files.

## What to Review

- Look for changes that worsen transaction boundary invariants or hide existing risk.
- Tie the finding to a concrete failure mode or maintenance cost implied by transactional writes versus external side effects and lifecycle-hook safety.
- Suggest the smallest concrete fix that addresses the issue without broad refactoring unless the trade-off is explicit.

## Incorrect

```text
A diff introduces transaction boundary invariants risk, but the review only says 'this feels wrong' without explaining the failure mode.
```

## Correct

```text
The review identifies the exact path, explains why transaction boundary invariants is risky here, and recommends a concrete fix.
```

## Notes

- Mark `cross_lens_candidate=true` when this issue should trigger another specialized review pass.
- Mark `tradeoff_required=true` only when the fix genuinely changes design, ergonomics, or runtime behavior.
- If the diff touches transactions, migrations, or bulk persistence, inspect affected model hooks and external side effects before approving.
