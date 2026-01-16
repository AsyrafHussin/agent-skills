# Custom Matchers

Create custom assertion matchers for domain-specific validations to improve readability and reusability.

## Bad Example

```typescript
// Repeated complex assertions without abstraction
describe('OrderService', () => {
  test('creates pending order', () => {
    const order = orderService.create({ items: [{ id: 1 }] });

    // Repeated pattern for checking order state
    expect(order.status).toBe('pending');
    expect(order.paidAt).toBeNull();
    expect(order.shippedAt).toBeNull();
    expect(order.items.length).toBeGreaterThan(0);
  });

  test('creates another pending order', () => {
    const order = orderService.create({ items: [{ id: 2 }, { id: 3 }] });

    // Same assertions duplicated
    expect(order.status).toBe('pending');
    expect(order.paidAt).toBeNull();
    expect(order.shippedAt).toBeNull();
    expect(order.items.length).toBeGreaterThan(0);
  });
});

describe('UserValidator', () => {
  test('validates email format', () => {
    const email = 'user@example.com';

    // Complex regex repeated in tests
    expect(email).toMatch(/^[^\s@]+@[^\s@]+\.[^\s@]+$/);
    expect(email.split('@')[1]).toContain('.');
  });

  test('validates another email', () => {
    const email = 'admin@company.org';

    // Same validation duplicated
    expect(email).toMatch(/^[^\s@]+@[^\s@]+\.[^\s@]+$/);
    expect(email.split('@')[1]).toContain('.');
  });
});

describe('DateUtils', () => {
  test('returns date within business hours', () => {
    const date = dateUtils.nextBusinessHour();

    // Complex multi-step assertion
    const hours = date.getHours();
    expect(hours).toBeGreaterThanOrEqual(9);
    expect(hours).toBeLessThanOrEqual(17);
    expect(date.getDay()).not.toBe(0); // Not Sunday
    expect(date.getDay()).not.toBe(6); // Not Saturday
  });
});
```

## Good Example

```typescript
// Custom matchers encapsulate domain-specific assertions
expect.extend({
  toBePendingOrder(received: Order) {
    const isPending = received.status === 'pending';
    const isUnpaid = received.paidAt === null;
    const isUnshipped = received.shippedAt === null;
    const hasItems = received.items.length > 0;

    const pass = isPending && isUnpaid && isUnshipped && hasItems;

    const failures: string[] = [];
    if (!isPending) failures.push(`status is "${received.status}" (expected "pending")`);
    if (!isUnpaid) failures.push(`paidAt is ${received.paidAt} (expected null)`);
    if (!isUnshipped) failures.push(`shippedAt is ${received.shippedAt} (expected null)`);
    if (!hasItems) failures.push('order has no items');

    return {
      pass,
      message: () =>
        pass
          ? `Expected order not to be a valid pending order`
          : `Expected order to be a valid pending order, but: ${failures.join(', ')}`
    };
  },

  toBeShippedOrder(received: Order) {
    const isShipped = received.status === 'shipped';
    const isPaid = received.paidAt !== null;
    const hasShippingDate = received.shippedAt !== null;
    const hasTrackingNumber = !!received.trackingNumber;

    const pass = isShipped && isPaid && hasShippingDate && hasTrackingNumber;

    return {
      pass,
      message: () =>
        pass
          ? `Expected order not to be a valid shipped order`
          : `Expected order to be shipped with payment and tracking info`
    };
  },

  toBeValidEmail(received: string) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    const pass = emailRegex.test(received);

    return {
      pass,
      message: () =>
        pass
          ? `Expected "${received}" not to be a valid email`
          : `Expected "${received}" to be a valid email (format: user@domain.tld)`
    };
  },

  toBeWithinBusinessHours(received: Date) {
    const hours = received.getHours();
    const dayOfWeek = received.getDay();
    const isWeekday = dayOfWeek !== 0 && dayOfWeek !== 6;
    const isBusinessHours = hours >= 9 && hours <= 17;

    const pass = isWeekday && isBusinessHours;

    return {
      pass,
      message: () =>
        pass
          ? `Expected ${received} not to be within business hours`
          : `Expected ${received} to be within business hours (Mon-Fri 9AM-5PM), ` +
            `but was ${['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'][dayOfWeek]} at ${hours}:00`
    };
  },

  toHaveValidationError(received: ValidationResult, field: string, message?: string) {
    const error = received.errors.find(e => e.field === field);
    const hasError = !!error;
    const messageMatches = message ? error?.message.includes(message) : true;

    const pass = hasError && messageMatches;

    return {
      pass,
      message: () =>
        pass
          ? `Expected no validation error for field "${field}"`
          : hasError
            ? `Expected error message to include "${message}", but got "${error?.message}"`
            : `Expected validation error for field "${field}", but none found. Errors: ${JSON.stringify(received.errors)}`
    };
  }
});

// TypeScript declarations for custom matchers
declare global {
  namespace jest {
    interface Matchers<R> {
      toBePendingOrder(): R;
      toBeShippedOrder(): R;
      toBeValidEmail(): R;
      toBeWithinBusinessHours(): R;
      toHaveValidationError(field: string, message?: string): R;
    }
  }
}

// Clean tests using custom matchers
describe('OrderService', () => {
  test('creates pending order', () => {
    const order = orderService.create({ items: [{ id: 1 }] });

    expect(order).toBePendingOrder();
  });

  test('ships order with tracking', () => {
    const order = orderService.create({ items: [{ id: 1 }] });
    orderService.pay(order.id);
    orderService.ship(order.id, 'TRACK123');

    expect(order).toBeShippedOrder();
  });
});

describe('UserValidator', () => {
  test('accepts valid email addresses', () => {
    expect('user@example.com').toBeValidEmail();
    expect('admin@company.org').toBeValidEmail();
  });

  test('rejects invalid email addresses', () => {
    expect('invalid').not.toBeValidEmail();
    expect('missing@domain').not.toBeValidEmail();
  });
});

describe('DateUtils', () => {
  test('returns date within business hours', () => {
    const date = dateUtils.nextBusinessHour();

    expect(date).toBeWithinBusinessHours();
  });
});

describe('FormValidation', () => {
  test('validates required fields', () => {
    const result = validator.validate({ email: '', password: '' });

    expect(result).toHaveValidationError('email', 'required');
    expect(result).toHaveValidationError('password', 'required');
  });
});
```

## Why

Custom matchers provide significant benefits:

1. **Readability**: Tests read like specifications in domain language
2. **Reusability**: Define complex validations once, use everywhere
3. **Better errors**: Custom messages explain failures in business terms
4. **Maintainability**: Update validation logic in one place
5. **Encapsulation**: Hide implementation details of validations
6. **Consistency**: Same validation rules applied uniformly

When to create custom matchers:
- Repeated assertion patterns across tests
- Domain-specific validations
- Complex multi-property checks
- Assertions that need better error messages
- Validations that would benefit from parameterization
