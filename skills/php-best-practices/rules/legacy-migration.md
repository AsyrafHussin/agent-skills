---
title: Legacy Code Migration Strategy
impact: HIGH
impactDescription: Incremental legacy codebase migration
tags: php, legacy, migration, refactoring
---

# Legacy Code Migration Strategy

## Why it matters

Rewriting a working legacy codebase in one large branch is one of the highest-risk activities in software engineering. The branch drifts from production, tests are missing or wrong, and the rewrite ships with a new set of bugs while the old bugs are rediscovered. Incremental migration with a safety net produces visible, verifiable progress without risking the system that is currently earning money.

## Rule

Establish a safety net first, then modernize in small automated steps. Never rewrite a working class from scratch. Use the Strangler Fig pattern to isolate legacy code behind typed interfaces so new code replaces it module by module without disrupting callers. Start with the smallest modernization seam that resolves the real problem: a typed boundary, value object, adapter, explicit method, or narrow request cleanup. Stop after the smallest safe slice unless more is clearly required.

## Bad

```php
<?php

// DON'T: big-bang rewrite
// Copy the class, rewrite from scratch, merge weeks later, pray tests cover it

// DON'T: mix legacy and modern in the same function boundary
function processOrder($id)           // no types
{
    $order = legacy_get_order($id);  // old procedural call
    return new ModernOrderDTO(...);  // new code — incompatible styles, no type contract
}

// DON'T: suppress errors to make migration easier
$result = @legacy_function();        // hides failures that should surface

// DON'T: delete tests to speed up the migration
```

## Better

```php
<?php

declare(strict_types=1);

// Step 1: Add strict_types and basic types to the most-called class
// without changing behavior — safe mechanical improvement
final class UserService
{
    public function __construct(
        private readonly \PDO $db,
    ) {}

    public function getUser(int $id): ?array
    {
        $stmt = $this->db->prepare('SELECT * FROM users WHERE id = ?');
        $stmt->execute([$id]);
        $row = $stmt->fetch(\PDO::FETCH_ASSOC);
        return $row !== false ? $row : null;
    }
}
```

## Best

```php
<?php

declare(strict_types=1);

// Phase 1: Safety net — capture current state before touching anything
// $ vendor/bin/phpstan analyse --generate-baseline phpstan-baseline.neon
// $ vendor/bin/pest --stop-on-failure  (tests must pass before migration starts)

// Phase 2: Automated syntax modernization via Rector (dry-run first)
// $ vendor/bin/rector process src --dry-run
// rector.php:
// return RectorConfig::configure()
//     ->withPaths([__DIR__ . '/src'])
//     ->withSets([LevelSetList::UP_TO_PHP_81, SetList::TYPE_DECLARATION]);

// Phase 3: Incremental type coverage — BEFORE
class UserService
{
    function getUser($id)
    {
        return $this->db->query("SELECT * FROM users WHERE id = $id");  // SQL injection
    }
}

// Phase 3: Incremental type coverage — AFTER
final class UserService
{
    public function __construct(
        private readonly UserRepository $repository,
    ) {}

    public function getUser(int $id): ?User
    {
        return $this->repository->find(new UserId($id));
    }

    /** @param array{name: string, email: string} $data */
    public function createUser(array $data): User
    {
        return $this->repository->create(
            name:  new UserName($data['name']),
            email: new Email($data['email']),
        );
    }
}

// Phase 4: Strangler Fig — wrap legacy behind a typed interface
interface LegacyOrderGateway
{
    /** @return array{id: int, total: float, status: string} */
    public function fetchOrder(int $id): array;
}

// Adapter delegates to legacy code
final class LegacyOrderAdapter implements LegacyOrderGateway
{
    public function fetchOrder(int $id): array
    {
        $raw = legacy_get_order($id);   // still calls old code
        return [
            'id'     => (int)    $raw['order_id'],
            'total'  => (float)  $raw['order_total'],
            'status' => (string) $raw['order_status'],
        ];
    }
}

// New service depends on the interface, not the legacy code
final class OrderService
{
    public function __construct(
        private readonly LegacyOrderGateway $gateway,
    ) {}

    public function getOrder(int $id): Order
    {
        $data = $this->gateway->fetchOrder($id);
        return Order::fromArray($data);
    }
}

// When ready, swap LegacyOrderAdapter for a real repository —
// OrderService does not change at all.
```

## Tracking migration progress

Commit a migration checklist to the repository to make progress visible:

```markdown
## Migration Progress

- [x] PHPStan baseline captured (1,423 violations at level 0)
- [x] Rector applied — PHP 8.1 syntax transformations
- [x] `declare(strict_types=1)` added to all files
- [ ] PHPStan level raised to 3 (current: 1)
- [ ] SQL injection: 12 of 34 query sites migrated to PDO
- [ ] Value objects: UserId, Email, Money created; 8 of 25 services migrated
- [ ] God class `OrderManager` (1,200 lines) → extract 4 focused classes
```

## Priority order for what to fix first

| Priority | Fix | Why |
|---|---|---|
| 1 | Add `declare(strict_types=1)` | Foundation for all subsequent type work |
| 2 | Fix SQL injection (prepared statements) | Security — immediate risk |
| 3 | Add return types to public methods | Unblocks PHPStan analysis |
| 4 | Replace untyped `array` params with shapes or value objects | Eliminates `mixed` propagation |
| 5 | Extract classes from god classes | Enables focused unit tests |
| 6 | Replace `@` suppression with proper error handling | Unmasks hidden failures |
| 7 | Raise PHPStan level one step at a time | Validates all prior work |

## Exceptions / trade-offs
- **Legacy code with no tests**: write characterization tests (golden-master tests) before touching anything. Tests that prove "this is what it currently does" are better than no tests.
- **Tight deadlines**: a PHPStan baseline is not tech debt — it is an honest inventory. Generate it, commit it, and shrink it over time. Leaving it at the initial size is tech debt.
- **Third-party integrations**: wrap them behind typed interfaces from day one. Do not let their untyped APIs bleed into your domain.

## Modernization slice checklist

- Name the observed symptom before refactoring.
- Identify the immediate cause and the structural root cause.
- Pick one seam to improve instead of broad aesthetic cleanup.
- Add characterization or regression coverage before widening a risky slice.
- Re-run the narrowest validation for the touched seam, then stop.

## Static-analysis notes

- **`phpstan-baseline.neon`**: generated with `--generate-baseline`, acknowledged violations that do not block CI. Shrink it sprint by sprint — treat any baseline growth as a regression.
- **PHPStan level progression**: start at level 0 for legacy, fix violations, raise by 1. The level number is an objective metric of type-safety maturity.
- **Rector** automates ~80% of mechanical transforms (constructor promotion, match expressions, typed properties). Always dry-run first; review the diff before applying.
- **`reportUnmatchedIgnoredErrors: true`** in `phpstan.neon` ensures stale `@phpstan-ignore` comments are flagged when the underlying issue is fixed.

## Related topics

- [tooling-phpstan-cs-fixer.md](tooling-phpstan-cs-fixer.md) — the PHPStan + php-cs-fixer loop used throughout migration
- [type-strict-mode.md](type-strict-mode.md) — first migration step: add `declare(strict_types=1)` to every file
- [sec-sql-prepared.md](sec-sql-prepared.md) — second migration step: fix SQL injection
- [design-value-objects.md](design-value-objects.md) — replace primitive arrays with typed value objects
- [error-never-suppress.md](error-never-suppress.md) — remove `@` error suppression during migration
