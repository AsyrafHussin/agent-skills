# Code Review Architecture

Architecture review lens for design quality, coupling, cohesion, module boundaries, and rollback-safe side effects.

## Overview

This skill provides:
- A repo-structured adaptation of the Pi Ensemble review lens
- Focused review guidance for architecture
- A standardized Must Fix / Observations / Summary output contract
- Adversarial-input discipline to avoid shallow approvals

## Categories

### 1. Coupling & Cohesion (Critical)
Tight coupling, low cohesion, circular dependencies, dependency inversion issues.
### 2. Separation of Concerns (Critical)
Business logic placement, layering, and single-responsibility boundaries.
### 3. Abstraction & Interfaces (High)
Abstraction levels, interface quality, and leaky boundaries.
### 4. Module Boundaries (High)
API contracts, visibility, and boundary enforcement.
### 5. Design Patterns (Medium)
Appropriate pattern use, anti-patterns, and consistency.
### 6. Data Flow (Medium)
Clear data movement, state handling, and persistence separation.
### 7. Extensibility & Maintainability (Medium)
Configurability, duplication, and future change safety.
### 8. Transaction Boundary Invariants (Critical)
Transactional writes versus external side effects and lifecycle-hook safety.

## Usage

- "review architecture"
- "architecture review"
- "module boundaries"
- "Review this diff for architecture issues"

## References

- [Pi Ensemble architecture lens](https://raw.githubusercontent.com/randomm/pi-ensemble/main/skill/code-review-architecture/SKILL.md)
- [Martin Fowler on refactoring](https://refactoring.com/)
