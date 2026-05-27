---
title: PSR Namespace Usage
impact: HIGH
impactDescription: PSR standards and code structure conventions
tags: php, psr, coding-standards
---

# PSR Namespace Usage

## Why it matters
Namespaces that mirror directories without carrying domain meaning (`App\Http\Controllers\Api\V1`) describe where a file lives, not what it is. When the directory is reorganized, every import breaks. Namespaces should express bounded context and domain concepts so that `use App\Payment\Invoice` is self-documenting and stable across refactors.

## Rule
Namespaces should reflect domain or bounded context, not directory trivia. `App\Payment\Invoice` is a domain concept. `App\Http\Controllers\Api\V1\PaymentController` is a locator path — valid as a technical namespace but should not be mistaken for domain design. Always import at the top; never use fully qualified names inline.

## Bad

```php
<?php

// No namespace — pollutes global scope
class UserService {}

// Redundant prefix
namespace MyApp;
class MyApp_User {}

// Fully qualified names used inline — cluttered and fragile
final class OrderService
{
    public function find(int $id): \App\Domain\Order\Order
    {
        return $this->repo->find($id);
    }

    public function create(\App\Domain\Order\CreateOrderDto $dto): \App\Domain\Order\Order
    {
        // ...
    }
}
```

## Better

```php
<?php

declare(strict_types=1);

namespace App\Domain\Order;

use DateTimeImmutable;

final class Order
{
    public function __construct(
        private readonly OrderId $id,
        private readonly CustomerId $customerId,
        private readonly DateTimeImmutable $createdAt,
    ) {}
}
```

```php
<?php

declare(strict_types=1);

namespace App\Application\Order;

use App\Domain\Order\CreateOrderDto;
use App\Domain\Order\Order;
use App\Domain\Order\OrderRepository;

final class OrderService
{
    public function __construct(
        private readonly OrderRepository $repository,
    ) {}

    public function find(OrderId $id): ?Order
    {
        return $this->repository->find($id);
    }

    public function create(CreateOrderDto $dto): Order
    {
        // Clean, imported at top
    }
}
```

## Best

```php
<?php

declare(strict_types=1);

namespace App\Http\Controllers;

// Group 1: PHP native
use DateTimeImmutable;
use InvalidArgumentException;

// Group 2: Framework
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;
use Illuminate\Routing\Controller;

// Group 3: Application
use App\Application\Order\OrderService;
use App\Domain\Order\OrderId;
use App\Http\Requests\CreateOrderRequest;
use App\Http\Resources\OrderResource;

final class OrderController extends Controller
{
    public function __construct(
        private readonly OrderService $service,
    ) {}

    public function show(int $id): JsonResponse
    {
        $order = $this->service->find(new OrderId((string) $id));
        return new JsonResponse(new OrderResource($order));
    }
}
```

```php
<?php

declare(strict_types=1);

namespace App\Services;

// Alias to resolve name collision between domain User and HTTP resource User
use App\Domain\User\User as DomainUser;
use App\Http\Resources\User as UserResource;

final class UserSyncService
{
    public function toResource(DomainUser $user): UserResource
    {
        return new UserResource($user);
    }
}
```

## Exceptions / trade-offs
Deep technical namespaces like `App\Infrastructure\Persistence\Doctrine` are fine for infrastructure code — they don't need to carry domain meaning. The rule about domain namespaces applies to entities, value objects, services, and repositories in the domain/application layers. Aliases are a last resort; if you need many aliases, the namespace design probably needs rethinking.

## Namespace layer conventions

```
Domain layer:
  App\Domain\User          — User aggregate (Entity, ValueObject, Event)
  App\Domain\Order         — Order aggregate
  App\Domain\Shared        — Shared value objects across aggregates

Application layer:
  App\Application\Command  — Command objects
  App\Application\Query    — Query objects
  App\Application\Dto      — Data transfer objects
  App\Application\Handler  — Command and query handlers

Infrastructure layer:
  App\Infrastructure\Persistence    — Database implementations
  App\Infrastructure\Http           — HTTP controllers, middleware, resources
  App\Infrastructure\Queue          — Queue workers and jobs
  App\Infrastructure\Cache          — Cache implementations

Tests:
  Tests\Unit\Domain\User
  Tests\Integration\Application
  Tests\Feature\Http
```

## Resolving name conflicts with aliases

```php
<?php

declare(strict_types=1);

namespace App\Application\Sync;

// Alias to resolve collision between domain User and API resource User
use App\Domain\User\User as DomainUser;
use App\Http\Resources\User as UserResource;

// Or use more descriptive aliases
use App\External\Payment\User as PaymentUser;

final class UserSyncService
{
    public function toResource(DomainUser $user): UserResource
    {
        return new UserResource($user);
    }
}
```

## Static-analysis notes
PHPStan and Psalm will error on undefined class names from missing or incorrect imports. PHP-CS-Fixer's `ordered_imports` and `no_unused_imports` rules keep import blocks tidy automatically. Deptrac can enforce that domain namespaces do not import from infrastructure namespaces.

## Related topics
- [psr-file-structure.md](psr-file-structure.md) — imports must appear in the declared position in the file
- [psr-4-autoloading.md](psr-4-autoloading.md) — namespace structure must match directory structure for autoloading
