# Test Data Builders

Use the builder pattern for complex test data with fluent, chainable configuration.

## Bad Example

```typescript
// Complex nested object construction is hard to read and maintain
describe('ReportGenerator', () => {
  test('generates monthly sales report', () => {
    const report = {
      id: 'report-1',
      type: 'monthly-sales',
      dateRange: {
        start: new Date('2024-01-01'),
        end: new Date('2024-01-31')
      },
      filters: {
        regions: ['US', 'EU'],
        categories: ['electronics'],
        minAmount: 100,
        maxAmount: null,
        includeRefunds: false,
        excludeTestOrders: true
      },
      groupBy: ['region', 'category'],
      sortBy: { field: 'totalSales', direction: 'desc' },
      format: {
        type: 'pdf',
        includeCharts: true,
        chartTypes: ['bar', 'pie'],
        pageSize: 'A4',
        orientation: 'landscape'
      },
      recipients: [
        { email: 'manager@example.com', name: 'Manager' }
      ],
      schedule: {
        frequency: 'monthly',
        dayOfMonth: 1,
        time: '09:00'
      }
    };

    expect(generator.canGenerate(report)).toBe(true);
  });
});
```

## Good Example

```typescript
// Builder pattern for fluent test data construction
class ReportBuilder {
  private report: Partial<Report> = {
    id: `report-${Date.now()}`,
    type: 'basic',
    filters: {},
    format: { type: 'pdf' }
  };

  static create(): ReportBuilder {
    return new ReportBuilder();
  }

  withId(id: string): this {
    this.report.id = id;
    return this;
  }

  ofType(type: ReportType): this {
    this.report.type = type;
    return this;
  }

  forDateRange(start: Date, end: Date): this {
    this.report.dateRange = { start, end };
    return this;
  }

  forMonth(year: number, month: number): this {
    const start = new Date(year, month - 1, 1);
    const end = new Date(year, month, 0);
    return this.forDateRange(start, end);
  }

  forRegions(...regions: string[]): this {
    this.report.filters = { ...this.report.filters, regions };
    return this;
  }

  forCategories(...categories: string[]): this {
    this.report.filters = { ...this.report.filters, categories };
    return this;
  }

  withAmountRange(min?: number, max?: number): this {
    this.report.filters = {
      ...this.report.filters,
      minAmount: min,
      maxAmount: max
    };
    return this;
  }

  includingRefunds(): this {
    this.report.filters = { ...this.report.filters, includeRefunds: true };
    return this;
  }

  excludingTestOrders(): this {
    this.report.filters = { ...this.report.filters, excludeTestOrders: true };
    return this;
  }

  groupedBy(...fields: string[]): this {
    this.report.groupBy = fields;
    return this;
  }

  sortedBy(field: string, direction: 'asc' | 'desc' = 'asc'): this {
    this.report.sortBy = { field, direction };
    return this;
  }

  asPdf(options: Partial<PdfOptions> = {}): this {
    this.report.format = {
      type: 'pdf',
      pageSize: 'A4',
      orientation: 'portrait',
      ...options
    };
    return this;
  }

  withCharts(...chartTypes: ChartType[]): this {
    this.report.format = {
      ...this.report.format,
      includeCharts: true,
      chartTypes
    };
    return this;
  }

  sendTo(email: string, name?: string): this {
    this.report.recipients = [
      ...(this.report.recipients || []),
      { email, name: name ?? email.split('@')[0] }
    ];
    return this;
  }

  scheduledMonthly(dayOfMonth: number, time: string): this {
    this.report.schedule = {
      frequency: 'monthly',
      dayOfMonth,
      time
    };
    return this;
  }

  build(): Report {
    return this.report as Report;
  }
}

// Preset builders for common scenarios
class MonthlySalesReportBuilder extends ReportBuilder {
  constructor() {
    super();
    this.ofType('monthly-sales')
      .excludingTestOrders()
      .groupedBy('region', 'category')
      .sortedBy('totalSales', 'desc')
      .asPdf({ orientation: 'landscape' });
  }

  static create(): MonthlySalesReportBuilder {
    return new MonthlySalesReportBuilder();
  }
}

// Clean, readable tests using builders
describe('ReportGenerator', () => {
  test('generates basic monthly sales report', () => {
    const report = ReportBuilder.create()
      .ofType('monthly-sales')
      .forMonth(2024, 1)
      .build();

    expect(generator.canGenerate(report)).toBe(true);
  });

  test('generates regional sales report with charts', () => {
    const report = MonthlySalesReportBuilder.create()
      .forMonth(2024, 1)
      .forRegions('US', 'EU')
      .withCharts('bar', 'pie')
      .build();

    const result = generator.generate(report);

    expect(result.pages).toHaveLength(2);
    expect(result.charts).toHaveLength(2);
  });

  test('generates filtered report for high-value sales', () => {
    const report = ReportBuilder.create()
      .ofType('sales-detail')
      .forMonth(2024, 1)
      .forCategories('electronics')
      .withAmountRange(1000, undefined)
      .excludingTestOrders()
      .build();

    const result = generator.generate(report);

    expect(result.data.every(row => row.amount >= 1000)).toBe(true);
  });

  test('schedules and emails monthly report', () => {
    const report = MonthlySalesReportBuilder.create()
      .forMonth(2024, 1)
      .sendTo('manager@example.com', 'Sales Manager')
      .sendTo('ceo@example.com', 'CEO')
      .scheduledMonthly(1, '09:00')
      .build();

    expect(report.recipients).toHaveLength(2);
    expect(report.schedule?.frequency).toBe('monthly');
  });
});
```

## Why

The builder pattern excels for complex test data:

1. **Fluent API**: Chainable methods read like natural language
2. **Self-documenting**: Method names explain what each configuration does
3. **Flexible**: Build exactly the data needed, nothing more
4. **Discoverable**: IDE autocomplete shows available options
5. **Composable**: Extend base builders for common scenarios
6. **Maintainable**: Changes to data structure are isolated to the builder

When to use builders vs factories:
- **Builders**: Complex objects with many optional configurations
- **Factories**: Simpler objects with few variations
- **Both**: Factories can use builders internally

Builder design tips:
- Return `this` for chainability
- Provide preset methods for common combinations
- Create specialized subclass builders for common scenarios
- Include a `build()` method that validates and returns the final object
