---
title: Useless Wrapper Functions
impact: HIGH
impactDescription: "Adds indirection layer for no behavioural gain; callers chase one extra hop to find real code"
tags: over-engineering, wrappers, indirection, ai-fingerprint
---

## Useless Wrapper Functions

**Impact: HIGH (Adds indirection layer for no behavioural gain; callers chase one extra hop to find real code)**

A function that exists only to delegate to another function, with no added validation, transformation, or context. AI generates these because it pattern-matches on "wrap external dependencies for testability" but applies it indiscriminately.

A wrapper earns its place when it adds **something** the underlying call doesn't: validation, default arguments, error normalisation, logging, retry logic, type narrowing. A wrapper that just forwards arguments is pure indirection.

## Incorrect

```php
// ❌ Wrappers that just delegate

class UserService
{
    public function __construct(private UserRepository $repo) {}

    public function findUser(int $id): ?User
    {
        return $this->repo->find($id);
    }

    public function deleteUser(int $id): void
    {
        $this->repo->delete($id);
    }

    public function createUser(array $data): User
    {
        return $this->repo->create($data);
    }
}
```

```typescript
// ❌ Same TS pattern
class StripeWrapper {
  private stripe: Stripe;
  constructor(stripe: Stripe) { this.stripe = stripe; }

  charge(amount: number, token: string) {
    return this.stripe.charges.create({ amount, source: token });
  }

  refund(chargeId: string) {
    return this.stripe.refunds.create({ charge: chargeId });
  }
}
```

**Why it's slop:**
- `UserService::findUser($id)` does exactly what `$repo->find($id)` does
- Every caller now goes through TWO layers (`Service → Repo`) to find the actual logic
- Adding a parameter requires touching both files
- Tests of `UserService` mostly verify "did we call the repo correctly?" — they don't verify business behaviour

## Correct

```php
// ✅ Drop the wrapper; use the underlying type directly

// Controller depends on the Repository or Model directly
public function show(int $id, UserRepository $users): UserResource
{
    return new UserResource($users->findOrFail($id));
}
```

## When a wrapper IS worth keeping

A wrapper earns its place when it adds something real:

```php
// ✅ Wrapper that normalises errors + adds retry — real value
final class ResilientStripeGateway
{
    public function __construct(private Stripe $stripe) {}

    public function charge(Money $amount, string $token): Charge
    {
        return retry(times: 3, sleepMs: 200, callback: function () use ($amount, $token) {
            try {
                return $this->stripe->charges->create([
                    'amount' => $amount->cents(),
                    'currency' => $amount->currency()->code(),
                    'source' => $token,
                ]);
            } catch (CardException $e) {
                throw new PaymentDeclined($e->getMessage(), $e);   // domain exception
            }
        });
    }
}
```

This wrapper adds:
- Retry on transient failures
- Domain-specific exception (`PaymentDeclined`, not `CardException`)
- Type-safe `Money` input rather than raw cents + currency strings

That's worth the file.

**Why it reads human:**
- Wrappers exist when they add value; non-wrappers don't pad the layer count
- Caller can see "this charge call is resilient" because the type is `ResilientStripeGateway`
- No mystery hops to chase

## Detection

```bash
# Heuristic: methods that are one-liners and just call $this->dependency->method($args)
# PHP — find public methods whose body is exactly one delegate call
grep -rEn -A1 '^\s+public function [a-zA-Z]+\([^)]*\)' --include='*.php' app/ | \
  awk '/public function/{name=$0; next} /^\s+return \$this->[a-zA-Z]+->[a-zA-Z]+\(/{print name; print $0; print "---"}'

# TS — same heuristic
grep -rEn -A1 '^\s+(public |async )?[a-zA-Z]+\([^)]*\):' --include='*.ts' src/ | \
  awk '/[a-zA-Z]+\(/{name=$0; next} /return this\.[a-zA-Z]+\./{print name; print $0; print "---"}'
```

The cleanest signal: **call-sites**. If a wrapper is called from exactly one place, it's almost certainly slop. Inline it.

```bash
# For each public method in Service.php, count call sites
# (rough — use phpstorm 'Find Usages' for accurate results)
```

Reference: [Sandi Metz — The Wrong Abstraction](https://sandimetz.com/blog/2016/1/20/the-wrong-abstraction) · [Martin Fowler — Inline Function](https://refactoring.com/catalog/inlineFunction.html) · Internal: [`over-eng-single-method-class`](over-eng-single-method-class.md), [`over-eng-premature-interface`](over-eng-premature-interface.md)
