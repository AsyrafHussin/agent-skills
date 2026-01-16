# Given-When-Then Pattern

Use behavior-driven development (BDD) style for tests that describe user-facing behavior.

## Bad Example

```typescript
// Technical, implementation-focused test
test('order processing', () => {
  const order = new Order();
  order.items = [{ id: 1, price: 100 }];
  order.customerId = 'cust-123';
  const result = order.process();
  expect(result.status).toBe('processed');
  expect(result.total).toBe(100);
  expect(emailService.send).toHaveBeenCalled();
});
```

## Good Example

```typescript
// BDD-style with Given-When-Then structure
describe('Order Processing', () => {
  describe('given a customer with items in their cart', () => {
    let order: Order;
    let customer: Customer;

    beforeEach(() => {
      // Given
      customer = CustomerFactory.create({ id: 'cust-123' });
      order = OrderFactory.createWithItems([
        { name: 'Widget', price: 100 }
      ]);
      order.assignToCustomer(customer);
    });

    describe('when the order is submitted', () => {
      let result: OrderResult;

      beforeEach(() => {
        // When
        result = order.submit();
      });

      it('then the order status is confirmed', () => {
        expect(result.status).toBe('confirmed');
      });

      it('then the order total reflects item prices', () => {
        expect(result.total).toBe(100);
      });

      it('then a confirmation email is sent to the customer', () => {
        expect(emailService.send).toHaveBeenCalledWith(
          expect.objectContaining({
            to: customer.email,
            template: 'order-confirmation'
          })
        );
      });
    });

    describe('when the order is submitted with an expired promotion', () => {
      beforeEach(() => {
        order.applyPromotion(ExpiredPromotion.create());
      });

      it('then the order is rejected with promotion error', () => {
        expect(() => order.submit()).toThrow(PromotionExpiredError);
      });
    });
  });

  describe('given a customer with an empty cart', () => {
    it('then submitting order throws EmptyCartError', () => {
      const order = OrderFactory.createEmpty();
      expect(() => order.submit()).toThrow(EmptyCartError);
    });
  });
});
```

## Why

The Given-When-Then pattern provides several advantages:

1. **Business language**: Tests read like specifications that non-developers can understand
2. **Clear scenarios**: Each test scenario is explicit about its preconditions
3. **Behavior focus**: Tests describe what the system does, not how it does it
4. **Living documentation**: Tests serve as executable requirements
5. **Better collaboration**: Product owners and QA can review and suggest test cases
6. **Scenario coverage**: The structure naturally leads to thinking about different scenarios

This pattern is especially valuable for:
- User-facing features
- Business logic with complex rules
- Acceptance tests
- Integration tests

The pattern maps directly to user stories: "Given [context], when [action], then [outcome]"
