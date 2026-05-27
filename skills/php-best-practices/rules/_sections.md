# Sections

This file defines all sections, their ordering, impact levels, and descriptions.
The section ID (in parentheses) is the filename prefix used to group rules.

---

## 1. Types (type)

**Impact:** CRITICAL
**Description:** Strict type enforcement is the foundation of reliable PHP code. `declare(strict_types=1)`, typed parameters, return types, property types, union and intersection types, nullable discipline, and avoiding `mixed` — these rules make every other rule possible. Type precision is the single highest-leverage investment in PHP quality.

## 2. Modern PHP (modern)

**Impact:** CRITICAL
**Description:** PHP 8.x features that reduce boilerplate, improve code clarity, and reinforce type contracts. Every rule is annotated with its minimum PHP version. Always verify the project's PHP version before suggesting features. Covers: constructor promotion (8.0+), enums (8.1+), readonly properties and classes (8.1+/8.2+), typed constants and `#[\Override]` (8.3+), property hooks and asymmetric visibility (8.4+), and pipe operator (8.5+, deployment-dependent).

## 3. Error Handling (error)

**Impact:** HIGH
**Description:** Catch only what you can handle. Throw when you cannot continue. Use `finally` for cleanup, not flow control. Create custom exceptions when callers need to react differently. Keep exception hierarchies shallow and purposeful. Never suppress errors silently.

## 4. Security (sec)

**Impact:** CRITICAL
**Description:** Validate at boundaries. Escape by output context. Use prepared statements without exception. Hash passwords with `password_hash()`. Treat file uploads as untrusted blobs until proven otherwise. Security rules are non-negotiable regardless of PHP version.

## 5. Performance (perf)

**Impact:** MEDIUM
**Description:** Prefer native array and string functions over manual loops and regex for simple operations. Use generators for streaming large datasets. Avoid globals because they destroy testability and local reasoning, not only for performance. Lazy-load only genuinely expensive dependencies.

## 6. SOLID / Design (solid, design)

**Impact:** HIGH
**Description:** Single responsibility, open for extension, substitutable subtypes, focused interfaces, dependency on abstractions. Prefer explicit, typed value objects over primitive obsession. Avoid magic-heavy design (`__get`, `__call`) that destroys static-analysis visibility. Apply SOLID principles pragmatically — one reason to change means one coherent purpose, not one method per class.

## 7. PSR / Structure (psr)

**Impact:** HIGH
**Description:** PSR-4 autoloading, PSR-12 coding style, file structure discipline, namespace conventions, and naming. One automated style tool enforced by CI eliminates bike-shedding entirely.

## 8. Tooling / Static Analysis (tooling, phpstan)

**Impact:** CRITICAL
**Description:** PHPStan + php-cs-fixer run before every commit and enforced in CI. PHPDoc annotations are a precision layer for static analysis — use them only when native PHP cannot express enough (`@template`, array shapes, `class-string`, `int<min,max>`, conditional return types). Inline `@var` is a last resort. The goal is analyzable code, not annotation decoration.

## 9. Legacy Migration (legacy)

**Impact:** HIGH
**Description:** Structured, incremental strategy for modernising legacy PHP codebases: safety-net first (PHPStan baseline + tests), automated Rector syntax transforms, incremental type coverage file-by-file, and Strangler Fig isolation of legacy modules behind typed interfaces. Never rewrite everything at once.
