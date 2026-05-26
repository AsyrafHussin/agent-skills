
---
title: Input Validation
impact: HIGH
impactDescription: Whitelisting, bounds checks, sanitization, and schema validation
tags: review, code-review, code-review-security, sec-input-validation
---

## Rule

Review input validation concerns and flag concrete defects with evidence from the diff and any directly coupled files.

## What to Review

- Look for changes that worsen input validation or hide existing risk.
- Tie the finding to a concrete failure mode or maintenance cost implied by whitelisting, bounds checks, sanitization, and schema validation.
- Suggest the smallest concrete fix that addresses the issue without broad refactoring unless the trade-off is explicit.

## Incorrect

```text
A diff introduces input validation risk, but the review only says 'this feels wrong' without explaining the failure mode.
```

## Correct

```text
The review identifies the exact path, explains why input validation is risky here, and recommends a concrete fix.
```

## Notes

- Mark `cross_lens_candidate=true` when this issue should trigger another specialized review pass.
- Mark `tradeoff_required=true` only when the fix genuinely changes design, ergonomics, or runtime behavior.
- Construct at least one adversarial input and trace whether validation or sanitization rejects it safely.
