# Single Logical Assertion

Each test should verify one specific behavior or logical concept.

## Bad Example

```typescript
// Testing multiple unrelated behaviors
test('user registration', () => {
  const user = registerUser({
    email: 'test@example.com',
    password: 'SecurePass123!',
    name: 'John Doe'
  });

  // Testing too many things at once
  expect(user.id).toBeDefined();
  expect(user.email).toBe('test@example.com');
  expect(user.name).toBe('John Doe');
  expect(user.password).toBeUndefined(); // Password shouldn't be returned
  expect(user.createdAt).toBeInstanceOf(Date);
  expect(user.isActive).toBe(false); // Needs email verification
  expect(user.role).toBe('user');
  expect(sendWelcomeEmail).toHaveBeenCalledWith('test@example.com');
  expect(auditLog.entries).toHaveLength(1);
});
```

## Good Example

```typescript
// Each test verifies one logical concept
describe('registerUser', () => {
  const validInput = {
    email: 'test@example.com',
    password: 'SecurePass123!',
    name: 'John Doe'
  };

  test('returns user with generated id', () => {
    const user = registerUser(validInput);
    expect(user.id).toBeDefined();
  });

  test('stores provided email and name', () => {
    const user = registerUser(validInput);
    expect(user).toMatchObject({
      email: 'test@example.com',
      name: 'John Doe'
    });
  });

  test('does not expose password in returned user object', () => {
    const user = registerUser(validInput);
    expect(user.password).toBeUndefined();
  });

  test('sets user as inactive pending email verification', () => {
    const user = registerUser(validInput);
    expect(user.isActive).toBe(false);
  });

  test('assigns default user role', () => {
    const user = registerUser(validInput);
    expect(user.role).toBe('user');
  });

  test('sends welcome email to registered user', () => {
    registerUser(validInput);
    expect(sendWelcomeEmail).toHaveBeenCalledWith('test@example.com');
  });

  test('creates audit log entry for registration', () => {
    registerUser(validInput);
    expect(auditLog.entries).toHaveLength(1);
  });
});
```

## Why

Single-assertion tests provide several benefits:

1. **Precise failure identification**: When a test fails, you know exactly which behavior broke
2. **Independent verification**: Each behavior is tested in isolation
3. **Better documentation**: Test names clearly map to specific behaviors
4. **Easier maintenance**: Changing one behavior only affects its specific test
5. **Granular feedback**: CI/CD reports show exactly what's working and what's not

Note: "Single assertion" means testing one logical concept. Using multiple `expect` statements is fine when they all verify different aspects of the same concept (like using `toMatchObject` for related properties).
