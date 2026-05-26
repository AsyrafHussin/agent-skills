---
title: Undocumented Public APIs
impact: MEDIUM
impactDescription: "Consumers reverse-engineer behaviour; breaking changes blindside everyone"
tags: documentation, api, public-interface
---

## Undocumented Public APIs

**Impact: MEDIUM (Consumers reverse-engineer behaviour; breaking changes blindside everyone)**

A public API without documentation forces consumers to read source or guess. Once they guess wrong and ship, that "wrong guess" becomes the de facto contract — you can no longer change the implementation without breaking them.

## How to Detect

```bash
# TypeScript: public exports without TSDoc
# Requires eslint-plugin-jsdoc installed and registered in your eslint config; the
# rule below is a config-file rule (not a CLI one-liner override):
#   { "plugins": ["jsdoc"], "rules": { "jsdoc/require-jsdoc": ["error", {"publicOnly": true}] } }

# PHP: public methods on public classes without docblocks
# PHPStan does NOT natively flag missing docblocks — use PHP_CodeSniffer with a
# docblock-aware standard (e.g. Squiz.Commenting.FunctionComment), or phpDocumentor's
# validator. PHPStan can still catch missing parameter/return *types* via higher levels.
vendor/bin/phpcs --standard=Squiz --sniffs=Squiz.Commenting.FunctionComment app/

# OpenAPI / REST APIs
# Compare routes/controllers to swagger.json / openapi.yaml — drift is debt
```

## Incorrect

```typescript
// ❌ Exported function with non-obvious behaviour and no docs
export function calculateRefund(order: Order, items: Item[]): RefundResult {
  // ... 80 lines including special cases for partial refunds,
  // restocking fees, expired return windows, original-payment-method routing,
  // and tax recalculation under different jurisdictions
}
```

Questions a consumer can't answer without reading the body:
- Does `items` default to all items if empty? Or refund nothing?
- Are restocking fees deducted? How are they configured?
- What happens if the original payment method is invalidated?
- Are taxes recalculated, or refunded proportionally?

## Correct

```typescript
/**
 * Calculate a refund for some or all items in an order.
 *
 * - If `items` is empty, refunds the entire order.
 * - Restocking fees (configured per category) are deducted from the refund amount.
 * - Taxes are recalculated based on the remaining order subtotal (not refunded proportionally).
 * - Refunds route back to the original payment method; if invalidated, returns
 *   `RefundResult.requiresAlternativeMethod = true` and the caller must collect new instrument.
 * - Refunds are not allowed past the return window (`order.returnsCloseAt`).
 *
 * @throws RefundWindowClosed   — if `order.returnsCloseAt` has passed
 * @throws PartialRefundNotAllowed — if the order's policy disallows partial returns
 *
 * @see docs/refunds.md for business rules and worked examples.
 */
export function calculateRefund(order: Order, items: Item[]): RefundResult {
  // ...
}
```

**Benefits:**
- Consumers depend on documented contract, not observed behaviour
- You can change implementation freely as long as docs hold
- Edge cases are surfaced — usually catching a bug or missing test in the process

## Remediation Strategy

- **Effort:** S per public function (write the doc; if it's hard to write, the function is doing too much)
- **When to pay down:**
  - **Now:** add CI rule (`jsdoc/require-jsdoc`, PHPStan public-API check) so new APIs ship with docs
  - **Gradually:** document existing APIs as you touch them. Don't try to backfill all at once.
- **Side benefit:** writing the doc is the cheapest way to find an API with too many concerns. If you need 3 paragraphs to describe a single function, split it.

Reference: [TSDoc](https://tsdoc.org/) · [PHPDoc](https://docs.phpdoc.org/) · [OpenAPI Specification](https://swagger.io/specification/)
