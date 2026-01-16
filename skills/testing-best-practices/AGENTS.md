# Testing Best Practices - Agent Guide

## Overview

This skill provides comprehensive testing best practices covering unit testing, integration testing, and test-driven development (TDD). The skill helps AI agents write better tests, review test code, and guide developers toward reliable, maintainable test suites.

## When to Use This Skill

Activate this skill when:
- Writing new unit or integration tests
- Reviewing existing test code
- Refactoring test suites
- Setting up testing infrastructure
- Teaching testing patterns
- Debugging flaky or brittle tests
- Implementing TDD workflows

## Skill Structure

### 7 Rule Categories

1. **Test Structure** (CRITICAL) - 6 rules
   - AAA pattern organization
   - Descriptive test names
   - Proper describe/it hierarchy
   - Setup and teardown hooks
   - Given-When-Then for BDD
   - Single assertion per test

2. **Test Isolation** (CRITICAL) - 6 rules
   - Independent test execution
   - No shared mutable state
   - Deterministic results
   - No order dependencies
   - Proper cleanup
   - Test doubles for isolation

3. **Assertions** (HIGH) - 5 rules
   - Specific assertions
   - Meaningful error messages
   - Expected vs actual order
   - No magic numbers
   - Custom matchers

4. **Test Data** (HIGH) - 6 rules
   - Factory pattern usage
   - Builder pattern for complex data
   - Faker libraries for realism
   - Minimal test data
   - Realistic edge cases
   - Fixture management

5. **Mocking & Test Doubles** (MEDIUM) - 0 rules currently
   - To be added in future updates

6. **Coverage Strategy** (MEDIUM) - 0 rules currently
   - To be added in future updates

7. **Test Performance** (LOW) - 0 rules currently
   - To be added in future updates

## Rule Format

Each rule follows this structure:

```markdown
---
title: Rule Name
priority: CRITICAL|HIGH|MEDIUM|LOW
category: Test Structure|Test Isolation|Assertions|Test Data
---

# Rule Name

Brief description.

## Bad Example
Code showing antipattern

## Good Example
Code showing best practice

## Why
Benefits and rationale
```

## How Agents Should Apply These Rules

### 1. Writing New Tests

When writing tests, agents should:

1. Start with the AAA (Arrange-Act-Assert) pattern
2. Use descriptive test names that explain the scenario
3. Create minimal but realistic test data using factories
4. Use specific assertions with clear messages
5. Ensure tests are isolated and independent
6. Clean up resources in teardown hooks

Example workflow:
```
1. Read struct-aaa-pattern.md
2. Read struct-descriptive-names.md
3. Read data-factories.md or data-minimal.md
4. Read assert-specific.md
5. Apply patterns while writing
```

### 2. Reviewing Test Code

When reviewing tests, agents should check for:

- **Structure issues**: Mixed arrange/act/assert, unclear names
- **Isolation issues**: Shared state, order dependencies
- **Assertion issues**: Generic assertions, missing messages
- **Data issues**: Overly complex or unrealistic test data

Output format for reviews:
```
file:line - [category] Description of issue
```

Example:
```
tests/user.test.ts:15 - [struct] Missing AAA separation
tests/order.test.ts:42 - [iso] Test depends on previous test's state
tests/cart.test.ts:28 - [assert] Use toHaveLength instead of toBe
```

### 3. Refactoring Tests

When refactoring tests, prioritize:

1. **CRITICAL** issues first (structure, isolation)
2. **HIGH** issues next (assertions, test data)
3. **MEDIUM** and **LOW** as time permits

### 4. Teaching Patterns

When explaining testing concepts:

1. Reference specific rule files
2. Show both bad and good examples
3. Explain the "why" behind each pattern
4. Suggest tools that help (Jest, Vitest, Testing Library)

## Testing Frameworks

This skill applies to multiple testing frameworks:

### JavaScript/TypeScript
- **Jest**: Primary examples use Jest syntax
- **Vitest**: Compatible with most patterns (similar API)
- **Testing Library**: For React/Vue/etc component testing

### Key Differences
- Jest uses `jest.fn()` for mocks
- Vitest uses `vi.fn()` for mocks
- Testing Library emphasizes user-centric queries

## Common Patterns

### Factory Pattern
```typescript
const UserFactory = {
  create(overrides = {}) {
    return {
      id: generateId(),
      name: 'Test User',
      email: 'test@example.com',
      ...overrides
    };
  }
};
```

### Builder Pattern
```typescript
class OrderBuilder {
  private order = { /* defaults */ };

  withItems(items) {
    this.order.items = items;
    return this;
  }

  build() {
    return this.order;
  }
}
```

### Test Doubles
```typescript
const mockService = {
  save: jest.fn().mockResolvedValue(true),
  findById: jest.fn()
};
```

## Anti-Patterns to Watch For

1. **Test interdependence**: Tests that only pass when run in sequence
2. **Overly complex test data**: Objects with unnecessary properties
3. **Generic assertions**: `expect(x).toBeTruthy()` instead of specific checks
4. **Missing cleanup**: Resources not freed, state not reset
5. **Magic numbers**: Unexplained literals in assertions
6. **Flaky tests**: Non-deterministic due to time, randomness, or race conditions

## Output Formats

### Code Generation
When generating tests, use:
- TypeScript for type safety examples
- Jest/Vitest syntax
- AAA pattern comments
- Descriptive test names

### Code Reviews
Use the format: `file:line - [category] message`

### Explanations
Structure as:
1. Problem description
2. Example of antipattern
3. Example of best practice
4. Rationale (why it matters)

## Integration with Other Skills

This skill complements:
- **TypeScript patterns**: Type-safe test data
- **React patterns**: Component testing
- **API design**: Integration test patterns
- **Database patterns**: Test database setup

## References

Core resources embedded in this skill:
- Jest documentation (jestjs.io)
- Vitest documentation (vitest.dev)
- Testing Library (testing-library.com)
- Kent C. Dodds' testing principles
- TDD by Kent Beck
- Working Effectively with Legacy Code by Michael Feathers

## Limitations

This skill does NOT cover:
- E2E testing frameworks (Cypress, Playwright)
- Performance/load testing
- Security testing
- Visual regression testing
- Mobile-specific testing

These may be separate skills.

## Version Information

- **Version**: 1.0.0
- **Rules**: 23 across 7 categories (4 categories populated)
- **Last Updated**: 2026-01-17
- **License**: MIT

## Future Enhancements

Planned additions:
- Mocking & Test Doubles rules (5-7 rules)
- Coverage Strategy rules (4-6 rules)
- Test Performance rules (3-5 rules)
- E2E testing guidance
- Visual testing patterns
- API contract testing
