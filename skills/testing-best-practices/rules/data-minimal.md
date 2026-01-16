# Minimal Test Data

Use only the data necessary to test the specific behavior being verified.

## Bad Example

```typescript
// Excessive data obscures what's actually being tested
describe('EmailValidator', () => {
  test('validates email format', () => {
    // Most of this data is irrelevant to email validation
    const user = {
      id: 'user-123',
      firstName: 'John',
      lastName: 'Doe',
      email: 'john.doe@example.com',
      phone: '+1-555-123-4567',
      dateOfBirth: new Date('1990-05-15'),
      address: {
        street: '123 Main St',
        city: 'Springfield',
        state: 'IL',
        zip: '62701',
        country: 'USA'
      },
      preferences: {
        newsletter: true,
        notifications: { email: true, sms: false, push: true },
        theme: 'dark',
        language: 'en-US'
      },
      subscription: {
        plan: 'premium',
        startDate: new Date('2023-01-01'),
        renewalDate: new Date('2024-01-01'),
        features: ['analytics', 'priority-support', 'api-access']
      },
      createdAt: new Date('2022-06-15'),
      updatedAt: new Date('2023-12-01'),
      lastLoginAt: new Date('2024-01-10')
    };

    expect(emailValidator.isValid(user.email)).toBe(true);
  });
});

describe('DiscountCalculator', () => {
  test('applies percentage discount', () => {
    // Excessive item details when only price matters
    const items = [
      {
        id: 'item-001',
        sku: 'WIDGET-BLU-LRG',
        name: 'Large Blue Widget',
        description: 'A high-quality widget in blue color, large size',
        category: 'widgets',
        subcategory: 'colored-widgets',
        brand: 'WidgetCo',
        manufacturer: 'WidgetCo Inc.',
        price: 100,
        cost: 45,
        weight: 2.5,
        dimensions: { length: 10, width: 8, height: 5 },
        images: ['widget-1.jpg', 'widget-2.jpg'],
        tags: ['popular', 'bestseller'],
        inventory: { available: 150, reserved: 10, warehouse: 'A1' },
        rating: { average: 4.5, count: 234 }
      }
    ];

    const total = calculator.applyDiscount(items, 10);
    expect(total).toBe(90);
  });
});
```

## Good Example

```typescript
// Minimal data focuses attention on what matters
describe('EmailValidator', () => {
  test('returns true for valid email format', () => {
    const email = 'user@example.com';

    expect(emailValidator.isValid(email)).toBe(true);
  });

  test('returns false for email without domain', () => {
    const email = 'user@';

    expect(emailValidator.isValid(email)).toBe(false);
  });

  test('returns false for email without @ symbol', () => {
    const email = 'userexample.com';

    expect(emailValidator.isValid(email)).toBe(false);
  });
});

describe('DiscountCalculator', () => {
  test('applies percentage discount to item price', () => {
    const items = [{ price: 100 }];
    const discountPercent = 10;

    const total = calculator.applyDiscount(items, discountPercent);

    expect(total).toBe(90);
  });

  test('applies discount to multiple items', () => {
    const items = [
      { price: 100 },
      { price: 50 }
    ];

    const total = calculator.applyDiscount(items, 20);

    expect(total).toBe(120); // 150 - 20%
  });
});

// When more context is needed, use only relevant properties
describe('ShippingCalculator', () => {
  test('calculates shipping based on weight', () => {
    const item = {
      weight: 2.5,
      price: 100 // Only if price affects shipping calculation
    };
    const destination = { country: 'USA' };

    const shipping = calculator.calculate(item, destination);

    expect(shipping).toBe(12.50);
  });
});

// For complex objects, use partial types or factories with minimal data
describe('OrderProcessor', () => {
  test('rejects order with invalid status', () => {
    const order = OrderFactory.create({
      status: 'cancelled' // Only the property that matters
    });

    expect(() => processor.process(order)).toThrow(InvalidStatusError);
  });

  test('processes pending order', () => {
    const order = OrderFactory.create({
      status: 'pending'
    });

    const result = processor.process(order);

    expect(result.status).toBe('processing');
  });
});
```

## Why

Minimal test data improves tests in several ways:

1. **Clarity**: It's immediately obvious what data affects the behavior
2. **Maintainability**: Less data to update when requirements change
3. **Focus**: Tests clearly document which inputs matter
4. **Speed**: Less memory allocation and processing
5. **Noise reduction**: Irrelevant data doesn't distract readers

Guidelines for minimal data:
- Include only properties the function actually uses
- Use factories with defaults for required but irrelevant fields
- Consider what would change the test outcome
- If removing data doesn't break the test, remove it
- Document why specific values were chosen if not obvious
