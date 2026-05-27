---
title: PHPStan + php-cs-fixer Validation Loop
impact: CRITICAL
impactDescription: PHPStan and php-cs-fixer static analysis tooling
tags: php, phpstan, static-analysis, tooling
---

# PHPStan + php-cs-fixer Validation Loop

## Why it matters

Without automated tooling in CI, code style becomes a matter of individual habit, type errors are discovered at runtime instead of analysis time, and `declare(strict_types=1)` is something developers remember to add "most of the time." The PHPStan + php-cs-fixer loop converts subjective reviews into objective, automated gates. A type mismatch or missing return type that slips past a distracted reviewer does not slip past PHPStan.

## Rule

Run `php-cs-fixer fix` then `vendor/bin/phpstan analyse` before every commit and enforce both in CI. Never skip either step. Never merge a PR that fails either check.

## Bad

```bash
# No tooling — style and types are "someone's responsibility"
git add .
git commit -m "add user service"
git push
# 3 days later: TypeError in production, inconsistent indentation across 4 devs
```

## Better

```bash
# Manual, ad-hoc — better than nothing but not enforced
vendor/bin/php-cs-fixer fix src/
vendor/bin/phpstan analyse src/
git add .
git commit -m "add user service"
```

## Best

```json
// composer.json — automated loop, same command for every developer
{
    "scripts": {
        "cs":       "php-cs-fixer fix",
        "cs:check": "php-cs-fixer fix --dry-run --diff",
        "stan":     "phpstan analyse",
        "analyse":  ["@cs", "@stan"],
        "test":     "pest",
        "check":    ["@analyse", "@test"]
    }
}
```

```bash
# Developer workflow — one command, no guesswork
composer analyse   # fix style, then verify types
composer check     # full gate: style + types + tests
```

```neon
# phpstan.neon — recommended baseline configuration
parameters:
    level: 8
    paths:
        - src
        - tests
    checkMissingIterableValueType: true
    checkGenericClassInNonGenericObjectType: true
    reportUnmatchedIgnoredErrors: true
```

```php
<?php

// .php-cs-fixer.php — PSR-12 + strict types enforced automatically
$finder = PhpCsFixer\Finder::create()
    ->in([__DIR__ . '/src', __DIR__ . '/tests'])
    ->name('*.php')
    ->ignoreDotFiles(true)
    ->ignoreVCS(true);

return (new PhpCsFixer\Config())
    ->setRiskyAllowed(true)
    ->setRules([
        '@PSR12'                       => true,
        '@PHP81Migration'              => true,
        'strict_param'                 => true,
        'declare_strict_types'         => true,   // auto-adds declare(strict_types=1)
        'array_syntax'                 => ['syntax' => 'short'],
        'ordered_imports'              => ['sort_algorithm' => 'alpha'],
        'no_unused_imports'            => true,
        'trailing_comma_in_multiline'  => true,
        'single_quote'                 => true,
        'no_extra_blank_lines'         => true,
        'binary_operator_spaces'       => true,
    ])
    ->setFinder($finder);
```

```yaml
# .github/workflows/quality.yml — CI enforcement
name: Quality
on: [push, pull_request]
jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: shivammathur/setup-php@v2
        with:
          php-version: '8.3'
          tools: composer, cs2pr
      - run: composer install --no-interaction --prefer-dist
      - name: php-cs-fixer
        run: vendor/bin/php-cs-fixer fix --dry-run --diff --format=checkstyle | cs2pr
      - name: PHPStan
        run: vendor/bin/phpstan analyse --error-format=github
```

## Exceptions / trade-offs

- **Legacy projects**: generate a PHPStan baseline first (`--generate-baseline`) so existing violations are acknowledged without blocking CI immediately. Then fix violations and raise the level incrementally.
- **Generated code** (migrations, compiled templates): exclude from both tools via `.php-cs-fixer.php` `Finder` and `phpstan.neon` `excludePaths`.
- **`@phpstan-ignore-next-line`**: acceptable only for false positives from third-party libraries. Document the reason inline. Never use it to paper over real type errors.

## Static-analysis notes

- **Run order matters**: php-cs-fixer reformats code (adds `declare(strict_types=1)`, adjusts whitespace). Run PHPStan after so it analyses the final file state and reports accurate line numbers.
- **PHPStan level guide**:
  - Level 0: unknown classes, methods, properties
  - Level 3: return type checking
  - Level 5: argument type checking
  - Level 6: native function type checking
  - Level 8: nullable type mismatches
  - Level 9: `mixed` not allowed without explicit annotation
  - **Recommended for new projects**: start at level 8, target level 9
- **`phpstan/phpstan-strict-rules`** adds extra strictness beyond the base levels; recommended for new projects.
- **`phpstan/phpstan-deprecation-rules`** flags calls to deprecated methods — useful during upgrades.

## Related topics

- [phpstan-phpdoc.md](phpstan-phpdoc.md) — precision PHPDoc annotations that PHPStan can verify
- [legacy-migration.md](legacy-migration.md) — using the PHPStan baseline during legacy migration
- [type-strict-mode.md](type-strict-mode.md) — `declare(strict_types=1)` which php-cs-fixer can enforce automatically
- [psr-12-coding-style.md](psr-12-coding-style.md) — PSR-12 style that php-cs-fixer enforces
