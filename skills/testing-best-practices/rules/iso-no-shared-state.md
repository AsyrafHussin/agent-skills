---
title: No Shared Mutable State
priority: CRITICAL
category: Test Isolation
---

# No Shared Mutable State

Avoid sharing mutable state between tests to prevent interference and flakiness.

## Bad Example

```typescript
// Shared mutable state causes test interference
let counter = 0;
const cache: Map<string, User> = new Map();
const users: User[] = [];

describe('UserCache', () => {
  test('caches user on first access', () => {
    const user = { id: ++counter, name: 'Alice' };
    users.push(user);
    cache.set(user.id.toString(), user);

    expect(cache.get('1')).toEqual(user);
  });

  test('retrieves cached user', () => {
    // Relies on cache populated by previous test
    expect(cache.has('1')).toBe(true);
  });

  test('adds second user to cache', () => {
    const user = { id: ++counter, name: 'Bob' };
    users.push(user);
    cache.set(user.id.toString(), user);

    // This assertion depends on test execution order
    expect(users.length).toBe(2);
    expect(counter).toBe(2);
  });
});

// Global mock that persists between tests
jest.mock('./emailService', () => ({
  send: jest.fn()
}));
```

## Good Example

```typescript
// Each test has its own isolated state
describe('UserCache', () => {
  let cache: UserCache;
  let userIdCounter: number;

  beforeEach(() => {
    cache = new UserCache();
    userIdCounter = 0;
  });

  const createUser = (name: string): User => ({
    id: ++userIdCounter,
    name
  });

  test('caches user on first access', () => {
    const user = createUser('Alice');

    cache.set(user);

    expect(cache.get(user.id)).toEqual(user);
  });

  test('returns undefined for uncached user', () => {
    expect(cache.get(999)).toBeUndefined();
  });

  test('stores multiple users independently', () => {
    const alice = createUser('Alice');
    const bob = createUser('Bob');

    cache.set(alice);
    cache.set(bob);

    expect(cache.size()).toBe(2);
    expect(cache.get(alice.id)).toEqual(alice);
    expect(cache.get(bob.id)).toEqual(bob);
  });
});

describe('EmailNotificationService', () => {
  let emailService: jest.Mocked<EmailService>;
  let notificationService: NotificationService;

  beforeEach(() => {
    // Fresh mock for each test
    emailService = {
      send: jest.fn().mockResolvedValue(true)
    };
    notificationService = new NotificationService(emailService);
  });

  test('sends welcome email to new user', async () => {
    await notificationService.welcomeUser('alice@example.com');

    expect(emailService.send).toHaveBeenCalledTimes(1);
  });

  test('sends password reset email', async () => {
    await notificationService.sendPasswordReset('bob@example.com');

    // Not affected by previous test
    expect(emailService.send).toHaveBeenCalledTimes(1);
  });
});
```

## Why

Shared mutable state is one of the most common causes of flaky tests:

1. **Predictable behavior**: Each test starts with known state
2. **No side effects**: One test cannot corrupt another's data
3. **Parallel execution**: Tests can run concurrently without conflicts
4. **Easier debugging**: State is clearly defined within each test
5. **Reliable results**: Same results whether running one test or the full suite

Types of state to watch out for:
- Module-level variables
- Global singletons
- Shared database records
- File system state
- Cached values
- Mock function call counts (always reset mocks)

Use `beforeEach` to create fresh instances and reset any necessary state.
