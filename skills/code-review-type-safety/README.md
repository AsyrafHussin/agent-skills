# Code Review Type Safety

Type-safety review lens for public API typing, runtime validation, unsafe casts, and generic discipline.

## Overview

This skill provides:
- A repo-structured adaptation of the Pi Ensemble review lens
- Focused review guidance for type safety
- A standardized Must Fix / Observations / Summary output contract
- Adversarial-input discipline to avoid shallow approvals

## Categories

### 1. Type Coverage (High)
Typed public APIs, critical internals, and avoiding implicit any gaps.
### 2. Type Correctness (Critical)
Annotations, assertions, and guards that match real behavior.
### 3. Type Safety (Critical)
Unsafe casts, unknown data, discriminated unions, and runtime checks.
### 4. Generic Discipline (Medium)
Constraints, parameter bounds, and avoiding generic overuse.

## Usage

- "type safety review"
- "review typings"
- "schema mismatch"
- "Review this diff for type-safety issues"

## References

- [Pi Ensemble type-safety lens](https://raw.githubusercontent.com/randomm/pi-ensemble/main/skill/code-review-type-safety/SKILL.md)
- [TypeScript handbook](https://www.typescriptlang.org/docs/)
