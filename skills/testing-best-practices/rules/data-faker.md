---
title: Faker Libraries for Test Data
priority: HIGH
category: Test Data
---

# Faker Libraries for Test Data

Use faker libraries to generate realistic, varied test data programmatically.

## Bad Example

```typescript
// Static, repetitive test data lacks variety and realism
describe('UserService', () => {
  test('creates user', async () => {
    const user = await userService.create({
      email: 'test@test.com',
      name: 'Test User',
      phone: '123-456-7890'
    });
    expect(user.id).toBeDefined();
  });

  test('creates another user', async () => {
    // Same predictable data
    const user = await userService.create({
      email: 'test2@test.com',
      name: 'Test User 2',
      phone: '123-456-7891'
    });
    expect(user.id).toBeDefined();
  });
});

describe('AddressService', () => {
  test('validates address', () => {
    // Obviously fake data
    const valid = addressService.validate({
      street: 'street',
      city: 'city',
      state: 'st',
      zip: '12345'
    });
    expect(valid).toBe(true);
  });
});

// Generating many records is painful
describe('BulkImport', () => {
  test('imports 100 users', async () => {
    const users = [];
    for (let i = 0; i < 100; i++) {
      users.push({
        email: `user${i}@test.com`,
        name: `User ${i}`,
        age: 25
      });
    }
    // All users look identical except for number suffix
    const result = await importService.bulkImport(users);
    expect(result.imported).toBe(100);
  });
});
```

## Good Example

```typescript
import { faker } from '@faker-js/faker';

// Configure faker for reproducible tests
beforeAll(() => {
  faker.seed(12345); // Same seed = same sequence of values
});

// Custom factories using faker
const UserDataFactory = {
  create(overrides: Partial<UserInput> = {}): UserInput {
    return {
      email: faker.internet.email(),
      name: faker.person.fullName(),
      phone: faker.phone.number(),
      dateOfBirth: faker.date.birthdate({ min: 18, max: 80, mode: 'age' }),
      address: {
        street: faker.location.streetAddress(),
        city: faker.location.city(),
        state: faker.location.state({ abbreviated: true }),
        zip: faker.location.zipCode(),
        country: faker.location.country()
      },
      ...overrides
    };
  },

  createMany(count: number, overrides: Partial<UserInput> = {}): UserInput[] {
    return Array.from({ length: count }, () => this.create(overrides));
  }
};

const ProductDataFactory = {
  create(overrides: Partial<ProductInput> = {}): ProductInput {
    return {
      name: faker.commerce.productName(),
      description: faker.commerce.productDescription(),
      price: parseFloat(faker.commerce.price({ min: 5, max: 500 })),
      category: faker.commerce.department(),
      sku: faker.string.alphanumeric(10).toUpperCase(),
      stock: faker.number.int({ min: 0, max: 1000 }),
      ...overrides
    };
  }
};

const OrderDataFactory = {
  create(overrides: Partial<OrderInput> = {}): OrderInput {
    const itemCount = faker.number.int({ min: 1, max: 5 });

    return {
      customerId: faker.string.uuid(),
      items: Array.from({ length: itemCount }, () => ({
        productId: faker.string.uuid(),
        quantity: faker.number.int({ min: 1, max: 10 }),
        price: parseFloat(faker.commerce.price())
      })),
      shippingAddress: {
        name: faker.person.fullName(),
        street: faker.location.streetAddress(),
        city: faker.location.city(),
        state: faker.location.state(),
        zip: faker.location.zipCode(),
        country: 'USA'
      },
      ...overrides
    };
  }
};

describe('UserService', () => {
  test('creates user with valid data', async () => {
    const userData = UserDataFactory.create();

    const user = await userService.create(userData);

    expect(user.id).toBeDefined();
    expect(user.email).toBe(userData.email);
  });

  test('validates unique email constraint', async () => {
    const email = faker.internet.email();
    await userService.create(UserDataFactory.create({ email }));

    await expect(
      userService.create(UserDataFactory.create({ email }))
    ).rejects.toThrow('Email already exists');
  });

  test('rejects invalid email format', async () => {
    const userData = UserDataFactory.create({
      email: faker.string.alpha(10) // No @ symbol
    });

    await expect(userService.create(userData)).rejects.toThrow('Invalid email');
  });
});

describe('BulkImport', () => {
  test('imports 100 users with varied data', async () => {
    const users = UserDataFactory.createMany(100);

    const result = await importService.bulkImport(users);

    expect(result.imported).toBe(100);
  });

  test('handles duplicate emails in batch', async () => {
    const duplicateEmail = faker.internet.email();
    const users = [
      UserDataFactory.create({ email: duplicateEmail }),
      ...UserDataFactory.createMany(8),
      UserDataFactory.create({ email: duplicateEmail })
    ];

    const result = await importService.bulkImport(users);

    expect(result.imported).toBe(9);
    expect(result.duplicates).toBe(1);
  });
});

describe('SearchService', () => {
  test('finds products by name', async () => {
    const targetName = faker.commerce.productName();
    const products = [
      ProductDataFactory.create({ name: targetName }),
      ...ProductDataFactory.createMany(9)
    ];
    await productService.bulkCreate(products);

    const results = await searchService.search(targetName.split(' ')[0]);

    expect(results).toContainEqual(
      expect.objectContaining({ name: targetName })
    );
  });
});

describe('OrderProcessing', () => {
  test('calculates shipping for various order sizes', async () => {
    // Test with different randomly generated orders
    const orders = Array.from({ length: 10 }, () => OrderDataFactory.create());

    for (const order of orders) {
      const shipping = await shippingService.calculate(order);

      expect(shipping.cost).toBeGreaterThan(0);
      expect(shipping.estimatedDays).toBeGreaterThanOrEqual(1);
    }
  });
});

// Locale-specific data for internationalization testing
describe('I18n', () => {
  test('handles German address format', () => {
    faker.setLocale('de');

    const address = {
      street: faker.location.streetAddress(),
      city: faker.location.city(),
      zip: faker.location.zipCode()
    };

    const formatted = addressFormatter.format(address, 'de');

    expect(formatted).toMatch(/^\d{5}/); // German zip first
  });

  afterEach(() => {
    faker.setLocale('en'); // Reset locale
  });
});
```

## Why

Faker libraries enhance test data quality:

1. **Realistic variety**: Each test run uses different realistic data
2. **Edge case discovery**: Random data reveals issues static data misses
3. **Internationalization**: Generate locale-specific data easily
4. **Bulk generation**: Create hundreds of records with one line
5. **Reproducibility**: Seeded faker produces consistent results
6. **Reduced bias**: Random data avoids author bias in test data

Popular faker libraries:
- **JavaScript**: @faker-js/faker
- **Python**: Faker
- **Ruby**: Faker
- **Java**: JavaFaker
- **Go**: gofakeit

Best practices:
- Seed faker for reproducible tests
- Combine faker with custom factories
- Use appropriate locales for i18n testing
- Override specific values when testing specific behaviors
