---
title: Premature Interface (One Implementation, No Plan for a Second)
impact: HIGH
impactDescription: "Adds indirection nobody asked for; doubles the surface for every change"
tags: over-engineering, interfaces, abstractions, ai-fingerprint
---

## Premature Interface (One Implementation, No Plan for a Second)

**Impact: HIGH (Adds indirection nobody asked for; doubles the surface for every change)**

AI defaults to "enterprise-grade" because its training distribution is enterprise-grade. The result: every service gets an interface and one implementation, with no second implementation in the foreseeable future. The interface is pure ceremony — every change updates two files, every test mocks the interface, every reader chases two clicks to find the actual code.

**Interfaces earn their place when:**
- There IS a second implementation (e.g., `StripeGateway` and `AdyenGateway`)
- There IS a planned second implementation (in this quarter's roadmap)
- The implementation is being swapped in tests with a fake (and even then, dependency injection of the concrete type is often fine)

**Otherwise: write the concrete type. Add the interface when a second implementation actually shows up.** YAGNI is real.

## Incorrect

```php
// ❌ Interface with exactly one implementation, no plan for a second

interface UserRepositoryInterface
{
    public function find(int $id): ?User;
    public function save(User $user): void;
    public function delete(int $id): void;
}

final class UserRepository implements UserRepositoryInterface
{
    public function find(int $id): ?User { /* ... */ }
    public function save(User $user): void { /* ... */ }
    public function delete(int $id): void { /* ... */ }
}

// Binding in a service provider, mostly to satisfy the interface
$this->app->bind(UserRepositoryInterface::class, UserRepository::class);
```

```typescript
// ❌ Same TS pattern
interface IOrderService {
  placeOrder(payload: OrderPayload): Promise<Order>;
  cancelOrder(orderId: string): Promise<void>;
}

class OrderService implements IOrderService {
  async placeOrder(payload: OrderPayload): Promise<Order> { /* ... */ }
  async cancelOrder(orderId: string): Promise<void> { /* ... */ }
}
```

**Why it's slop:**
- Every method declared in two files
- `find` / `save` / `delete` change — both interface AND implementation must be updated; nothing prevents drift
- Reader hits `UserRepositoryInterface` in a controller, has to navigate through to find the actual code
- The `I*` prefix or `*Interface` suffix is itself a tell — clean naming uses domain words without the marker

## Correct

```php
// ✅ Concrete class until you need a second implementation

final class UserRepository
{
    public function find(int $id): ?User { /* ... */ }
    public function save(User $user): void { /* ... */ }
    public function delete(int $id): void { /* ... */ }
}

// Controller depends on the concrete class — Laravel auto-resolves it
public function show(int $id, UserRepository $users): UserResource
{
    $user = $users->find($id) ?? throw new NotFoundException;
    return new UserResource($user);
}
```

```typescript
// ✅ Concrete class
class OrderService {
  async placeOrder(payload: OrderPayload): Promise<Order> { /* ... */ }
  async cancelOrder(orderId: string): Promise<void> { /* ... */ }
}
```

**When a second implementation arrives** — e.g., you need a `CachedUserRepository` — *then* extract the interface. By that point you'll know what shape it should have because you have two concrete examples.

**Why it reads human:**
- Concrete class is reachable in one click
- No double-maintenance
- Adding a second implementation is a deliberate refactor, not a default scaffold

## Tests: don't create an interface just to mock

If you need to swap the dependency in tests, prefer:

1. **Inject the concrete class; instantiate with fakes** (e.g., `UserRepository` with an in-memory `Database` connection)
2. **Use Laravel's container-bound fake** (`$this->mock(UserRepository::class, ...)`) — no interface needed
3. **Method-level mocks** via PHPUnit / Vitest

An interface created *only* to mock the dependency in tests is also slop. Real interfaces serve production code paths.

## Detection

```bash
# Interfaces in the codebase
grep -rn '^interface\s\|^export interface' --include='*.php' --include='*.ts' app/ src/ \
  | wc -l

# For each interface, count implementations — flag interfaces with exactly 1
# (manual review with `phpstorm` or VSCode Go to Implementation)

# PHP heuristic: interfaces with 'Interface' suffix or 'I*' prefix
grep -rEn '^interface\s+(I[A-Z]|[A-Z][a-zA-Z]+Interface)\b' --include='*.php' app/

# TS heuristic
grep -rEn '^export interface I[A-Z]|^interface I[A-Z]' --include='*.ts' --include='*.tsx' src/
```

Reference: [Martin Fowler — YAGNI](https://martinfowler.com/bliki/Yagni.html) · [Sandi Metz — The Wrong Abstraction](https://sandimetz.com/blog/2016/1/20/the-wrong-abstraction) · Internal: [`over-eng-useless-wrapper`](over-eng-useless-wrapper.md), [`naming-suffix-abuse`](naming-suffix-abuse.md)
