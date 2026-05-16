---
title: Placeholder Comments Left In
impact: CRITICAL
impactDescription: "Reveals AI-generated code that was never completed; ships unfinished work as 'done'"
tags: comments, placeholder, todo, ai-fingerprint
---

## Placeholder Comments Left In

**Impact: CRITICAL (Reveals AI-generated code that was never completed; ships unfinished work as 'done')**

Common AI-generated placeholders that should never reach `main`:

```
// implementation
// implementation here
// your code here
// helper function
// TODO: implement
// TODO: implement this
// TODO: add error handling
// TODO: implement validation
// FIXME: figure out edge case
// HACK: temporary
// placeholder
// add your logic here
```

These mean the AI produced scaffolding the developer was supposed to fill in. If they reach the PR, they signal the author shipped without reading.

Distinct from properly-tagged debt markers — a `// TODO(#1234, asyraf): rewrite after Stripe v2 migration` with an owner, ticket, and concrete unblocking condition is *not* slop. That's tracked debt.

## Incorrect

```php
// ❌ Placeholder comments shipped as 'done'

public function calculateRefund(Order $order): Money
{
    // TODO: implement
    return Money::zero();
}

public function process(Order $order): void
{
    // Validate the order
    // TODO: add validation

    $this->charge($order);

    // helper function
    // TODO: implement error handling
}
```

```typescript
// ❌ Same in TS
function exportUsers(): User[] {
  // your code here
  return [];
}

function handlePayment(token: string) {
  // implementation
  // TODO: implement
}
```

**Why it's slop:**
- The function name promises a behaviour the body doesn't deliver
- Reviewers usually skip past `// TODO` without realising the function is empty
- These slip through tests because the placeholder body type-checks
- If genuinely intended as future work, it belongs in the issue tracker, not the source file

## Correct

```php
// ✅ Either implement, or fail loud, or remove the method entirely

public function calculateRefund(Order $order): Money
{
    throw new BadMethodCallException(
        'Refund calculation not yet implemented — see #1842 for the policy spec.'
    );
}

// ✅ Or genuine, tracked work-in-progress with owner + ticket:
// TODO(#1842, asyraf): cap refunds at original-charge-minus-shipping
public function calculateRefund(Order $order): Money
{
    return $order->total->subtract($order->shippingCost);
}
```

```typescript
// ✅ Fail loud so the gap is impossible to ship past
function exportUsers(): User[] {
  throw new Error('exportUsers: not implemented (see #420)');
}
```

**Why it reads human:**
- An empty function body that throws is impossible to merge by accident; tests will catch it
- A tracked `TODO` with `(#issue, @owner)` is real debt, not abandoned scaffolding
- A human author either implements the function or explicitly fails — they don't ship "Returns nothing for now"

## Detection

```bash
# Common AI placeholders — match only when prefixed by a comment marker
# (without the required prefix, the regex would flood on class names like "ServiceImplementation").
grep -rEn --include='*.php' --include='*.ts' --include='*.tsx' --include='*.js' --include='*.jsx' \
  '(//|/\*|#)\s*(implementation|implementation here|your code here|helper function|placeholder|add your logic|add error handling|add validation)\b' \
  app/ src/ 2>/dev/null

# Bare TODO/FIXME (without ticket reference)
grep -rEn --include='*.php' --include='*.ts' --include='*.tsx' --include='*.js' --include='*.jsx' \
  '(TODO|FIXME)([^(#]|$)' app/ src/ 2>/dev/null | grep -v '#[0-9]'

# Combined: TODOs lacking issue references — CI gate
NEW=$(git diff --name-only --diff-filter=ACM origin/main...HEAD)
BARE=$(echo "$NEW" | xargs grep -EnH '\b(TODO|FIXME)\b' 2>/dev/null | grep -v '#[0-9]')
test -z "$BARE" || { echo "Bare TODO/FIXME found — add (#issue, @owner):"; echo "$BARE"; exit 1; }
```

See also `technical-debt`'s `process-todo-fixme-aging` rule — the broader policy for tracked debt markers.

Reference: Internal: [`comments-narration`](comments-narration.md)
