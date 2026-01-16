# Specific Assertions

Use the most specific assertion available for clearer tests and better error messages.

## Bad Example

```typescript
// Generic assertions hide intent and produce poor error messages
describe('UserService', () => {
  test('returns user by id', async () => {
    const user = await userService.findById('user-123');

    // Too generic - doesn't express what we're checking
    expect(user !== null).toBe(true);
    expect(user.name !== undefined).toBe(true);
  });

  test('validates email format', () => {
    const result = validateEmail('invalid-email');

    // Loses context about what failed
    expect(result).toBe(false);
  });

  test('returns list of users', async () => {
    const users = await userService.findAll();

    // Doesn't verify structure
    expect(users.length > 0).toBe(true);
  });

  test('user has required properties', () => {
    const user = createUser({ name: 'John', email: 'john@example.com' });

    // Testing everything loosely
    expect(JSON.stringify(user).includes('John')).toBe(true);
  });

  test('throws error for invalid id', async () => {
    try {
      await userService.findById('');
      expect(true).toBe(false); // Force fail if no error
    } catch (error) {
      expect(error).toBeDefined();
    }
  });
});
```

## Good Example

```typescript
// Specific assertions with clear intent
describe('UserService', () => {
  test('returns user by id', async () => {
    const user = await userService.findById('user-123');

    expect(user).not.toBeNull();
    expect(user).toHaveProperty('name');
  });

  test('returns null for non-existent user', async () => {
    const user = await userService.findById('non-existent');

    expect(user).toBeNull();
  });

  test('validates correct email format', () => {
    expect(validateEmail('valid@example.com')).toBe(true);
  });

  test('rejects email without @ symbol', () => {
    expect(validateEmail('invalid-email')).toBe(false);
  });

  test('returns array of users', async () => {
    const users = await userService.findAll();

    expect(users).toBeInstanceOf(Array);
    expect(users).not.toHaveLength(0);
  });

  test('returns users with expected structure', async () => {
    const users = await userService.findAll();

    expect(users).toEqual(
      expect.arrayContaining([
        expect.objectContaining({
          id: expect.any(String),
          name: expect.any(String),
          email: expect.stringContaining('@')
        })
      ])
    );
  });

  test('user contains all required properties', () => {
    const user = createUser({ name: 'John', email: 'john@example.com' });

    expect(user).toMatchObject({
      name: 'John',
      email: 'john@example.com'
    });
  });

  test('user has generated id', () => {
    const user = createUser({ name: 'John', email: 'john@example.com' });

    expect(user.id).toMatch(/^user-[a-z0-9]+$/);
  });

  test('throws ValidationError for empty id', async () => {
    await expect(userService.findById('')).rejects.toThrow(ValidationError);
  });

  test('throws error with descriptive message for empty id', async () => {
    await expect(userService.findById('')).rejects.toThrow('ID cannot be empty');
  });
});
```

## Why

Specific assertions improve test quality in several ways:

1. **Better error messages**: When tests fail, you immediately understand what went wrong
2. **Clear intent**: Anyone reading the test knows exactly what's being verified
3. **Documentation**: Assertions serve as executable specifications
4. **Type safety**: Specific matchers work better with TypeScript
5. **Maintainability**: Easier to update tests when requirements change

Common specific assertions:
- `toBeNull()` / `toBeUndefined()` instead of `toBe(null)`
- `toHaveLength(n)` instead of `.length === n`
- `toContain(item)` for arrays
- `toMatchObject({})` for partial object matching
- `toThrow(ErrorType)` for error checking
- `toHaveBeenCalledWith()` for mock verification
- `toMatch(/regex/)` for string patterns
