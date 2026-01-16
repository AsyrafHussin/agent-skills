# Meaningful Assertion Messages

Add descriptive messages to assertions to clarify intent and improve debugging.

## Bad Example

```typescript
// No messages - unclear what failed when tests break
describe('PaymentProcessor', () => {
  test('processes valid payment', async () => {
    const result = await processor.processPayment({
      amount: 100,
      currency: 'USD',
      cardNumber: '4111111111111111'
    });

    expect(result.status).toBe('success');
    expect(result.transactionId).toBeDefined();
    expect(result.amount).toBe(100);
    expect(result.fee).toBeLessThan(5);
  });

  test('validates payment data', () => {
    const errors = validator.validate({
      amount: -50,
      currency: 'INVALID',
      cardNumber: '1234'
    });

    expect(errors.length).toBe(3);
    expect(errors[0].field).toBe('amount');
    expect(errors[1].field).toBe('currency');
    expect(errors[2].field).toBe('cardNumber');
  });

  test('handles currency conversion', () => {
    const result = converter.convert(100, 'USD', 'EUR');

    // When this fails, what was the actual value?
    expect(result).toBeGreaterThan(80);
    expect(result).toBeLessThan(120);
  });
});
```

## Good Example

```typescript
// Descriptive messages explain intent and help debugging
describe('PaymentProcessor', () => {
  test('processes valid payment', async () => {
    const result = await processor.processPayment({
      amount: 100,
      currency: 'USD',
      cardNumber: '4111111111111111'
    });

    expect(result.status).toBe('success');
    expect(result.transactionId).toBeDefined();
    expect(result.amount).toBe(100);

    // Custom message for context
    expect(result.fee).toBeLessThan(
      5,
      `Processing fee ${result.fee} exceeds maximum allowed fee of 5`
    );
  });

  test('validates payment data', () => {
    const errors = validator.validate({
      amount: -50,
      currency: 'INVALID',
      cardNumber: '1234'
    });

    expect(errors).toHaveLength(3);

    // Using describe.each for clarity on multiple validations
    const expectedErrors = [
      { field: 'amount', reason: 'Amount must be positive' },
      { field: 'currency', reason: 'Invalid currency code' },
      { field: 'cardNumber', reason: 'Card number too short' }
    ];

    expectedErrors.forEach(({ field, reason }, index) => {
      expect(errors[index]).toMatchObject(
        { field },
        `Expected error at index ${index} to be for field "${field}" (${reason})`
      );
    });
  });

  test('handles currency conversion within expected range', () => {
    const result = converter.convert(100, 'USD', 'EUR');

    expect(result).toBeGreaterThan(
      80,
      `Converted amount ${result} EUR is below minimum expected (80 EUR for 100 USD)`
    );
    expect(result).toBeLessThan(
      120,
      `Converted amount ${result} EUR exceeds maximum expected (120 EUR for 100 USD)`
    );
  });
});

// Using custom matchers for domain-specific assertions
expect.extend({
  toBeValidEmail(received: string) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    const pass = emailRegex.test(received);

    return {
      pass,
      message: () =>
        pass
          ? `Expected "${received}" not to be a valid email address`
          : `Expected "${received}" to be a valid email address (format: user@domain.tld)`
    };
  },

  toBeWithinRange(received: number, floor: number, ceiling: number) {
    const pass = received >= floor && received <= ceiling;

    return {
      pass,
      message: () =>
        pass
          ? `Expected ${received} not to be within range ${floor} - ${ceiling}`
          : `Expected ${received} to be within range ${floor} - ${ceiling}, but it was ${
              received < floor ? `${floor - received} below minimum` : `${received - ceiling} above maximum`
            }`
    };
  }
});

describe('with custom matchers', () => {
  test('validates email format', () => {
    expect('user@example.com').toBeValidEmail();
    expect('invalid-email').not.toBeValidEmail();
  });

  test('conversion rate within expected range', () => {
    const result = converter.convert(100, 'USD', 'EUR');
    expect(result).toBeWithinRange(80, 120);
  });
});
```

## Why

Meaningful assertion messages provide significant benefits:

1. **Faster debugging**: Immediately understand what failed without reading code
2. **Context preservation**: Know the actual values involved in the failure
3. **Documentation**: Messages explain the business reasoning behind assertions
4. **CI/CD clarity**: Build logs show meaningful failures, not cryptic comparisons
5. **Code review**: Reviewers understand the intent of assertions

When to add messages:
- Complex assertions with multiple parts
- Numeric comparisons with business meaning
- Validations with specific requirements
- Any assertion where the failure reason isn't obvious
- Loop-based assertions where index matters

Consider custom matchers for domain-specific assertions that you use frequently.
