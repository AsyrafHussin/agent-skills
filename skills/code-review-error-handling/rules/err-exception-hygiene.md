
---
title: Exception & Error Hygiene
impact: CRITICAL
impactDescription: Overbroad catches, swallowed errors, and lost causal context
tags: review, code-review, code-review-error-handling, err-exception-hygiene
---

## Rule

Review exception & error hygiene concerns and flag concrete defects with evidence from the diff and any directly coupled files.

## What to Review

- Look for changes that worsen exception & error hygiene or hide existing risk.
- Tie the finding to a concrete failure mode or maintenance cost implied by overbroad catches, swallowed errors, and lost causal context.
- Suggest the smallest concrete fix that addresses the issue without broad refactoring unless the trade-off is explicit.

## Incorrect

```text
A diff introduces exception & error hygiene risk, but the review only says 'this feels wrong' without explaining the failure mode.
```

## Correct

```text
The review identifies the exact path, explains why exception & error hygiene is risky here, and recommends a concrete fix.
```

## Notes

- Mark `cross_lens_candidate=true` when this issue should trigger another specialized review pass.
- Mark `tradeoff_required=true` only when the fix genuinely changes design, ergonomics, or runtime behavior.

