---
title: Shotgun Surgery
impact: HIGH
impactDescription: "One conceptual change forces edits across many files"
tags: shotgun-surgery, design, refactoring
---

## Shotgun Surgery

**Impact: HIGH (One conceptual change forces edits across many files)**

A change is "shotgun surgery" when adding one concept (a new payment method, a new locale, a new currency) requires edits in 5+ files. The information is fragmented; whoever makes the change is guaranteed to miss a spot.

## How to Detect

```bash
# Find files commonly changed together (co-change hotspots)
git log --since='6 months ago' --name-only --pretty=format:'COMMIT' | \
  awk '/COMMIT/{print "---"; next} {print}' | \
  # process: count file pairs appearing in the same commit
  # any pair > 10 co-changes is a shotgun-surgery suspect

# Search for switch/if chains on the same enum across multiple files
grep -rEn "case PaymentMethod::|=== ['\"]stripe['\"]" --include='*.{php,ts}'
```

## Incorrect

```typescript
// ❌ Adding a new payment method "klarna" requires editing 6 files

// src/payments/types.ts
type PaymentMethod = 'stripe' | 'paypal' | 'wire';

// src/payments/validator.ts
if (m === 'stripe') { /* */ } else if (m === 'paypal') { /* */ } else if (m === 'wire') { /* */ }

// src/payments/feeCalculator.ts
const FEES = { stripe: 0.029, paypal: 0.034, wire: 0 };

// src/payments/iconUrl.ts
const ICONS = { stripe: '...', paypal: '...', wire: '...' };

// src/payments/displayName.ts
const NAMES = { stripe: 'Card', paypal: 'PayPal', wire: 'Bank wire' };

// src/payments/router.ts
switch (m) { case 'stripe': return new StripeClient(); /* ... */ }
```

**Problems:**
- Adding Klarna means hunting through 6+ files — easy to miss one
- New engineers cannot find "what makes a payment method valid"
- Tests don't catch a forgotten file until production

## Correct

```typescript
// ✅ One Strategy per method — adding Klarna is one new file
// src/payments/methods/StripeMethod.ts
export const StripeMethod: PaymentMethod = {
  id: 'stripe',
  displayName: 'Card',
  iconUrl: '/icons/stripe.svg',
  feeRate: 0.029,
  validate: (payload) => /* ... */,
  charge: (amount, token) => new StripeClient().charge(amount, token),
};

// src/payments/methods/index.ts
export const METHODS = [StripeMethod, PayPalMethod, WireMethod /*, KlarnaMethod */];
```

**Benefits:**
- Adding Klarna = one new file + one export — nothing else changes
- All knowledge about a method lives in one place
- TypeScript types prevent "forgot a branch" bugs

## Remediation Strategy

- **Effort:** M (collect scattered knowledge into one type per concept)
- **When to pay down:** Before the *next* new variant is added — the pain is highest right when adding one.

Reference: [Refactoring Guru — Shotgun Surgery](https://refactoring.guru/smells/shotgun-surgery)
