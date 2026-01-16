# Realistic Test Data

Use data that resembles real-world inputs to catch edge cases and validation issues.

## Bad Example

```typescript
// Unrealistic placeholder data misses real-world issues
describe('UserRegistration', () => {
  test('registers new user', async () => {
    const result = await register({
      name: 'test',
      email: 'test@test.com',
      password: '123',
      phone: '123',
      address: 'test'
    });

    expect(result.success).toBe(true);
  });
});

describe('PaymentProcessor', () => {
  test('processes payment', async () => {
    const result = await processPayment({
      cardNumber: '1234',
      expiry: '12/24',
      cvv: '123',
      amount: 1
    });

    // Might pass with mock but fails with real validation
    expect(result.status).toBe('success');
  });
});

describe('SearchService', () => {
  test('searches products', async () => {
    const results = await search('aaa');

    expect(results).toHaveLength(0);
  });
});

describe('InternationalizationService', () => {
  test('formats user name', () => {
    // Only tests ASCII names
    const formatted = formatName('John Doe');

    expect(formatted).toBe('Doe, John');
  });
});
```

## Good Example

```typescript
// Realistic data catches real-world edge cases
describe('UserRegistration', () => {
  test('registers user with common name format', async () => {
    const result = await register({
      name: 'Sarah Johnson',
      email: 'sarah.johnson@gmail.com',
      password: 'SecureP@ss123!',
      phone: '+1-555-867-5309',
      address: '742 Evergreen Terrace, Springfield, IL 62701'
    });

    expect(result.success).toBe(true);
  });

  test('registers user with hyphenated name', async () => {
    const result = await register({
      name: 'Mary-Jane Watson-Parker',
      email: 'mj.watson@example.com',
      password: 'WebSl!ng3r2024'
    });

    expect(result.success).toBe(true);
  });

  test('registers user with international characters', async () => {
    const result = await register({
      name: 'José García-López',
      email: 'jose.garcia@empresa.es',
      password: 'Contraseña123!'
    });

    expect(result.success).toBe(true);
  });

  test('registers user with unicode name', async () => {
    const result = await register({
      name: '田中 太郎',
      email: 'tanaka.taro@example.jp',
      password: 'パスワード123!'
    });

    expect(result.success).toBe(true);
  });
});

describe('PaymentProcessor', () => {
  // Using realistic test card numbers (Stripe test cards)
  const testCards = {
    visa: '4242424242424242',
    mastercard: '5555555555554444',
    amex: '378282246310005',
    declined: '4000000000000002'
  };

  test('processes Visa payment', async () => {
    const result = await processPayment({
      cardNumber: testCards.visa,
      expiry: '12/28',
      cvv: '123',
      amount: 49.99,
      currency: 'USD'
    });

    expect(result.status).toBe('success');
    expect(result.last4).toBe('4242');
  });

  test('handles declined card', async () => {
    const result = await processPayment({
      cardNumber: testCards.declined,
      expiry: '12/28',
      cvv: '123',
      amount: 99.99
    });

    expect(result.status).toBe('declined');
    expect(result.error).toContain('insufficient funds');
  });

  test('processes realistic order amount', async () => {
    // Real e-commerce amounts, not round numbers
    const result = await processPayment({
      cardNumber: testCards.visa,
      expiry: '12/28',
      cvv: '123',
      amount: 147.83
    });

    expect(result.amount).toBe(147.83);
  });
});

describe('SearchService', () => {
  test('finds products with common search terms', async () => {
    const results = await search('wireless headphones');

    expect(results.length).toBeGreaterThan(0);
    expect(results[0].name.toLowerCase()).toContain('headphone');
  });

  test('handles search with typos', async () => {
    const results = await search('wireles headphone');

    expect(results.length).toBeGreaterThan(0); // Fuzzy matching
  });

  test('handles search with special characters', async () => {
    const results = await search("kid's toys");

    expect(results).toBeDefined();
  });
});

describe('AddressFormatter', () => {
  test('formats US address', () => {
    const formatted = formatAddress({
      street: '1600 Pennsylvania Avenue NW',
      city: 'Washington',
      state: 'DC',
      zip: '20500',
      country: 'USA'
    });

    expect(formatted).toBe('1600 Pennsylvania Avenue NW\nWashington, DC 20500\nUSA');
  });

  test('formats UK address with postcode', () => {
    const formatted = formatAddress({
      street: '221B Baker Street',
      city: 'London',
      postcode: 'NW1 6XE',
      country: 'UK'
    });

    expect(formatted).toContain('NW1 6XE');
  });

  test('formats address with apartment number', () => {
    const formatted = formatAddress({
      street: '350 Fifth Avenue',
      unit: 'Floor 102',
      city: 'New York',
      state: 'NY',
      zip: '10118'
    });

    expect(formatted).toContain('Floor 102');
  });
});
```

## Why

Realistic test data provides significant benefits:

1. **Edge case discovery**: Real data reveals issues placeholder data misses
2. **Validation testing**: Ensures validation rules work with actual formats
3. **Internationalization**: Tests with non-ASCII characters catch encoding issues
4. **Documentation**: Tests show examples of valid real-world inputs
5. **Integration readiness**: Tests are more likely to pass in production

Guidelines for realistic data:
- Use actual format patterns (phone numbers, addresses, emails)
- Include international characters and formats
- Test with realistic amounts and quantities
- Use known test values (like Stripe test card numbers)
- Include edge cases found in production data
- Consider data from different regions and locales
