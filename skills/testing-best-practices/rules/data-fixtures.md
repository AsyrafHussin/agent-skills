# Test Fixtures

Use fixtures to manage reusable test data and complex setup scenarios.

## Bad Example

```typescript
// Repeated inline data setup across tests
describe('ReportService', () => {
  test('generates sales report', async () => {
    // Setup data inline - duplicated across tests
    const salesData = [
      { date: '2024-01-01', product: 'Widget', quantity: 10, price: 29.99 },
      { date: '2024-01-01', product: 'Gadget', quantity: 5, price: 49.99 },
      { date: '2024-01-02', product: 'Widget', quantity: 15, price: 29.99 },
      { date: '2024-01-02', product: 'Gadget', quantity: 8, price: 49.99 },
      // ... 50 more lines of data
    ];

    await db.insert('sales', salesData);

    const report = await reportService.generate('sales', { month: 'january' });

    expect(report.totalRevenue).toBe(1799.55);
  });

  test('filters report by product', async () => {
    // Same data duplicated again
    const salesData = [
      { date: '2024-01-01', product: 'Widget', quantity: 10, price: 29.99 },
      // ... same 50+ lines
    ];

    await db.insert('sales', salesData);

    const report = await reportService.generate('sales', {
      month: 'january',
      product: 'Widget'
    });

    expect(report.items).toHaveLength(2);
  });
});
```

## Good Example

```typescript
// fixtures/sales.fixture.ts
export const salesFixtures = {
  january2024: [
    { date: '2024-01-01', product: 'Widget', quantity: 10, price: 29.99 },
    { date: '2024-01-01', product: 'Gadget', quantity: 5, price: 49.99 },
    { date: '2024-01-02', product: 'Widget', quantity: 15, price: 29.99 },
    { date: '2024-01-02', product: 'Gadget', quantity: 8, price: 49.99 }
  ],

  // Pre-calculated expected values
  january2024Expected: {
    totalRevenue: 1099.55,
    widgetRevenue: 749.75,
    gadgetRevenue: 349.80,
    widgetCount: 25,
    gadgetCount: 13
  }
};

// fixtures/users.fixture.ts
export const userFixtures = {
  admin: {
    id: 'user-admin',
    email: 'admin@example.com',
    role: 'admin',
    permissions: ['read', 'write', 'delete', 'admin']
  },

  regularUser: {
    id: 'user-regular',
    email: 'user@example.com',
    role: 'user',
    permissions: ['read']
  },

  premiumUser: {
    id: 'user-premium',
    email: 'premium@example.com',
    role: 'user',
    subscription: 'premium',
    permissions: ['read', 'write', 'export']
  }
};

// fixtures/products.fixture.ts
export const productFixtures = {
  basic: [
    { id: 'prod-1', name: 'Widget', price: 29.99, stock: 100 },
    { id: 'prod-2', name: 'Gadget', price: 49.99, stock: 50 }
  ],

  outOfStock: [
    { id: 'prod-3', name: 'Rare Item', price: 199.99, stock: 0 }
  ],

  discounted: [
    { id: 'prod-4', name: 'Sale Widget', price: 29.99, discount: 20, stock: 200 }
  ]
};

// helpers/fixture-loader.ts
export class FixtureLoader {
  constructor(private db: Database) {}

  async loadSales(fixture: typeof salesFixtures.january2024): Promise<void> {
    await this.db.insert('sales', fixture);
  }

  async loadUsers(...users: Array<typeof userFixtures.admin>): Promise<void> {
    await this.db.insert('users', users);
  }

  async loadProducts(products: typeof productFixtures.basic): Promise<void> {
    await this.db.insert('products', products);
  }

  async clearAll(): Promise<void> {
    await this.db.truncate(['sales', 'users', 'products']);
  }
}

// Clean tests using fixtures
describe('ReportService', () => {
  let fixtureLoader: FixtureLoader;

  beforeAll(() => {
    fixtureLoader = new FixtureLoader(db);
  });

  beforeEach(async () => {
    await fixtureLoader.clearAll();
  });

  describe('sales reports', () => {
    beforeEach(async () => {
      await fixtureLoader.loadSales(salesFixtures.january2024);
    });

    test('calculates total revenue for month', async () => {
      const report = await reportService.generate('sales', { month: 'january' });

      expect(report.totalRevenue).toBe(
        salesFixtures.january2024Expected.totalRevenue
      );
    });

    test('filters by product', async () => {
      const report = await reportService.generate('sales', {
        month: 'january',
        product: 'Widget'
      });

      expect(report.totalRevenue).toBe(
        salesFixtures.january2024Expected.widgetRevenue
      );
    });

    test('groups by product', async () => {
      const report = await reportService.generate('sales', {
        month: 'january',
        groupBy: 'product'
      });

      expect(report.groups).toHaveLength(2);
      expect(report.groups.find(g => g.product === 'Widget').quantity).toBe(
        salesFixtures.january2024Expected.widgetCount
      );
    });
  });
});

describe('AuthorizationService', () => {
  beforeEach(async () => {
    await fixtureLoader.clearAll();
  });

  test('admin can delete resources', async () => {
    await fixtureLoader.loadUsers(userFixtures.admin);

    const canDelete = await authService.checkPermission(
      userFixtures.admin.id,
      'delete'
    );

    expect(canDelete).toBe(true);
  });

  test('regular user cannot delete resources', async () => {
    await fixtureLoader.loadUsers(userFixtures.regularUser);

    const canDelete = await authService.checkPermission(
      userFixtures.regularUser.id,
      'delete'
    );

    expect(canDelete).toBe(false);
  });

  test('premium user can export data', async () => {
    await fixtureLoader.loadUsers(userFixtures.premiumUser);

    const canExport = await authService.checkPermission(
      userFixtures.premiumUser.id,
      'export'
    );

    expect(canExport).toBe(true);
  });
});
```

## Why

Test fixtures provide structured test data management:

1. **Reusability**: Same data used across multiple tests
2. **Consistency**: All tests use identical baseline data
3. **Maintainability**: Update data in one place
4. **Documentation**: Fixtures describe test scenarios
5. **Separation**: Test logic separate from test data
6. **Pre-calculated values**: Expected results stored with fixtures

Best practices for fixtures:
- Organize fixtures by domain/entity
- Include expected values alongside input data
- Create fixture loaders for database setup
- Use descriptive names for fixture scenarios
- Keep fixtures in dedicated directories
- Version control fixtures with tests
