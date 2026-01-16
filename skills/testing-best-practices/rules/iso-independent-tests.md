---
title: Independent Tests
priority: CRITICAL
category: Test Isolation
---

# Independent Tests

Each test should be completely independent and not rely on other tests.

## Bad Example

```typescript
// Tests depend on each other and share state
describe('ShoppingCart', () => {
  const cart = new ShoppingCart(); // Shared instance!

  test('adds first item to cart', () => {
    cart.addItem({ id: 1, name: 'Apple', price: 1.00 });
    expect(cart.getItemCount()).toBe(1);
  });

  test('adds second item to cart', () => {
    // Depends on previous test having added an item
    cart.addItem({ id: 2, name: 'Banana', price: 0.50 });
    expect(cart.getItemCount()).toBe(2);
  });

  test('calculates total', () => {
    // Depends on both previous tests
    expect(cart.getTotal()).toBe(1.50);
  });

  test('removes item', () => {
    // Depends on all previous tests
    cart.removeItem(1);
    expect(cart.getItemCount()).toBe(1);
    expect(cart.getTotal()).toBe(0.50);
  });
});
```

## Good Example

```typescript
// Each test is independent with its own setup
describe('ShoppingCart', () => {
  let cart: ShoppingCart;

  beforeEach(() => {
    cart = new ShoppingCart(); // Fresh instance for each test
  });

  test('adds item to empty cart', () => {
    cart.addItem({ id: 1, name: 'Apple', price: 1.00 });

    expect(cart.getItemCount()).toBe(1);
  });

  test('adds multiple items to cart', () => {
    cart.addItem({ id: 1, name: 'Apple', price: 1.00 });
    cart.addItem({ id: 2, name: 'Banana', price: 0.50 });

    expect(cart.getItemCount()).toBe(2);
  });

  test('calculates total for multiple items', () => {
    cart.addItem({ id: 1, name: 'Apple', price: 1.00 });
    cart.addItem({ id: 2, name: 'Banana', price: 0.50 });

    expect(cart.getTotal()).toBe(1.50);
  });

  test('removes item from cart with multiple items', () => {
    cart.addItem({ id: 1, name: 'Apple', price: 1.00 });
    cart.addItem({ id: 2, name: 'Banana', price: 0.50 });

    cart.removeItem(1);

    expect(cart.getItemCount()).toBe(1);
    expect(cart.getTotal()).toBe(0.50);
  });

  test('returns zero total for empty cart', () => {
    expect(cart.getTotal()).toBe(0);
  });
});
```

## Why

Independent tests are essential for reliable test suites:

1. **Run in any order**: Tests can be shuffled, parallelized, or run individually
2. **Isolated failures**: When one test fails, it doesn't cascade to other tests
3. **Easier debugging**: Failed tests can be run in isolation to reproduce issues
4. **Parallelization**: Independent tests can run concurrently for faster execution
5. **Maintainability**: Changes to one test don't affect others
6. **Reliable CI/CD**: No flaky failures due to test ordering

Signs of dependent tests:
- Tests that pass individually but fail when run together
- Tests that fail when run in a different order
- Tests that only pass when the full suite runs
- Shared mutable state between tests
