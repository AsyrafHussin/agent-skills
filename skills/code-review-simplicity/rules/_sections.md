# Sections

This file defines the section ordering, severity, and summary used by this review lens.

---

## 1. Readability & Clarity (simp-readability-clarity)

**Impact:** HIGH
**Description:** Naming, magic values, nesting, boolean complexity, and long functions.

## 2. Cognitive Load (simp-cognitive-load)

**Impact:** HIGH
**Description:** Parameter count, responsibility load, inheritance depth, and mental model cost.

## 3. Unnecessary Complexity (simp-unnecessary-complexity)

**Impact:** CRITICAL
**Description:** Over-engineering, premature abstraction, and patterns without value.

## 4. Colliding or Redundant Bounds (simp-colliding-bounds)

**Impact:** CRITICAL
**Description:** Bound-range collapse, dead-range logic, and constant-output paths.

## 5. Code Duplication (simp-code-duplication)

**Impact:** MEDIUM
**Description:** Copy-paste logic and repeated structures that should converge.

## 6. Documentation & Comments (simp-documentation-comments)

**Impact:** LOW
**Description:** Missing, outdated, or contradictory explanation.

## 7. Naming Conventions (simp-naming-conventions)

**Impact:** MEDIUM
**Description:** Non-descriptive, inconsistent, or misleading names.

## 8. Testing & Debugging Simplicity (simp-testing-debugging)

**Impact:** LOW
**Description:** Hidden side effects, difficult debugging, and hard-to-test design.
