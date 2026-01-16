# Arrange-Act-Assert Pattern

Structure every test using the AAA pattern to ensure clarity and consistency.

## Bad Example

```typescript
// Mixed arrangement and assertions - confusing
test('calculates total price', () => {
  const cart = new ShoppingCart();
  expect(cart.isEmpty()).toBe(true);
  cart.addItem({ name: 'Apple', price: 1.50 });
  cart.addItem({ name: 'Banana', price: 0.75 });
  const discount = new Discount(10);
  cart.applyDiscount(discount);
  expect(cart.getTotal()).toBe(2.025);
  expect(cart.getItemCount()).toBe(2);
});
```

## Good Example

```typescript
// Clear AAA structure
test('calculates total price with discount applied', () => {
  // Arrange
  const cart = new ShoppingCart();
  const discount = new Discount(10);
  cart.addItem({ name: 'Apple', price: 1.50 });
  cart.addItem({ name: 'Banana', price: 0.75 });

  // Act
  cart.applyDiscount(discount);

  // Assert
  expect(cart.getTotal()).toBe(2.025);
});

test('tracks item count correctly', () => {
  // Arrange
  const cart = new ShoppingCart();

  // Act
  cart.addItem({ name: 'Apple', price: 1.50 });
  cart.addItem({ name: 'Banana', price: 0.75 });

  // Assert
  expect(cart.getItemCount()).toBe(2);
});
```

## Why

The AAA pattern makes tests easier to read, understand, and maintain:

1. **Arrange**: Set up the test conditions and inputs
2. **Act**: Execute the behavior being tested
3. **Assert**: Verify the expected outcome

This structure provides a clear narrative flow, making it immediately obvious what is being tested and what the expected behavior is. When tests fail, this pattern helps quickly identify whether the issue is in setup, execution, or verification.
