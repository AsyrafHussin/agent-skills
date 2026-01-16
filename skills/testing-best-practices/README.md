# Testing Best Practices

Unit testing, integration testing, and TDD principles for reliable, maintainable test suites.

## Overview

This skill provides guidance for:
- Test structure and organization
- Test isolation and independence
- Effective assertions
- Test data management
- Mocking strategies
- Coverage goals

## Categories

### 1. Test Structure (Critical)
AAA pattern, descriptive names, one assertion per test.

### 2. Test Isolation (Critical)
Independent tests, no shared state, deterministic results.

### 3. Assertions (High)
Specific assertions, meaningful messages, named constants.

### 4. Test Data (High)
Factories, minimal data, realistic edge cases.

### 5. Mocking (Medium)
Mock boundaries, verify interactions, realistic behavior.

### 6. Coverage (Medium)
Meaningful coverage, edge cases, error scenarios.

## Usage

Ask Claude to:
- "Write tests for this function"
- "Review test coverage"
- "Check test isolation"
- "Improve test structure"

## Key Principles

### AAA Pattern
- **Arrange**: Set up test data
- **Act**: Execute the code
- **Assert**: Verify results

### Test Isolation
- Each test independent
- No shared mutable state
- Clean up after tests
- Run in any order

### Test Pyramid
- Many unit tests (fast)
- Some integration tests
- Few E2E tests (slow)

## References

- [Testing Library](https://testing-library.com/)
- [Jest Documentation](https://jestjs.io/)
- [PHPUnit Documentation](https://phpunit.de/)
