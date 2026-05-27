---
name: php-best-practices
description: PHP 8.x strict typing, modern language features, type safety, RCA + Kanban workflow, PHPStan + php-cs-fixer validation, value objects, no-magic design, and legacy migration. 56 rules across 9 sections. Use when reviewing PHP code, checking type safety, auditing code quality, or ensuring PHP best practices. Triggers on "review PHP", "check PHP code", "audit PHP", "PHP best practices", or "PHP static analysis".
license: MIT
metadata:
  author: php-community
  version: "3.0.0"
  phpVersion: "8.0 - 8.5"
---

# PHP Best Practices

Modern PHP 8.x patterns, PSR standards, strict type system, SOLID principles, security, performance, value objects, no-magic design, PHPStan PHPDoc, legacy migration, and a mandatory static-analysis tooling loop. Contains **56 rules across 9 sections** for writing clean, maintainable, analyzable PHP code.

> **Core philosophy:** Prefer explicit, analyzable, intention-revealing code with strict contracts and safe defaults. Reject magic, duplication, vague naming, and hidden failure modes.

## Metadata

- **Version:** 3.0.0
- **PHP Version:** 8.0 - 8.5
- **Rule Count:** 56 rules across 9 sections
- **License:** MIT

## Step 1: Detect PHP Version

**Always check the project's PHP version before giving any advice.** Features vary significantly across 8.0 - 8.5. Never suggest syntax that doesn't exist in the project's version.
```json
{ "require": { "php": "^8.1" } }   // -> 8.1 rules and below
{ "require": { "php": "^8.3" } }   // -> 8.3 rules and below
{ "require": { "php": ">=8.4" } }  // -> 8.4 rules and below
```

Also check the runtime version:
```bash
php -v   # e.g. PHP 8.3.12
```

### Feature Availability by Version

| Feature | Version | Rule Prefix |
|---------|---------|-------------|
| Union types, match, nullsafe, named args, constructor promotion, attributes | 8.0+ | `type-`, `modern-` |
| Enums, readonly properties, intersection types, first-class callables, never, fibers | 8.1+ | `modern-` |
| Readonly classes, DNF types, true/false/null standalone types | 8.2+ | `modern-` |
| Typed class constants, `#[\Override]`, `json_validate()` | 8.3+ | `modern-` |
| Property hooks, asymmetric visibility, `#[\Deprecated]`, `new` without parens | 8.4+ | `modern-` |
| Pipe operator `|>` | 8.5+ | `modern-` |

**Only suggest features available in the detected version.** If the user asks about upgrading or newer features, mention what becomes available at each version.

## When to Apply

Reference these guidelines when:
- Writing or reviewing PHP code
- Implementing classes and interfaces
- Using PHP 8.x modern features
- Ensuring type safety
- Following PSR standards
- Applying design patterns

## Rule Categories by Priority

| Priority | Category | Impact | Prefix | Rules |
|----------|----------|--------|--------|-------|
| 1 | Types | CRITICAL | `type-` | 9 |
| 2 | Modern PHP | CRITICAL | `modern-` | 16 |
| 3 | Error Handling | HIGH | `error-` | 5 |
| 4 | Security | CRITICAL | `sec-` | 5 |
| 5 | Performance | MEDIUM | `perf-` | 5 |
| 6 | SOLID / Design | HIGH | `solid-`, `design-` | 7 |
| 7 | PSR / Structure | HIGH | `psr-` | 6 |
| 8 | Tooling / Static Analysis | CRITICAL | `tooling-`, `phpstan-` | 2 |
| 9 | Legacy Migration | HIGH | `legacy-` | 1 |

## Quick Reference

### 1. Types (CRITICAL) — 9 rules

- `type-strict-mode` - Declare strict types in every file
- `type-parameter-types` - Type all parameters
- `type-return-types` - Always declare return types
- `type-property-types` - Type class properties
- `type-nullable-types` - Handle nullable types properly
- `type-union-types` - Use union types effectively
- `type-intersection-types` - Use intersection types
- `type-void-never` - Use void/never for appropriate return types
- `type-mixed-avoid` - Avoid mixed type when possible

### 2. Modern PHP (CRITICAL) — 16 rules

**8.0+:**
- `modern-constructor-promotion` - Constructor property promotion
- `modern-match-expression` - Match over switch
- `modern-named-arguments` - Named arguments for clarity
- `modern-nullsafe-operator` - Nullsafe operator (?->)
- `modern-attributes` - Attributes for metadata

**8.1+:**
- `modern-enums` - Enums instead of constants
- `modern-enums-methods` - Enums with methods and interfaces
- `modern-readonly-properties` - Readonly for immutable data
- `modern-first-class-callables` - First-class callable syntax
- `modern-arrow-functions` - Arrow functions (7.4+, pairs well with 8.1 features)

**8.2+:**
- `modern-readonly-classes` - Readonly classes

**8.3+:**
- `modern-typed-constants` - Typed class constants (`const string NAME = 'foo'`)
- `modern-override-attribute` - `#[\Override]` to catch parent method typos

**8.4+:**
- `modern-property-hooks` - Property hooks replacing getters/setters
- `modern-asymmetric-visibility` - `public private(set)` for controlled access

**8.5+:**
- `modern-pipe-operator` - Pipe operator (`|>`) for functional chaining (deployment-dependent)

### 3. Error Handling (HIGH) — 5 rules

- `error-try-catch-specific` - Catch specific exceptions, not generic \Exception
- `error-custom-exceptions` - Create specific exceptions for different errors
- `error-exception-hierarchy` - Organize exceptions into meaningful hierarchy
- `error-finally-cleanup` - Use finally for guaranteed resource cleanup
- `error-never-suppress` - Never use @ error suppression operator

### 4. Security (CRITICAL) — 5 rules

- `sec-input-validation` - Validate and sanitize all external input
- `sec-output-escaping` - Escape output based on context (HTML, JS, URL)
- `sec-password-hashing` - Use password_hash/verify, never MD5/SHA1
- `sec-sql-prepared` - Use prepared statements for all SQL queries
- `sec-file-uploads` - Validate file type, size, name; store outside web root

### 5. Performance (MEDIUM) — 5 rules

- `perf-string-functions` - Use native string functions over regex
- `perf-array-functions` - Use native array functions over manual loops
- `perf-generators` - Use generators for large datasets
- `perf-lazy-loading` - Defer expensive operations until needed
- `perf-avoid-globals` - Avoid global variables, use dependency injection

### 6. SOLID / Design (HIGH) — 7 rules

- `solid-srp` - Single Responsibility: one reason to change
- `solid-ocp` - Open/Closed: extend, don't modify
- `solid-lsp` - Liskov Substitution: subtypes must be substitutable
- `solid-isp` - Interface Segregation: small, focused interfaces
- `solid-dip` - Dependency Inversion: depend on abstractions
- `design-value-objects` - Value objects over primitives; self-validating, immutable domain types
- `design-no-magic` - Avoid `__get`/`__set`/`__call`; prefer explicit, typed methods and properties

### 7. PSR / Structure (HIGH) — 6 rules

- `psr-file-structure` - One class per file
- `psr-namespace-usage` - Proper namespace usage
- `psr-4-autoloading` - Follow PSR-4 autoloading
- `psr-naming-classes` - Class naming conventions
- `psr-naming-methods` - Method naming conventions
- `psr-12-coding-style` - Follow PSR-12 coding style

### 8. Tooling / Static Analysis (CRITICAL) — 2 rules

- `tooling-phpstan-cs-fixer` - Mandatory PHPStan + php-cs-fixer loop; CI integration; level guide
- `phpstan-phpdoc` - Generics (`@template`), array shapes, `class-string`, `int<min,max>`, conditional return types

PHPDoc is a precision layer — use it only when native PHP cannot express enough. Inline `@var` is a last resort.
If a stricter contract such as `literal-string`, `class-string<T>`, or `non-empty-string` still expresses the real design, fix the proof gap instead of weakening the contract just to satisfy current inference.

### 9. Legacy Migration (HIGH) — 1 rule

- `legacy-migration` - Incremental modernisation: PHPStan baseline → Rector syntax transforms → type coverage → Strangler Fig isolation

## Problem Definition (First Principles)

Before writing or reviewing any PHP code, define the problem explicitly:

```
🧐 Problem
├── Task: What must the code accomplish?
├── Constraints: PHP version, framework, performance budget, backwards-compat
├── Assumptions: What inputs are guaranteed? What can be null?
├── Edge cases: Empty collections, zero values, concurrent access, missing config
└── Risk spots: Security surface (user input), type boundaries, legacy call sites
```

Use this checklist on every non-trivial implementation:

- [ ] PHP version confirmed (`composer.json` → `require.php`)
- [ ] All external inputs identified and validation strategy defined
- [ ] Nullable paths enumerated — `?Type` vs `Type|null` vs exception
- [ ] Legacy call sites listed if touching existing code
- [ ] PHPStan level of the project confirmed (see `phpstan.neon`)

## RCA — Root Cause Analysis

Apply structured root-cause analysis **before** coding when fixing a bug or addressing a quality violation:

```
🌳 RCA
├── Symptom: What error / test failure / PHPStan violation is observed?
├── Immediate cause: The specific line / type mismatch / missing check
├── Contributing factor: Why was this possible? (missing types, no validation, legacy design)
└── Root cause: What structural gap allowed this? (no value objects, no strict types, missing tests)
```

### Example

```
Symptom:        TypeError in OrderService::create() — string passed where int expected
Immediate cause: $data['user_id'] is never cast; HTTP input is always string
Contributing:   No value object for UserId; array{} shape not annotated
Root cause:     declare(strict_types=1) missing; no PHPStan analysis in CI
Fix:            Add declare(strict_types=1); introduce UserId value object;
                annotate $data shape; add PHPStan to CI at level 5+
```

## Kanban Workflow

Follow this three-phase workflow for every PHP coding task:

### Phase 1 — DEFINE / SCOPE

- Confirm PHP version and available language features
- State the Problem (first principles above)
- Run RCA if fixing an existing issue
- Identify files to create or modify
- List new dependencies (check advisory DB before adding)

### Phase 2 — DESIGN / PLANNING

- Sketch class / interface boundaries
- Choose value objects for all domain primitives
- Define PHPDoc generics and array shapes for public APIs
- Decide final / readonly / abstract modifiers
- Plan exception hierarchy if new error paths are introduced
- Outline PHPStan annotations needed

### Phase 3 — CODING / FEEDBACK

- Write code following all rules below
- Run `vendor/bin/php-cs-fixer fix` — fix style
- Run `vendor/bin/phpstan analyse` — fix type violations
- Re-run until both pass with zero errors
- Update tests; confirm all pass
- Review output against the Problem statement

## Essential Guidelines

For detailed examples and explanations, see the rule files:

- [type-strict-mode.md](rules/type-strict-mode.md) - Strict types declaration
- [modern-constructor-promotion.md](rules/modern-constructor-promotion.md) - Constructor property promotion
- [modern-enums.md](rules/modern-enums.md) - PHP 8.1+ enums with methods
- [solid-srp.md](rules/solid-srp.md) - Single responsibility principle
- [design-value-objects.md](rules/design-value-objects.md) - Value objects over primitives
- [design-no-magic.md](rules/design-no-magic.md) - Avoid magic-heavy design
- [sec-input-validation.md](rules/sec-input-validation.md) - Input validation and sanitization
- [error-try-catch-specific.md](rules/error-try-catch-specific.md) - Specific exception handling
- [perf-generators.md](rules/perf-generators.md) - Generators for large datasets
- [phpstan-phpdoc.md](rules/phpstan-phpdoc.md) - PHPStan generics, array shapes, class-strings
- [tooling-phpstan-cs-fixer.md](rules/tooling-phpstan-cs-fixer.md) - PHPStan + php-cs-fixer loop
- [legacy-migration.md](rules/legacy-migration.md) - Legacy code migration strategy
- [psr-12-coding-style.md](rules/psr-12-coding-style.md) - PSR-12 coding style

### Key Patterns (Quick Reference)

```php
<?php
declare(strict_types=1);

// 8.0+ Constructor promotion + readonly (8.1+)
class User
{
    public function __construct(
        public readonly string $id,
        private string $email,
    ) {}
}

// 8.1+ Enums with methods
enum Status: string
{
    case Active = 'active';
    case Inactive = 'inactive';

    public function label(): string
    {
        return match($this) {
            self::Active => 'Active',
            self::Inactive => 'Inactive',
        };
    }
}

// 8.0+ Match expression
$result = match($status) {
    'pending' => 'Waiting',
    'active' => 'Running',
    default => 'Unknown',
};

// 8.0+ Nullsafe operator
$country = $user?->getAddress()?->getCountry();

// 8.3+ Typed class constants + #[\Override]
class PaymentService extends BaseService
{
    public const string GATEWAY = 'stripe';

    #[\Override]
    public function process(): void { /* ... */ }
}

// 8.4+ Property hooks + asymmetric visibility
class Product
{
    public string $name { set => trim($value); }
    public private(set) float $price;
}

// 8.5+ Pipe operator
$result = $input
    |> trim(...)
    |> strtolower(...)
    |> htmlspecialchars(...);
```

## Response Template

Use this structure when responding to any PHP coding request:

```
🧐 Problem
Task:        [what the code must do]
Constraints: [PHP version, framework, existing interfaces]
Assumptions: [guaranteed inputs, null handling, scope boundaries]
Edge cases:  [empty collections, zero values, concurrent writes, etc.]
Risk spots:  [user input surfaces, legacy call sites, type boundaries]

🌳 RCA  (only for bug fixes / refactors)
Symptom:        [observed error / violation]
Immediate:      [specific line or type mismatch]
Contributing:   [missing types / no value objects / no validation]
Root cause:     [structural gap — strict_types off, no PHPStan, no tests]

📋 Kanban
[x] DEFINE / SCOPE   — PHP version confirmed, problem stated
[x] DESIGN / PLANNING — class boundaries, value objects, PHPDoc shapes
[ ] CODING / FEEDBACK — implementation + php-cs-fixer + phpstan loop
```

**PHP implementation:**

```php
<?php

declare(strict_types=1);

namespace App\YourNamespace;

// … typed, final, readonly-where-appropriate implementation …
```

**Validation:**

```bash
vendor/bin/php-cs-fixer fix && vendor/bin/phpstan analyse
```

```
✅ php-cs-fixer: no violations
✅ phpstan (level 8): no errors
```

When auditing code, output findings in this format:

```
file:line - [category] Description of issue
```

Example:
```
src/Services/UserService.php:15 - [type] Missing return type declaration
src/Models/Order.php:42 - [modern] Use match expression instead of switch
src/Controllers/ApiController.php:28 - [solid] Class has multiple responsibilities
```

## How to Use

Read individual rule files for detailed explanations:

```
rules/type-strict-mode.md
rules/type-return-types.md
rules/modern-constructor-promotion.md
rules/modern-enums.md
rules/modern-match-expression.md
rules/error-try-catch-specific.md
rules/error-custom-exceptions.md
rules/sec-input-validation.md
rules/sec-sql-prepared.md
rules/perf-generators.md
rules/solid-srp.md
rules/solid-dip.md
rules/design-value-objects.md
rules/design-no-magic.md
rules/psr-4-autoloading.md
rules/psr-12-coding-style.md
rules/phpstan-phpdoc.md
rules/tooling-phpstan-cs-fixer.md
rules/legacy-migration.md
```
