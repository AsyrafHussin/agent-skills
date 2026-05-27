---
title: Pipe Operator
impact: CRITICAL
impactDescription: Modern PHP 8.x language features
tags: php, modern-php, php8
---

# Pipe Operator

## Why it matters
> **⚠ PHP 8.5+ only.** PHP 8.5 is not yet universally available. Only adopt the pipe operator if your entire deployment pipeline — dev, CI, staging, and production — runs PHP 8.5. Enabling it prematurely on older runtimes causes a parse error.

Deeply nested function calls must be read inside-out, which is the opposite of how the transformation actually flows. Temporary variables reduce nesting but litter the scope. The `|>` pipe operator expresses a linear data transformation in the order it happens, left-to-right, top-to-bottom.

## Rule
Use `|>` for pure, linear data transformation chains where readability genuinely improves. Do not use it across codebases that must support PHP < 8.5.

## Bad
```php
<?php

declare(strict_types=1);

// Read inside-out to understand the flow: trim → lower → escape
$result = htmlspecialchars(strtolower(trim($input)));
```

## Better
```php
<?php

declare(strict_types=1);

// Temporary variables clarify order but clutter scope
$trimmed  = trim($input);
$lowered  = strtolower($trimmed);
$result   = htmlspecialchars($lowered);
```

## Best
```php
<?php

declare(strict_types=1);

// PHP 8.5+ — requires deployment on PHP 8.5+
$result = $input
    |> trim(...)
    |> strtolower(...)
    |> htmlspecialchars(...);

// Multi-argument steps use arrow functions (pipe always passes one argument)
$slug = $title
    |> trim(...)
    |> strtolower(...)
    |> (fn(string $s) => preg_replace('/[^a-z0-9]+/', '-', $s))
    |> (fn(string $s) => trim($s, '-'));
```

## Exceptions / trade-offs
- **Deployment gate**: Do not use `|>` unless PHP 8.5+ is guaranteed in every environment. There is no polyfill — the operator simply does not parse on earlier versions.
- The operator passes exactly one argument (the left-hand value) to the right-hand callable. For functions with multiple required arguments, wrap them in closures or arrow functions.
- Do not use `|>` for chains that include side effects (logging, I/O) — those should remain explicit sequential statements.

## Static-analysis notes
PHPStan and Psalm support for `|>` is expected to follow the PHP 8.5 release. IDE support will require plugin updates. Until tooling catches up, some editors may show false-positive parse warnings.

## Version notes
`PHP 8.5+`

## Related topics
- [modern-arrow-functions.md](modern-arrow-functions.md) — arrow functions compose naturally with pipe
- [modern-first-class-callables.md](modern-first-class-callables.md) — `trim(...)` syntax is the idiomatic right-hand operand
