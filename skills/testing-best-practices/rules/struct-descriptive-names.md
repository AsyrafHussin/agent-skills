---
title: Descriptive Test Names
priority: CRITICAL
category: Test Structure
---

# Descriptive Test Names

Write test names that clearly describe the scenario being tested and the expected outcome.

## Bad Example

```typescript
// Vague and uninformative names
test('test1', () => {
  expect(validateEmail('test@example.com')).toBe(true);
});

test('email validation', () => {
  expect(validateEmail('invalid')).toBe(false);
});

test('it works', () => {
  const user = createUser({ name: 'John' });
  expect(user.name).toBe('John');
});

test('user test', () => {
  expect(() => createUser({ name: '' })).toThrow();
});
```

## Good Example

```typescript
// Clear, descriptive names following a consistent pattern
describe('validateEmail', () => {
  test('returns true for valid email with standard format', () => {
    expect(validateEmail('user@example.com')).toBe(true);
  });

  test('returns false when email is missing @ symbol', () => {
    expect(validateEmail('userexample.com')).toBe(false);
  });

  test('returns false when email has multiple @ symbols', () => {
    expect(validateEmail('user@@example.com')).toBe(false);
  });
});

describe('createUser', () => {
  test('creates user with provided name', () => {
    const user = createUser({ name: 'John' });
    expect(user.name).toBe('John');
  });

  test('throws ValidationError when name is empty string', () => {
    expect(() => createUser({ name: '' })).toThrow(ValidationError);
  });
});
```

## Why

Descriptive test names serve as documentation and provide immediate insight when tests fail:

1. **Self-documenting**: Test names explain what the code should do without reading the implementation
2. **Faster debugging**: When a test fails, you immediately know what behavior broke
3. **Better test coverage visibility**: Reviewing test names reveals gaps in coverage
4. **Communication**: Team members can understand tested behaviors at a glance

Good naming patterns include:
- `[method/feature] [expected behavior] [under condition]`
- `should [expected behavior] when [condition]`
- `returns [value] for [input description]`
