---
title: PSR-12 Coding Style
impact: HIGH
impactDescription: PSR standards and code structure conventions
tags: php, psr, coding-standards
---

# PSR-12 Coding Style

## Why it matters
Inconsistent formatting is a tax on every code review. Debating brace placement, spacing around operators, and import ordering consumes reviewer attention that should go to correctness and design. A machine can enforce formatting; humans should not waste time on it. Code formatted inconsistently across a codebase also makes `git diff` harder to read — a pure formatting change pollutes a meaningful change.

## Rule
One automated style, zero bike-shedding. Use `php-cs-fixer` or PHP_CodeSniffer with the PSR-12 preset. Run it in CI as a non-negotiable check. Nobody should ever debate brace placement in a code review again.

## Bad

```php
<?php
namespace App\Services;
use App\Models\User;use App\Repositories\UserRepository;

class UserService{
    private $repository;
    public function __construct(UserRepository $repo){
        $this->repository=$repo;
    }
    public function find($id){
        if($id<1){return null;}
        return $this->repository->find($id);
    }
    public function create($data){
        if(!isset($data['email'])||!isset($data['name'])){throw new \Exception('Missing data');}
        return $this->repository->create($data);
    }
}
```

## Better

```php
<?php

declare(strict_types=1);

namespace App\Services;

use App\Models\User;
use App\Repositories\UserRepository;
use InvalidArgumentException;

class UserService
{
    public function __construct(
        private UserRepository $repository,
    ) {}

    public function find(int $id): ?User
    {
        if ($id < 1) {
            return null;
        }

        return $this->repository->find($id);
    }

    public function create(array $data): User
    {
        if (!isset($data['email']) || !isset($data['name'])) {
            throw new InvalidArgumentException('Missing required data');
        }

        return $this->repository->create($data);
    }
}
```

## Best

```php
<?php

declare(strict_types=1);

namespace App\Services;

use App\Contracts\OrderServiceInterface;
use App\Domain\Order\Order;
use App\Domain\Order\OrderId;
use App\Domain\Order\OrderRepository;
use App\Exceptions\OrderNotFoundException;
use Psr\Log\LoggerInterface;

final class OrderService implements OrderServiceInterface
{
    public function __construct(
        private readonly OrderRepository $repository,
        private readonly LoggerInterface $logger,
    ) {}

    public function findOrFail(OrderId $id): Order
    {
        $order = $this->repository->find($id);

        if ($order === null) {
            throw new OrderNotFoundException($id);
        }

        return $order;
    }

    public function ship(
        OrderId $id,
        string $trackingNumber,
        bool $notifyCustomer = true,
    ): Order {
        $order = $this->findOrFail($id);
        $order->markAsShipped($trackingNumber);
        $this->repository->save($order);

        $this->logger->info('Order shipped', [
            'order_id' => $id->value,
            'tracking' => $trackingNumber,
        ]);

        return $order;
    }

    public function cancel(OrderId $id, string $reason): Order
    {
        $order = $this->findOrFail($id);

        match ($order->status()) {
            OrderStatus::Pending  => $order->cancel($reason),
            OrderStatus::Shipped  => throw new \DomainException('Cannot cancel a shipped order'),
            OrderStatus::Canceled => throw new \DomainException('Order is already canceled'),
        };

        $this->repository->save($order);

        return $order;
    }
}
```

## Exceptions / trade-offs
Legacy codebases with thousands of files may need a phased rollout: configure php-cs-fixer to run only on modified files first, then batch-format the rest in a dedicated commit with no logic changes. Template/stub files used by generators may intentionally deviate from formatting to produce correct output — exclude them from the linter.

## Static-analysis notes
`php-cs-fixer` with `@PSR12` rule set is the canonical enforcement tool. Add `--dry-run --diff` in CI to fail on violations without auto-fixing. For IDE integration, `.php-cs-fixer.dist.php` in the repo root enables on-save formatting in PhpStorm and VS Code. PHP_CodeSniffer with `phpcs --standard=PSR12` is the alternative if your project uses it.

## Version notes
PSR-12 was finalized in 2019 and supersedes PSR-2. It is the current standard for all new PHP projects. PHP 8.x additions (named arguments, match expressions, readonly properties, enums) are not yet covered by PSR-12 formally — follow php-cs-fixer's `@PHP81Migration` or `@PHP82Migration` rule sets for those.

## Related topics
- [psr-file-structure.md](psr-file-structure.md) — file-level ordering that PSR-12 depends on
- [psr-namespace-usage.md](psr-namespace-usage.md) — import ordering enforced by php-cs-fixer
