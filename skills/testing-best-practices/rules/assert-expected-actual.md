---
title: Expected vs Actual Order
priority: HIGH
category: Assertions
---

# Expected vs Actual Order

Follow the convention of placing expected values first in assertions for consistent, readable error messages.

## Bad Example

```typescript
// Inconsistent or reversed order causes confusing error messages
describe('Calculator', () => {
  test('adds numbers', () => {
    const result = calculator.add(2, 3);

    // Reversed: actual first, expected second
    expect(5).toBe(result);
  });

  test('multiplies numbers', () => {
    // Mixed conventions in same test
    expect(calculator.multiply(3, 4)).toBe(12);  // Correct
    expect(24).toBe(calculator.multiply(4, 6)); // Reversed
  });
});

// In assertion libraries with assertEquals(expected, actual):
describe('StringUtils', () => {
  test('capitalizes string', () => {
    const result = capitalize('hello');

    // Wrong order - confusing error message
    assertEquals(result, 'Hello'); // Error: expected "Hello" but got "hello"
                                   // Actually means the opposite!
  });
});
```

## Good Example

```typescript
// Consistent order: actual value being tested first
describe('Calculator', () => {
  test('adds numbers correctly', () => {
    const result = calculator.add(2, 3);

    // Jest/Vitest convention: expect(actual).toBe(expected)
    expect(result).toBe(5);
  });

  test('multiplies numbers correctly', () => {
    expect(calculator.multiply(3, 4)).toBe(12);
    expect(calculator.multiply(4, 6)).toBe(24);
  });

  test('divides numbers correctly', () => {
    const result = calculator.divide(10, 2);

    expect(result).toBe(5);
  });
});

describe('StringUtils', () => {
  test('capitalizes first letter', () => {
    const result = capitalize('hello');

    // Clear: we're testing that result equals 'Hello'
    expect(result).toBe('Hello');
  });

  test('handles already capitalized string', () => {
    expect(capitalize('Hello')).toBe('Hello');
  });
});

// For libraries with assertEquals(expected, actual):
describe('StringUtils with assertEquals', () => {
  test('capitalizes string', () => {
    const result = capitalize('hello');

    // Correct order for assertEquals
    assertEquals('Hello', result);
    // Error message: expected "Hello" but got "hello" - makes sense!
  });
});

// Object comparison follows same principle
describe('UserFactory', () => {
  test('creates user with defaults', () => {
    const user = UserFactory.create({ name: 'John' });

    // Actual object tested against expected shape
    expect(user).toMatchObject({
      name: 'John',
      role: 'user',
      isActive: true
    });
  });

  test('overrides defaults when specified', () => {
    const user = UserFactory.create({ name: 'Admin', role: 'admin' });

    expect(user).toEqual(expect.objectContaining({
      name: 'Admin',
      role: 'admin'
    }));
  });
});
```

## Why

Consistent expected/actual ordering is important for several reasons:

1. **Clear error messages**: When tests fail, you immediately know what was expected vs received
2. **Readability**: Consistent patterns make tests easier to scan and understand
3. **Convention alignment**: Most modern frameworks use `expect(actual).toBe(expected)`
4. **Code review**: Reviewers can quickly spot assertion issues
5. **Debugging efficiency**: No mental translation needed when reading failures

Framework conventions:
- **Jest/Vitest**: `expect(actual).toBe(expected)`
- **Chai**: `expect(actual).to.equal(expected)`
- **JUnit**: `assertEquals(expected, actual)`
- **NUnit**: `Assert.AreEqual(expected, actual)`
- **pytest**: `assert actual == expected`

Always follow your framework's convention and be consistent throughout the codebase.
