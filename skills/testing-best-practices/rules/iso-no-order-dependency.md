---
title: No Order Dependency
priority: CRITICAL
category: Test Isolation
---

# No Order Dependency

Tests should pass regardless of the order in which they run.

## Bad Example

```typescript
// Tests that depend on execution order
describe('Database Integration', () => {
  // This test creates data that later tests depend on
  test('inserts user into database', async () => {
    await db.query('INSERT INTO users (id, name) VALUES (1, "Alice")');
    const result = await db.query('SELECT * FROM users WHERE id = 1');
    expect(result.rows[0].name).toBe('Alice');
  });

  // Depends on previous test's insert
  test('updates user in database', async () => {
    await db.query('UPDATE users SET name = "Alicia" WHERE id = 1');
    const result = await db.query('SELECT * FROM users WHERE id = 1');
    expect(result.rows[0].name).toBe('Alicia');
  });

  // Depends on update from previous test
  test('verifies user state', async () => {
    const result = await db.query('SELECT * FROM users WHERE id = 1');
    expect(result.rows[0].name).toBe('Alicia');
  });

  // Must run last or will break other tests
  test('deletes user from database', async () => {
    await db.query('DELETE FROM users WHERE id = 1');
    const result = await db.query('SELECT * FROM users WHERE id = 1');
    expect(result.rows.length).toBe(0);
  });
});

// Static counter creates order dependency
let testSequence = 0;

test('first operation', () => {
  testSequence = 1;
  expect(processStep(testSequence)).toBe('step-1');
});

test('second operation', () => {
  // Fails if run before 'first operation'
  expect(testSequence).toBe(1);
  testSequence = 2;
  expect(processStep(testSequence)).toBe('step-2');
});
```

## Good Example

```typescript
// Each test is self-contained
describe('Database Integration', () => {
  beforeEach(async () => {
    await db.query('DELETE FROM users');
  });

  test('inserts user into database', async () => {
    await db.query('INSERT INTO users (id, name) VALUES (1, "Alice")');

    const result = await db.query('SELECT * FROM users WHERE id = 1');

    expect(result.rows[0].name).toBe('Alice');
  });

  test('updates existing user in database', async () => {
    // Arrange: Create the user this test needs
    await db.query('INSERT INTO users (id, name) VALUES (1, "Alice")');

    // Act
    await db.query('UPDATE users SET name = "Alicia" WHERE id = 1');

    // Assert
    const result = await db.query('SELECT * FROM users WHERE id = 1');
    expect(result.rows[0].name).toBe('Alicia');
  });

  test('deletes user from database', async () => {
    // Arrange: Create the user this test needs
    await db.query('INSERT INTO users (id, name) VALUES (1, "Alice")');

    // Act
    await db.query('DELETE FROM users WHERE id = 1');

    // Assert
    const result = await db.query('SELECT * FROM users WHERE id = 1');
    expect(result.rows.length).toBe(0);
  });
});

// Each test is independent
describe('Step Processing', () => {
  test('processes first step correctly', () => {
    expect(processStep(1)).toBe('step-1');
  });

  test('processes second step correctly', () => {
    expect(processStep(2)).toBe('step-2');
  });

  test('processes steps in sequence', () => {
    // Tests the sequence concept in isolation
    const sequence = [1, 2, 3];
    const results = sequence.map(step => processStep(step));

    expect(results).toEqual(['step-1', 'step-2', 'step-3']);
  });
});
```

## Why

Order-independent tests are essential for modern testing:

1. **Random ordering**: Many test runners shuffle test order to catch dependencies
2. **Parallel execution**: Tests running in parallel have no guaranteed order
3. **Selective running**: Developers often run single tests or subsets
4. **Watch mode**: Only changed tests may re-run during development
5. **Debugging**: Isolated tests are easier to run and debug individually

How to verify order independence:
- Run tests in random order (`jest --randomize`)
- Run tests in reverse order
- Run individual tests in isolation
- Run tests in parallel

Signs of order dependency:
- Tests pass in suite but fail in isolation
- Flaky tests that sometimes fail
- Tests that only work when run after specific other tests
