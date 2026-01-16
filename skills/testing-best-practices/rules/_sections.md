# Rule Sections

## Priority Levels

| Level | Description | When to Apply |
|-------|-------------|---------------|
| CRITICAL | Essential for reliable tests | Always |
| HIGH | Significant impact on quality | Most test suites |
| MEDIUM | Improves test clarity | When refactoring tests |
| LOW | Specialized scenarios | Large-scale applications |

## Section Overview

### Test Structure (CRITICAL)
Fundamental patterns for organizing test code. AAA pattern, descriptive naming, and proper test organization form the foundation of readable, maintainable test suites.

### Test Isolation (CRITICAL)
Ensuring tests run independently without shared state or order dependencies. Critical for reliable CI/CD and parallel execution.

### Assertions (HIGH)
Using specific, meaningful assertions with clear messages. Proper assertions make test failures immediately understandable and debuggable.

### Test Data (HIGH)
Managing test data through factories, builders, and fixtures. Quality test data is minimal yet realistic, making tests both focused and robust.

### Mocking & Test Doubles (MEDIUM)
Strategic use of mocks, stubs, and fakes to isolate units under test. Essential for fast, deterministic unit tests.

### Coverage Strategy (MEDIUM)
Balancing coverage goals with meaningful tests. Focus on edge cases and error paths rather than arbitrary coverage percentages.

### Test Performance (LOW)
Optimizing test execution speed. Important for large suites but shouldn't compromise test quality or clarity.
