---
title: Closing-Brace Labels
impact: MEDIUM
impactDescription: "Relic of older training corpora; clear AI fingerprint that modern editors make pointless"
tags: comments, closing-brace, ai-fingerprint, style
---

## Closing-Brace Labels

**Impact: MEDIUM (Relic of older training corpora; clear AI fingerprint that modern editors make pointless)**

`} // end function` / `} // end if block` / `} // close foreach` are a strong AI tell. The pattern comes from older C/COBOL/Verilog training data where blocks could span hundreds of lines and editors didn't fold scope. Modern PHP, TypeScript, and any editor with bracket-matching makes the label pure noise.

If your function is so long you need a label on its closing brace, the real fix is to split the function — not annotate the brace.

## Incorrect

```php
// ❌ Labels on closing braces

class OrderService
{
    public function process(Order $order): void
    {
        if ($order->isPaid()) {
            foreach ($order->items as $item) {
                if ($item->requiresShipping()) {
                    $this->ship($item);
                } // end if requiresShipping
            } // end foreach items
        } // end if isPaid
    } // end function process
} // end class OrderService
```

```typescript
// ❌ TS variant
function processOrder(order: Order): void {
  if (order.isPaid()) {
    order.items.forEach(item => {
      if (item.requiresShipping()) {
        ship(item);
      } // end if requiresShipping
    }); // end forEach
  } // end if isPaid
} // end function processOrder
```

**Why it's slop:**
- Every modern IDE highlights the matching opener when the cursor is on a brace
- The labels add bytes for zero information
- A function that needed labels for clarity is a function that needs to be split
- Pattern is recognisably training-data noise (PHP-from-2008, Verilog, COBOL)

## Correct

```php
// ✅ Either small enough not to need labels, OR split for clarity

class OrderService
{
    public function process(Order $order): void
    {
        if (!$order->isPaid()) {
            return;
        }
        $this->shipEligibleItems($order);
    }

    private function shipEligibleItems(Order $order): void
    {
        foreach ($order->items->filter->requiresShipping() as $item) {
            $this->ship($item);
        }
    }
}
```

```typescript
// ✅ Smaller scopes, no labels needed
function processOrder(order: Order): void {
  if (!order.isPaid()) return;
  shipEligibleItems(order);
}

function shipEligibleItems(order: Order): void {
  order.items
    .filter(i => i.requiresShipping())
    .forEach(ship);
}
```

**Why it reads human:**
- Short functions; the closing brace is visually close to the opener
- No annotation noise; editor handles brace matching
- If a developer felt the need to label, they'd split the function instead

## Detection

```bash
# Closing brace followed by an "end" / function/class/if/foreach annotation comment
grep -rEn '^\s*\}\s*(\)\s*)?\s*(//|#).*\b(end|close|finish)\b' \
  --include='*.php' --include='*.ts' --include='*.tsx' --include='*.js' --include='*.jsx' \
  app/ src/

# Specific PHP closing-tag labels
grep -rEn '^\s*\}\s*//.*(end (function|class|if|foreach|for|while|switch))' --include='*.php' app/
```

A single hit is borderline; **three or more in one file is an unmistakable AI fingerprint.**

Reference: [Code Complete — Chapter 32 on Self-Documenting Code](https://www.microsoftpressstore.com/store/code-complete-9780735619678) · Internal: [`comments-narration`](comments-narration.md)
