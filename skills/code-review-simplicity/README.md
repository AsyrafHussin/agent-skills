# Code Review Simplicity

Simplicity review lens for readability, cognitive load, unnecessary abstraction, duplication, and dead-range logic.

## Overview

This skill provides:
- A repo-structured adaptation of the Pi Ensemble review lens
- Focused review guidance for simplicity
- A standardized Must Fix / Observations / Summary output contract
- Adversarial-input discipline to avoid shallow approvals

## Categories

### 1. Readability & Clarity (High)
Naming, magic values, nesting, boolean complexity, and long functions.
### 2. Cognitive Load (High)
Parameter count, responsibility load, inheritance depth, and mental model cost.
### 3. Unnecessary Complexity (Critical)
Over-engineering, premature abstraction, and patterns without value.
### 4. Colliding or Redundant Bounds (Critical)
Bound-range collapse, dead-range logic, and constant-output paths.
### 5. Code Duplication (Medium)
Copy-paste logic and repeated structures that should converge.
### 6. Documentation & Comments (Low)
Missing, outdated, or contradictory explanation.
### 7. Naming Conventions (Medium)
Non-descriptive, inconsistent, or misleading names.
### 8. Testing & Debugging Simplicity (Low)
Hidden side effects, difficult debugging, and hard-to-test design.

## Usage

- "simplicity review"
- "reduce complexity"
- "readability review"
- "Review this diff for simplicity and maintainability issues"

## References

- [Pi Ensemble simplicity lens](https://raw.githubusercontent.com/randomm/pi-ensemble/main/skill/code-review-simplicity/SKILL.md)
- [Refactoring Guru code smells](https://refactoring.guru/refactoring/smells)
