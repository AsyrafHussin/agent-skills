---
title: Suffix Abuse — *Helper, *Manager, *Util, *Wrapper, *Processor
impact: HIGH
impactDescription: "Catch-all suffixes signal undecided design and pad code with empty abstractions"
tags: naming, suffixes, ai-fingerprint, abstractions
---

## Suffix Abuse — *Helper, *Manager, *Util, *Wrapper, *Processor

**Impact: HIGH (Catch-all suffixes signal undecided design and pad code with empty abstractions)**

`UserHelper`, `OrderManager`, `DataUtil`, `RequestWrapper`, `PaymentProcessor` — these suffixes are the AI's go-to when it doesn't know what to call something. The suffix is non-information: every class is in some sense a "manager" or "helper" or "util". The name describes the noun-form of "code I had to put somewhere".

When a model generates these, it's because:
- The "Helper" class is doing things that belong on the model
- The "Manager" is a procedural blob masquerading as a class
- The "Util" is a kitchen sink that should be split or absorbed into domain types

Real domain code uses domain names: `Pricing`, `RefundPolicy`, `WebhookVerifier` — not `PricingHelper`, `RefundManager`, `WebhookProcessor`.

## Incorrect

```php
// ❌ Suffix abuse — every concern wrapped in *Helper or *Manager

class UserHelper
{
    public function formatName(User $user): string {
        return $user->firstName . ' ' . $user->lastName;
    }
}

class OrderManager
{
    public function processOrder(Order $order): void {
        // ...
    }
}

class PaymentProcessor
{
    public function processPayment(Order $order, string $token): Charge {
        // ...
    }
}

class DataUtil
{
    public static function arrayToCsv(array $rows): string { /* ... */ }
    public static function csvToArray(string $csv): array { /* ... */ }
    public static function snakeToCamel(string $s): string { /* ... */ }
    public static function camelToSnake(string $s): string { /* ... */ }
    // …grows forever
}
```

```typescript
// ❌ Same TS pattern
class StringHelper {
  static capitalize(s: string): string { /* ... */ }
  static slugify(s: string): string { /* ... */ }
}

class ResponseWrapper {
  constructor(private res: ApiResponse) {}
  getData() { return this.res.data; }
}
```

**Why it's slop:**
- `UserHelper.formatName(user)` could just be `user.fullName` on the User model
- `OrderManager.processOrder(order)` is two ways of saying the same thing
- `DataUtil` is a kitchen sink — every utility ends up here, none have a clear home
- The suffix doesn't add information; it admits the author didn't decide what the class is

## Correct

```php
// ✅ Behaviour lives on the domain type; helpers split by concern

class User extends Model
{
    public function getFullNameAttribute(): string
    {
        return "{$this->firstName} {$this->lastName}";
    }
}

// Action class — replaces "OrderManager.processOrder"
final class PlaceOrder
{
    public function __invoke(Order $order, PaymentIntent $payment): void { /* ... */ }
}

// Domain class with a clear single responsibility — replaces "PaymentProcessor"
final class StripePaymentGateway implements PaymentGateway
{
    public function charge(Money $amount, string $token): Charge { /* ... */ }
}

// CSV becomes its own type — replaces the kitchen-sink DataUtil
final class CsvExporter
{
    public function export(iterable $rows, array $headers): string { /* ... */ }
}
```

```typescript
// ✅ Methods on the domain type, or named domain functions
class User {
  get fullName(): string { return `${this.firstName} ${this.lastName}`; }
}

// Free function — capitalize doesn't need a class wrapper
export function capitalize(s: string): string { /* ... */ }
export function slugify(s: string): string { /* ... */ }
```

**Why it reads human:**
- Each class has a specific domain responsibility (CsvExporter exports CSVs; PlaceOrder places orders)
- Behaviour-on-data is on the type that holds the data (`user.fullName`)
- Free functions stand on their own when there's no state to wrap

## When suffixes ARE okay

A handful of suffixes carry real meaning in their conventions:

- **`*Repository`** — DDD-style data access (specific contract)
- **`*Service`** — used sparingly for orchestration that doesn't fit on a domain type
- **`*Controller`** — HTTP entry point (Laravel/Express convention)
- **`*Middleware`** — request pipeline (Express/Laravel convention)
- **`*Gateway`** — external integration boundary (Stripe, AWS)
- **`*Listener`** / `*Observer` — event-handler conventions (Laravel)

`*Helper`, `*Manager`, `*Util`, `*Wrapper`, `*Processor` are the suspect tier.

## Detection

```bash
# Class declarations with suspect suffixes
grep -rEn 'class\s+[A-Z][a-zA-Z]*(Helper|Manager|Util|Utils|Wrapper|Processor|Handler)\b' \
  --include='*.php' --include='*.ts' --include='*.tsx' app/ src/

# Count by suffix to size the problem
grep -rE 'class\s+[A-Z][a-zA-Z]*' --include='*.php' --include='*.ts' app/ src/ \
  | grep -oE '(Helper|Manager|Util|Utils|Wrapper|Processor)\b' \
  | sort | uniq -c | sort -rn
```

A repo with **5+ `*Helper`** or **3+ `*Manager`** classes is almost certainly leaking domain logic into catch-all classes. Refactor by asking "what would I call this if I couldn't use the suffix?"

Reference: [Clean Code — Chapter 2: Meaningful Names](https://www.oreilly.com/library/view/clean-code/9780136083238/) · Internal: [`over-eng-single-method-class`](over-eng-single-method-class.md)
