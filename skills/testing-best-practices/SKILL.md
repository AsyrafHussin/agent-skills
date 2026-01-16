---
name: testing-best-practices
description: Unit testing, integration testing, and test-driven development principles. Use when writing tests, reviewing test code, improving test coverage, or setting up testing strategy. Triggers on "write tests", "review tests", "testing best practices", or "TDD".
license: MIT
metadata:
  author: agent-skills
  version: "1.0.0"
---

# Testing Best Practices

Unit testing, integration testing, and TDD principles for reliable, maintainable test suites. Guidelines for writing effective tests that provide confidence and documentation.

## When to Apply

Reference these guidelines when:
- Writing unit tests
- Writing integration tests
- Reviewing test code
- Improving test coverage
- Setting up testing strategy

## Rule Categories by Priority

| Priority | Category | Impact | Prefix |
|----------|----------|--------|--------|
| 1 | Test Structure | CRITICAL | `struct-` |
| 2 | Test Isolation | CRITICAL | `iso-` |
| 3 | Assertions | HIGH | `assert-` |
| 4 | Test Data | HIGH | `data-` |
| 5 | Mocking | MEDIUM | `mock-` |
| 6 | Coverage | MEDIUM | `cov-` |
| 7 | Performance | LOW | `perf-` |

## Quick Reference

### 1. Test Structure (CRITICAL)

- `struct-aaa` - Arrange, Act, Assert pattern
- `struct-naming` - Descriptive test names
- `struct-one-assert` - One logical assertion per test
- `struct-describe-it` - Organized test suites
- `struct-given-when-then` - BDD style when appropriate

### 2. Test Isolation (CRITICAL)

- `iso-independent` - Tests run independently
- `iso-no-shared-state` - No shared mutable state
- `iso-deterministic` - Same result every run
- `iso-no-order-dependency` - Run in any order
- `iso-cleanup` - Clean up after tests

### 3. Assertions (HIGH)

- `assert-specific` - Specific assertions
- `assert-meaningful-messages` - Helpful failure messages
- `assert-expected-first` - Expected value first
- `assert-no-magic-numbers` - Use named constants

### 4. Test Data (HIGH)

- `data-factories` - Use factories/builders
- `data-minimal` - Minimal test data
- `data-realistic` - Realistic edge cases
- `data-fixtures` - Manage test fixtures

### 5. Mocking (MEDIUM)

- `mock-only-boundaries` - Mock at boundaries
- `mock-verify-interactions` - Verify important calls
- `mock-minimal` - Don't over-mock
- `mock-realistic` - Realistic mock behavior

### 6. Coverage (MEDIUM)

- `cov-meaningful` - Focus on meaningful coverage
- `cov-edge-cases` - Cover edge cases
- `cov-unhappy-paths` - Test error scenarios
- `cov-not-100-percent` - 100% isn't the goal

### 7. Performance (LOW)

- `perf-fast-unit` - Unit tests run fast
- `perf-slow-integration` - Integration tests can be slower
- `perf-parallel` - Run tests in parallel

## Essential Guidelines

### Arrange, Act, Assert (AAA)

```typescript
// ✅ Clear AAA structure
describe('ShoppingCart', () => {
  it('calculates total with discount', () => {
    // Arrange
    const cart = new ShoppingCart();
    cart.addItem({ name: 'Book', price: 20 });
    cart.addItem({ name: 'Pen', price: 5 });
    cart.applyDiscount(0.1);

    // Act
    const total = cart.getTotal();

    // Assert
    expect(total).toBe(22.5);
  });
});

// ❌ Mixed up, hard to read
it('test cart', () => {
  const cart = new ShoppingCart();
  expect(cart.getTotal()).toBe(0);
  cart.addItem({ name: 'Book', price: 20 });
  expect(cart.itemCount).toBe(1);
  cart.addItem({ name: 'Pen', price: 5 });
  cart.applyDiscount(0.1);
  expect(cart.getTotal()).toBe(22.5);
});
```

### Descriptive Test Names

```typescript
// ✅ Describes behavior and scenario
describe('UserService', () => {
  describe('register', () => {
    it('creates user with hashed password', () => {});
    it('throws ValidationError when email is invalid', () => {});
    it('throws DuplicateError when email already exists', () => {});
    it('sends welcome email after successful registration', () => {});
  });
});

// ❌ Vague, unclear what's being tested
describe('UserService', () => {
  it('test1', () => {});
  it('should work', () => {});
  it('register', () => {});
});
```

### Test One Thing

```typescript
// ✅ One logical assertion per test
it('rejects password shorter than 8 characters', () => {
  const result = validatePassword('short');
  expect(result.valid).toBe(false);
  expect(result.error).toBe('Password must be at least 8 characters');
});

it('accepts password with 8 or more characters', () => {
  const result = validatePassword('longpassword');
  expect(result.valid).toBe(true);
});

// ❌ Testing multiple unrelated things
it('validates password', () => {
  expect(validatePassword('').valid).toBe(false);
  expect(validatePassword('short').valid).toBe(false);
  expect(validatePassword('longpassword').valid).toBe(true);
  expect(validatePassword('no-upper').valid).toBe(false);
  expect(validatePassword('NoNumber').valid).toBe(false);
  // If this fails, which case failed?
});
```

### Test Isolation

```typescript
// ✅ Each test sets up its own data
describe('OrderService', () => {
  let orderService: OrderService;
  let mockRepository: jest.Mocked<OrderRepository>;

  beforeEach(() => {
    mockRepository = {
      save: jest.fn(),
      find: jest.fn(),
    };
    orderService = new OrderService(mockRepository);
  });

  it('creates order with pending status', async () => {
    mockRepository.save.mockResolvedValue({ id: '1', status: 'pending' });

    const order = await orderService.create({ items: [] });

    expect(order.status).toBe('pending');
  });

  it('finds order by id', async () => {
    mockRepository.find.mockResolvedValue({ id: '1', status: 'completed' });

    const order = await orderService.findById('1');

    expect(order.status).toBe('completed');
  });
});

// ❌ Shared state between tests
let globalOrder: Order;

it('creates order', () => {
  globalOrder = orderService.create({ items: [] });
  expect(globalOrder).toBeDefined();
});

it('updates order', () => {
  // Depends on previous test!
  orderService.update(globalOrder.id, { status: 'shipped' });
});
```

### Factory/Builder Pattern for Test Data

```typescript
// ✅ Factory for test data
const createUser = (overrides: Partial<User> = {}): User => ({
  id: '1',
  email: 'test@example.com',
  name: 'Test User',
  role: 'user',
  createdAt: new Date('2024-01-01'),
  ...overrides,
});

const createOrder = (overrides: Partial<Order> = {}): Order => ({
  id: '1',
  userId: '1',
  items: [],
  total: 0,
  status: 'pending',
  ...overrides,
});

// Usage
it('calculates total from items', () => {
  const order = createOrder({
    items: [
      { productId: '1', quantity: 2, price: 10 },
      { productId: '2', quantity: 1, price: 25 },
    ],
  });

  const total = calculateTotal(order);

  expect(total).toBe(45);
});

// ❌ Repetitive inline data
it('test 1', () => {
  const user = {
    id: '1',
    email: 'test@example.com',
    name: 'Test User',
    role: 'user',
    createdAt: new Date(),
  };
  // ...
});

it('test 2', () => {
  const user = {
    id: '2',
    email: 'test2@example.com',
    name: 'Test User 2',
    role: 'admin',
    createdAt: new Date(),
  };
  // ... same structure repeated
});
```

### Mock at Boundaries

```typescript
// ✅ Mock external dependencies, not internal logic
describe('OrderService', () => {
  it('sends confirmation email after order creation', async () => {
    // Mock the boundary (email service)
    const mockEmailService = {
      send: jest.fn().mockResolvedValue(true),
    };
    const orderService = new OrderService(
      new InMemoryOrderRepository(),
      mockEmailService,
    );

    await orderService.create(createOrder());

    expect(mockEmailService.send).toHaveBeenCalledWith(
      expect.objectContaining({
        template: 'order-confirmation',
      })
    );
  });
});

// ❌ Over-mocking internal implementation
it('creates order', () => {
  const mockValidator = jest.fn();
  const mockCalculator = jest.fn();
  const mockFormatter = jest.fn();
  // Mocking every internal detail makes tests brittle
});
```

### Test Edge Cases

```typescript
describe('divide', () => {
  // Happy path
  it('divides two numbers', () => {
    expect(divide(10, 2)).toBe(5);
  });

  // Edge cases
  it('returns Infinity when dividing by zero', () => {
    expect(divide(10, 0)).toBe(Infinity);
  });

  it('handles negative numbers', () => {
    expect(divide(-10, 2)).toBe(-5);
    expect(divide(10, -2)).toBe(-5);
    expect(divide(-10, -2)).toBe(5);
  });

  it('handles decimal numbers', () => {
    expect(divide(1, 3)).toBeCloseTo(0.333, 2);
  });

  it('handles zero dividend', () => {
    expect(divide(0, 5)).toBe(0);
  });
});
```

### Test Error Scenarios

```typescript
describe('UserService.findById', () => {
  it('returns user when found', async () => {
    mockRepo.find.mockResolvedValue(createUser());

    const user = await service.findById('1');

    expect(user).toBeDefined();
  });

  it('throws NotFoundError when user does not exist', async () => {
    mockRepo.find.mockResolvedValue(null);

    await expect(service.findById('999'))
      .rejects
      .toThrow(NotFoundError);
  });

  it('throws DatabaseError on connection failure', async () => {
    mockRepo.find.mockRejectedValue(new Error('Connection failed'));

    await expect(service.findById('1'))
      .rejects
      .toThrow(DatabaseError);
  });
});
```

### Async Testing

```typescript
// ✅ Proper async testing
it('fetches user data', async () => {
  const user = await userService.fetch('1');
  expect(user.name).toBe('John');
});

it('rejects with error on failure', async () => {
  await expect(userService.fetch('invalid'))
    .rejects
    .toThrow('User not found');
});

// ✅ Testing with fake timers
it('retries failed requests', async () => {
  jest.useFakeTimers();

  const promise = fetchWithRetry('/api/data');

  // Fast-forward through retries
  jest.advanceTimersByTime(3000);

  await expect(promise).rejects.toThrow();

  jest.useRealTimers();
});
```

### Integration Tests

```typescript
// ✅ Test real integration
describe('POST /api/users', () => {
  beforeEach(async () => {
    await database.clear();
  });

  it('creates user and returns 201', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({
        email: 'test@example.com',
        password: 'password123',
      });

    expect(response.status).toBe(201);
    expect(response.body.user.email).toBe('test@example.com');

    // Verify in database
    const user = await database.users.findByEmail('test@example.com');
    expect(user).toBeDefined();
  });

  it('returns 422 for invalid email', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({
        email: 'invalid',
        password: 'password123',
      });

    expect(response.status).toBe(422);
    expect(response.body.error.details[0].field).toBe('email');
  });
});
```

### PHPUnit (Laravel)

```php
<?php

class OrderServiceTest extends TestCase
{
    use RefreshDatabase;

    private OrderService $service;

    protected function setUp(): void
    {
        parent::setUp();
        $this->service = app(OrderService::class);
    }

    /** @test */
    public function it_creates_order_with_pending_status(): void
    {
        $user = User::factory()->create();
        $items = [
            ['product_id' => 1, 'quantity' => 2],
        ];

        $order = $this->service->create($user, $items);

        $this->assertEquals(OrderStatus::Pending, $order->status);
        $this->assertDatabaseHas('orders', [
            'id' => $order->id,
            'status' => 'pending',
        ]);
    }

    /** @test */
    public function it_throws_exception_for_empty_cart(): void
    {
        $user = User::factory()->create();

        $this->expectException(EmptyCartException::class);

        $this->service->create($user, []);
    }
}
```

### Test Naming Conventions

```typescript
// Pattern: describe_when_then or should_when
describe('Calculator', () => {
  describe('add', () => {
    it('returns sum of two positive numbers', () => {});
    it('returns negative when sum is negative', () => {});
    it('handles decimal numbers', () => {});
  });
});

// BDD style
describe('User Registration', () => {
  describe('given valid credentials', () => {
    it('should create a new user', () => {});
    it('should send welcome email', () => {});
  });

  describe('given existing email', () => {
    it('should throw DuplicateError', () => {});
  });
});
```

## Test Pyramid

```
        /\
       /  \      E2E Tests (few)
      /----\     - Test critical user flows
     /      \    - Slow, expensive
    /--------\   Integration Tests (some)
   /          \  - Test component interactions
  /------------\ - Database, API calls
 /              \ Unit Tests (many)
/----------------\ - Fast, isolated
                   - Test single units
```

## Output Format

When reviewing tests, output findings:

```
file:line - [category] Description of issue
```

Example:
```
tests/user.test.ts:15 - [struct] Missing Arrange/Act/Assert separation
tests/order.test.ts:42 - [iso] Test depends on previous test's state
tests/cart.test.ts:28 - [assert] Multiple unrelated assertions in one test
```

## How to Use

Read individual rule files for detailed explanations:

```
rules/struct-aaa.md
rules/iso-independent.md
rules/mock-only-boundaries.md
```
