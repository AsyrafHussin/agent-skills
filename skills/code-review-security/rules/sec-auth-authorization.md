
---
title: Authentication & Authorization
impact: CRITICAL
impactDescription: Identity checks, session handling, access control, and token safety
tags: review, code-review, code-review-security, sec-auth-authorization
---

## Rule

Review authentication & authorization concerns and flag concrete defects with evidence from the diff and any directly coupled files.

## What to Review

- Look for changes that worsen authentication & authorization or hide existing risk.
- Tie the finding to a concrete failure mode or maintenance cost implied by identity checks, session handling, access control, and token safety.
- Suggest the smallest concrete fix that addresses the issue without broad refactoring unless the trade-off is explicit.

## Incorrect

```text
A diff introduces authentication & authorization risk, but the review only says 'this feels wrong' without explaining the failure mode.
```

## Correct

```text
The review identifies the exact path, explains why authentication & authorization is risky here, and recommends a concrete fix.
```

## Notes

- Mark `cross_lens_candidate=true` when this issue should trigger another specialized review pass.
- Mark `tradeoff_required=true` only when the fix genuinely changes design, ergonomics, or runtime behavior.

