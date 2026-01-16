---
title: Setup and Teardown
priority: CRITICAL
category: Test Structure
---

# Setup and Teardown

Use setup and teardown hooks appropriately to prepare and clean up test environments.

## Bad Example

```typescript
// Duplicated setup and missing cleanup
describe('UserService', () => {
  test('creates user successfully', async () => {
    const db = await Database.connect();
    const userService = new UserService(db);

    const user = await userService.create({ name: 'John' });

    expect(user.name).toBe('John');
    // Missing cleanup - database connection left open
  });

  test('finds user by id', async () => {
    const db = await Database.connect();
    const userService = new UserService(db);

    const created = await userService.create({ name: 'Jane' });
    const found = await userService.findById(created.id);

    expect(found.name).toBe('Jane');
    // Missing cleanup - test data left in database
  });

  test('updates user name', async () => {
    const db = await Database.connect();
    const userService = new UserService(db);

    const user = await userService.create({ name: 'Bob' });
    await userService.update(user.id, { name: 'Robert' });

    expect(await userService.findById(user.id)).toHaveProperty('name', 'Robert');
  });
});
```

## Good Example

```typescript
// Proper setup and teardown with hooks
describe('UserService', () => {
  let db: Database;
  let userService: UserService;

  // One-time setup for expensive operations
  beforeAll(async () => {
    db = await Database.connect();
  });

  // Per-test setup for service instance
  beforeEach(() => {
    userService = new UserService(db);
  });

  // Clean up test data after each test
  afterEach(async () => {
    await db.query('DELETE FROM users WHERE email LIKE $1', ['%@test.com']);
  });

  // Close resources after all tests
  afterAll(async () => {
    await db.disconnect();
  });

  test('creates user successfully', async () => {
    const user = await userService.create({
      name: 'John',
      email: 'john@test.com'
    });

    expect(user.name).toBe('John');
  });

  test('finds user by id', async () => {
    const created = await userService.create({
      name: 'Jane',
      email: 'jane@test.com'
    });

    const found = await userService.findById(created.id);

    expect(found.name).toBe('Jane');
  });

  describe('update operations', () => {
    let existingUser: User;

    // Nested setup for specific context
    beforeEach(async () => {
      existingUser = await userService.create({
        name: 'Bob',
        email: 'bob@test.com'
      });
    });

    test('updates user name', async () => {
      await userService.update(existingUser.id, { name: 'Robert' });

      const updated = await userService.findById(existingUser.id);
      expect(updated.name).toBe('Robert');
    });

    test('updates user email', async () => {
      await userService.update(existingUser.id, { email: 'robert@test.com' });

      const updated = await userService.findById(existingUser.id);
      expect(updated.email).toBe('robert@test.com');
    });
  });
});
```

## Why

Proper setup and teardown ensures:

1. **No resource leaks**: Database connections, file handles, and other resources are properly closed
2. **Test isolation**: Each test starts with a clean state
3. **DRY code**: Common setup logic is not duplicated across tests
4. **Reliable tests**: Tests don't fail due to leftover data from previous tests
5. **Performance**: Expensive setup (like database connections) is done once

Guidelines for hooks:
- `beforeAll`: Expensive one-time setup (database connections, server startup)
- `beforeEach`: Per-test setup (fresh instances, test data)
- `afterEach`: Clean up test-specific data
- `afterAll`: Clean up shared resources

Keep setup focused on what the tests actually need. Over-complicated setup is a code smell.
