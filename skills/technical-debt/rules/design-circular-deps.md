---
title: Circular Dependencies
impact: HIGH
impactDescription: "Indicates broken module boundaries; breaks tree-shaking and isolation"
tags: circular-deps, modules, design
---

## Circular Dependencies

**Impact: HIGH (Indicates broken module boundaries; breaks tree-shaking and isolation)**

Module A imports B, B imports A. Either the boundary is wrong, or one module is misplaced. Cycles break dead-code elimination, make modules impossible to test in isolation, and cause runtime initialization order bugs in many languages.

## How to Detect

```bash
# JavaScript / TypeScript
npx madge --circular --extensions ts,tsx src/

# PHP (architectural rules, including cycle detection between layers/modules)
vendor/bin/deptrac analyse                  # qossmic/deptrac
# or:  vendor/bin/phparkitect check
```

Threshold: **zero cycles** is the only acceptable target. Even one cycle indicates a layering problem.

## Incorrect

```typescript
// ❌ user/index.ts imports order/, order/index.ts imports user/
// src/user/index.ts
import { Order } from '../order';
export class User {
  orders: Order[] = [];
  totalSpent() { return this.orders.reduce((s, o) => s + o.total, 0); }
}

// src/order/index.ts
import { User } from '../user';
export class Order {
  customer: User;
  total: number;
}
```

**Problems:**
- Either module fails to initialize cleanly under some bundlers (one side is `undefined` at import time)
- Cannot extract `user` or `order` into a separate package
- A test of `user` necessarily pulls in `order`

## Correct

```typescript
// ✅ Option A: extract shared types to a third module
// src/shared/types.ts
export interface UserRef { id: string; }
export interface OrderRef { id: string; total: number; }

// src/user/index.ts → imports types only
// src/order/index.ts → imports types only

// ✅ Option B: invert the dependency — let one own the relationship
// src/order/index.ts owns the customer reference;
// user no longer knows about order. totalSpent() lives in an OrderService.
```

**Benefits:**
- No initialization-order bugs
- Each module can be packaged independently
- Tests load the minimum surface

## Remediation Strategy

- **Effort:** M (mechanical once you decide the direction)
- **When to pay down:** As soon as `madge`/equivalent reports a new cycle. Adding tests across the boundary first protects the refactor.

Reference: [Madge — Circular Dependencies](https://github.com/pahen/madge)
