---
title: Bind Work to Exact Validation Commands
impact: HIGH
impactDescription: Validation contracts turn vague quality claims into executable gates
tags: validation, commands, ci, stop-on-failure
---

# Bind Work to Exact Validation Commands

## Why it matters

Telling an agent to "run checks" leaves too much room for guessing. Exact commands, order, and failure behavior make the result reproducible for both humans and CI.

## Rule

List the exact validation commands, the order to run them, and the stop-on-failure behavior. If the repository does not define the command, the agent must not invent one silently.

## Bad

```markdown
Validate everything before finishing.
```

## Better

```markdown
- Run `composer test`
- Run `composer phpstan`
```

## Best

```markdown
## Validation contract
- Run `composer test -- --filter PaymentGatewayTest`
- Run `composer phpstan`
- Run `composer cs:check`
- Stop on the first failing command
- Report the failing command verbatim
```

## Exceptions / trade-offs

Documentation-only changes may not need the full suite. The repository should still say which checks are optional or skipped for that class of change.

## Related topics

- [machine-readable-first.md](machine-readable-first.md) — prefer deterministic output from validation tools
- [evidence-output.md](evidence-output.md) — summarize the actual commands and results
