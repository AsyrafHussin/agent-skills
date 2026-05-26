
---
title: Rule Title
impact: CRITICAL|HIGH|MEDIUM|LOW
impactDescription: One-sentence explanation of why the rule matters
tags: review, lens, code-review
---

## Rule

State what to look for and why it matters.

## What to Review

- Identify the relevant changed path or cross-file dependency.
- Explain the failure mode, not just the surface symptom.
- Propose the narrowest concrete fix.

## Incorrect

```text
Describe the risky pattern or pseudo-code that should be avoided.
```

## Correct

```text
Describe the safer or clearer pattern reviewers should recommend.
```

## Notes

- Mention when this rule should produce a cross-lens candidate.
- Mention when fixing it requires a trade-off.
