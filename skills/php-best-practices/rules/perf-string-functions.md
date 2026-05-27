---
title: Prefer Dedicated String Functions Over Regex for Simple Operations
impact: MEDIUM
impactDescription: Runtime performance and memory efficiency
tags: php, performance, optimization
---

# Prefer Dedicated String Functions Over Regex for Simple Operations

## Why it matters
Using `preg_match` to check whether a string starts with a prefix compiles a regex pattern, runs the PCRE engine, and allocates match data — all for an operation that `str_starts_with` handles with a pointer comparison. The performance gap is real, but the more significant cost is readability: `preg_match('/^https/', $url)` forces the reader to parse regex syntax to understand a trivial check.

## Rule
Use `str_starts_with`, `str_ends_with`, `str_contains`, `str_replace`, `explode`, and similar functions when they express the operation directly. Reserve regex for patterns that genuinely require it: character classes, repetition, capture groups, alternation.

## Bad
```php
<?php

declare(strict_types=1);

// Regex for operations that have direct function equivalents
if (preg_match('/^https/', $url)) { /* ... */ }
if (preg_match('/\.pdf$/', $filename)) { /* ... */ }
if (preg_match('/admin/', $role)) { /* ... */ }
$clean = preg_replace('/  +/', ' ', $text); // collapse multiple spaces
$lower = preg_replace('/[A-Z]/', fn($m) => strtolower($m[0]), $input);
```

## Better
```php
<?php

declare(strict_types=1);

// Correct functions, but strpos used where str_contains is clearer (PHP 8.0+)
if (strpos($url, 'https') === 0) { /* ... */ }
if (substr($filename, -4) === '.pdf') { /* ... */ }
if (strpos($role, 'admin') !== false) { /* ... */ }
```

## Best
```php
<?php

declare(strict_types=1);

// PHP 8.0+ — intent is immediately clear, no pattern parsing required
if (str_starts_with($url, 'https://')) { /* secured endpoint */ }
if (str_ends_with($filename, '.pdf')) { /* PDF document */ }
if (str_contains($role, 'admin')) { /* admin role check */ }

// Simple replacements
$slug     = str_replace(' ', '-', strtolower($title));
$trimmed  = trim($input);
$parts    = explode(',', $csv);
$sentence = implode(', ', $words);

// Substring extraction without regex
$ext = substr($filename, (int) strrpos($filename, '.') + 1);

// Use regex only when the pattern genuinely needs it
$isDate   = (bool) preg_match('/^\d{4}-\d{2}-\d{2}$/', $value);
$isEmail  = filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
$stripped = preg_replace('/[^\p{L}\p{N}\s]/u', '', $text); // Unicode-aware
```

## Exceptions / trade-offs
Regex is the right tool for tokenization, format validation with complex rules, and Unicode-aware character-class operations. `preg_replace_callback` is often cleaner than a manual loop for multi-pattern transformations. The rule is about avoiding regex where simpler alternatives exist, not about avoiding regex entirely.

## Static-analysis notes
PHPStan and Psalm do not flag `preg_match` used for simple prefix/suffix checks; this is a code review and linter concern. A custom PHPStan rule or a Rector migration rule can automate the `strpos`-to-`str_contains` upgrade.

## Version notes
`str_starts_with`, `str_ends_with`, `str_contains` — PHP 8.0+

## Related topics
- [perf-array-functions.md](perf-array-functions.md) — same principle applied to arrays
- [type-strict-mode.md](type-strict-mode.md) — strict types prevent silent string-to-int coercion in string functions
