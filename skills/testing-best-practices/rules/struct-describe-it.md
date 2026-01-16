# Describe-It Block Structure

Organize tests using nested describe blocks and it/test functions for clear hierarchy and context.

## Bad Example

```typescript
// Flat structure without organization
test('calculator add positive numbers', () => {
  expect(calculator.add(2, 3)).toBe(5);
});

test('calculator add negative numbers', () => {
  expect(calculator.add(-2, -3)).toBe(-5);
});

test('calculator subtract', () => {
  expect(calculator.subtract(5, 3)).toBe(2);
});

test('calculator divide', () => {
  expect(calculator.divide(10, 2)).toBe(5);
});

test('calculator divide by zero', () => {
  expect(() => calculator.divide(10, 0)).toThrow();
});

test('calculator multiply', () => {
  expect(calculator.multiply(3, 4)).toBe(12);
});
```

## Good Example

```typescript
// Well-organized hierarchical structure
describe('Calculator', () => {
  let calculator: Calculator;

  beforeEach(() => {
    calculator = new Calculator();
  });

  describe('add', () => {
    it('returns sum of two positive numbers', () => {
      expect(calculator.add(2, 3)).toBe(5);
    });

    it('returns sum of two negative numbers', () => {
      expect(calculator.add(-2, -3)).toBe(-5);
    });

    it('returns sum of positive and negative numbers', () => {
      expect(calculator.add(5, -3)).toBe(2);
    });
  });

  describe('subtract', () => {
    it('returns difference between two numbers', () => {
      expect(calculator.subtract(5, 3)).toBe(2);
    });

    it('returns negative when subtracting larger from smaller', () => {
      expect(calculator.subtract(3, 5)).toBe(-2);
    });
  });

  describe('divide', () => {
    it('returns quotient of two numbers', () => {
      expect(calculator.divide(10, 2)).toBe(5);
    });

    it('returns decimal for non-even division', () => {
      expect(calculator.divide(5, 2)).toBe(2.5);
    });

    it('throws DivisionByZeroError when dividing by zero', () => {
      expect(() => calculator.divide(10, 0)).toThrow(DivisionByZeroError);
    });
  });

  describe('multiply', () => {
    it('returns product of two numbers', () => {
      expect(calculator.multiply(3, 4)).toBe(12);
    });

    it('returns zero when multiplying by zero', () => {
      expect(calculator.multiply(5, 0)).toBe(0);
    });
  });
});
```

## Why

The describe-it structure provides multiple benefits:

1. **Logical grouping**: Related tests are grouped together under meaningful contexts
2. **Shared setup**: `beforeEach`/`afterEach` can be scoped to specific groups
3. **Readable output**: Test runners display results in a clear hierarchy
4. **Contextual naming**: Test names can be shorter since context is provided by describe blocks
5. **Easier navigation**: Large test files become easier to scan and understand
6. **Better filtering**: Run specific groups of tests using describe block names

The hierarchy should mirror the structure of what you're testing: class > method > scenario.
