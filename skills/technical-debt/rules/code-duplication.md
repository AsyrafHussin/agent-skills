---
title: Code Duplication
impact: CRITICAL
impactDescription: "Bug fixes multiply across copies; behaviour drifts silently"
tags: duplication, dry, refactoring
---

## Code Duplication

**Impact: CRITICAL (Bug fixes multiply across copies; behaviour drifts silently)**

Duplicated logic means every bug fix and behaviour change must be made in N places. Copies drift over time, producing inconsistent behaviour that is hard to detect and harder to test.

## How to Detect

```bash
# Find duplicated blocks across the repo (multi-language)
npx jscpd --min-lines 30 --min-tokens 100 src/
npx jscpd --languages php --min-lines 30 app/         # PHP support via jscpd

# PHP-specific (note: sebastian/phpcpd was archived in 2023 — prefer jscpd above)
# vendor/bin/phpcpd app/                              # legacy projects only

# Language-agnostic
git ls-files | xargs sha1sum | sort | uniq -d -w 40   # exact file dupes
```

Threshold: any duplicated block longer than **30 lines** or appearing in **3+ locations** counts as debt.

## Incorrect

```php
// ❌ Bad: same calculation duplicated in two controllers
// app/Http/Controllers/OrderController.php
public function summary(Order $order) {
    $subtotal = $order->items->sum(fn($i) => $i->price * $i->quantity);
    $tax = $subtotal * 0.06;
    $shipping = $subtotal > 100 ? 0 : 15;
    return ['total' => $subtotal + $tax + $shipping];
}

// app/Http/Controllers/CartController.php
public function checkout(Cart $cart) {
    $subtotal = $cart->items->sum(fn($i) => $i->price * $i->quantity);
    $tax = $subtotal * 0.06;
    $shipping = $subtotal > 100 ? 0 : 15;
    return ['total' => $subtotal + $tax + $shipping];
}
```

**Problems:**
- Tax rate change requires editing both files (and any others that copy this)
- One copy can drift (e.g., free-shipping threshold raised in one but not the other)
- No single place to add tests for pricing rules

## Correct

```php
// ✅ Single source of truth
final class PricingCalculator
{
    public function __construct(private TaxRate $tax, private ShippingPolicy $shipping) {}

    public function total(iterable $items): Money
    {
        $subtotal = collect($items)->sum(fn($i) => $i->price * $i->quantity);
        return $subtotal + $this->tax->for($subtotal) + $this->shipping->for($subtotal);
    }
}

// Both controllers inject and call PricingCalculator::total()
```

**Benefits:**
- One place to change tax/shipping rules
- One target for unit tests
- Behaviour cannot drift between call sites

## Remediation Strategy

- **Effort:** S–M (depends on number of duplicates and parameter variation)
- **When to pay down:** Before the next behaviour change touches *any* of the copies — the next edit pays for the refactor.

Reference: [Martin Fowler — Duplicated Code](https://refactoring.com/catalog/extractFunction.html)
