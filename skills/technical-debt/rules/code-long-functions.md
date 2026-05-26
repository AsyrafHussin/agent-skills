---
title: Long Functions and Methods
impact: CRITICAL
impactDescription: "Long methods hide multiple responsibilities and resist testing"
tags: srp, function-length, refactoring
---

## Long Functions and Methods

**Impact: CRITICAL (Long methods hide multiple responsibilities and resist testing)**

A function longer than ~50 lines almost always does more than one thing. It cannot be unit-tested in isolation, cannot be named accurately, and is the most common location for "scary code nobody touches".

## How to Detect

```bash
# Find functions over 50 lines (rough heuristic)
grep -rEn 'function |def |func ' --include='*.{php,ts,js,py,go}' | \
  awk -F: '{ print $1 ":" $2 }' | \
  while read line; do : ; done   # combine with editor stats or:

# PHP
vendor/bin/phpmd app text codesize  # reports ExcessiveMethodLength (default 100; lower to 50 via custom ruleset)
#   <property name="minimum" value="50"/>   in the ExcessiveMethodLength rule reference

# TS / JS
npx eslint . --rule 'max-lines-per-function: ["error", 50]'
```

Threshold: **method > 50 lines** OR **function > 50 lines** (excluding comments and blank lines).

## Incorrect

```php
// ❌ 180-line controller action doing fetch, validate, transform, persist, notify
public function checkout(Request $request)
{
    // 20 lines of input validation
    $data = $request->validate([ /* ... */ ]);

    // 30 lines of cart calculation
    $cart = Cart::findOrFail($data['cart_id']);
    $subtotal = 0;
    foreach ($cart->items as $item) { /* ... */ }
    $tax = /* ... */;

    // 25 lines of payment processing
    $charge = Stripe::charges()->create(/* ... */);
    if ($charge->status !== 'succeeded') { /* ... */ }

    // 40 lines of order creation + side effects
    $order = Order::create(/* ... */);
    foreach ($cart->items as $item) { /* ... */ }

    // 20 lines of notifications
    Mail::to($cart->user)->send(new OrderConfirmation($order));
    Slack::notify('#sales', /* ... */);

    return response()->json(/* ... */);
}
```

**Problems:**
- Cannot test calculation logic without mocking Stripe and Mail
- Reviewer must understand the entire flow to verify a one-line change
- Side effects (mail, Slack) hidden inside a request handler

## Correct

```php
// ✅ Each responsibility extracted; controller orchestrates only
public function checkout(CheckoutRequest $request, CheckoutService $service)
{
    $order = $service->process(
        cart: Cart::findOrFail($request->validated('cart_id')),
        paymentToken: $request->validated('payment_token'),
    );
    return new OrderResource($order);
}

// CheckoutService::process() is itself a short orchestrator that calls:
//   PricingCalculator, PaymentGateway, OrderRepository, NotificationDispatcher
```

**Benefits:**
- Controller is 5 lines, trivially testable as an HTTP wrapper
- Pricing, payment, persistence, notifications each have a focused unit test
- Adding a new notification channel changes one class

## Remediation Strategy

- **Effort:** S–M (Extract Method is mechanical; the hard part is naming)
- **When to pay down:** Whenever you need to add a feature inside a long method, extract first.

Reference: [Refactoring — Extract Function](https://refactoring.com/catalog/extractFunction.html)
