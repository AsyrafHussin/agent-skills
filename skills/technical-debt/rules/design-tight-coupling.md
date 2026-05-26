---
title: Tight Coupling
impact: HIGH
impactDescription: "Changes ripple unpredictably; modules cannot be replaced"
tags: coupling, design, dependencies
---

## Tight Coupling

**Impact: HIGH (Changes ripple unpredictably; modules cannot be replaced)**

Tight coupling means modules depend on each other's internals. A change in one ripples into many, and you can't swap an implementation without rewriting consumers. The symptom: small features take days, simple bug fixes break unrelated tests.

## How to Detect

- Count cross-module imports — modules importing > 10 files from other modules are suspect
- Look for direct instantiation (`new X()`) of cross-module dependencies in business logic
- Search for reaches into private/internal namespaces

```bash
# Find imports from too many other modules
grep -rE "^(import|use|require)" src/orders/ | \
  awk -F'[\"\\\\\\047]' '{print $2}' | sort -u | wc -l
```

## Incorrect

```typescript
// ❌ OrderService reaches deep into Payment, Inventory, and Email internals
import { StripeClient } from '../payments/stripe/client';
import { StripeWebhookSecret } from '../payments/stripe/config';
import { InventoryDB } from '../inventory/db/connection';
import { SesTransport } from '../email/transports/ses';

export class OrderService {
  async place(order: Order) {
    const stripe = new StripeClient(StripeWebhookSecret.value);
    const charge = await stripe.charges.create({ amount: order.total });

    const conn = await InventoryDB.connect();
    await conn.query('UPDATE inventory SET stock = stock - ? WHERE sku = ?', [order.qty, order.sku]);

    const ses = new SesTransport(process.env.AWS_KEY!);
    await ses.send({ to: order.email, body: '...' });
  }
}
```

**Problems:**
- Cannot test without a real Stripe key, DB, and SES credentials
- Switching from Stripe → Adyen means rewriting `place()`
- A schema change in Inventory breaks orders

## Correct

```typescript
// ✅ Depend on interfaces, inject implementations
export class OrderService {
  constructor(
    private payments: PaymentGateway,
    private inventory: InventoryRepository,
    private notifications: NotificationSender,
  ) {}

  async place(order: Order) {
    await this.payments.charge(order.total, order.token);
    await this.inventory.reserve(order.sku, order.qty);
    await this.notifications.orderConfirmed(order);
  }
}
```

**Benefits:**
- Each dependency is a stable interface — internals can change freely
- Tests use in-memory fakes; no credentials needed
- Stripe → Adyen swap is a one-line wiring change

## Remediation Strategy

- **Effort:** M per module boundary
- **When to pay down:** When two modules' release schedules need to decouple, or when a swap is on the roadmap.

Reference: [Robert Martin — Stable Dependencies Principle (Package Principles)](https://en.wikipedia.org/wiki/Package_principles)
