---
title: Require Evidence-First Summaries
impact: HIGH
impactDescription: Evidence keeps review output auditable and falsifiable
tags: evidence, summaries, review, audit
---

# Require Evidence-First Summaries

## Why it matters

Verbose process theater hides the few facts that reviewers actually need. Evidence-first summaries keep the output short, testable, and easy to challenge.

## Rule

Require final output to include files changed, commands run, validation results, tests added or updated, and unresolved risks. Do not substitute ritual narration for evidence.

## Bad

```markdown
I used deep reasoning, five review loops, and advanced reflection to ensure quality.
```

## Better

```markdown
Changed:
- `src/Service.php`
- `tests/ServiceTest.php`
```

## Best

```markdown
Changed:
- `src/Payments/Gateway.php`
- `tests/Payments/GatewayTest.php`

Validation:
- `composer test -- --filter GatewayTest` ✅
- `composer phpstan` ✅

Risk:
- Live provider credentials were unavailable for an integration check.
```

## Exceptions / trade-offs

Exploratory or planning-only tasks may not have file changes or commands to report. In that case, report the conclusions and open questions instead of fabricating execution evidence.

## Related topics

- [validation-contracts.md](validation-contracts.md) — evidence should map to explicit commands
- [machine-readable-first.md](machine-readable-first.md) — summaries should preserve underlying evidence
