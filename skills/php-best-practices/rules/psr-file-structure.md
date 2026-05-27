---
title: PSR File Structure
impact: HIGH
impactDescription: PSR standards and code structure conventions
tags: php, psr, coding-standards
---

# PSR File Structure

## Why it matters
A PHP file that mixes namespace declarations after class definitions, or opens with scripting code followed by a class, produces fatal errors or silent class-load failures depending on the autoloader. Beyond correctness, inconsistent file structure forces every reader to re-orient before they can read the code. Predictable structure is free documentation.

## Rule
Every PHP file follows this strict order: `<?php` → `declare(strict_types=1);` → namespace → imports → code. One class per file. File purpose is singular. Never mix procedural scripting and class definitions in the same file.

## Bad

```php
<?php
class UserService {
use LoggableTrait;
private $repo;
const MAX = 100;
public function find($id) {}
private $logger;
public function __construct($repo, $logger) {
$this->repo = $repo; $this->logger = $logger;
}
}
namespace App\Services;        // namespace AFTER the class — fatal
use App\Repositories\UserRepository;
```

## Better

```php
<?php

declare(strict_types=1);

namespace App\Services;

use App\Repositories\UserRepository;
use Psr\Log\LoggerInterface;

class UserService
{
    use LoggableTrait;

    private const MAX_PAGE_SIZE = 100;

    public function __construct(
        private UserRepository $repo,
        private LoggerInterface $logger,
    ) {}

    public function find(int $id): ?User
    {
        return $this->repo->find($id);
    }
}
```

## Best

```php
<?php

declare(strict_types=1);

namespace App\Services;

// PHP native classes first
use DateTimeImmutable;
use InvalidArgumentException;

// External packages
use Psr\EventDispatcher\EventDispatcherInterface;
use Psr\Log\LoggerInterface;

// Internal — ordered alphabetically within the group
use App\Domain\User\User;
use App\Domain\User\UserId;
use App\Domain\User\UserRepository;
use App\Events\UserCreated;

final class UserService
{
    // Constants: public → protected → private
    public const DEFAULT_PAGE_SIZE = 20;
    private const LOG_CHANNEL = 'user';

    // Constructor promotes properties; no separate property declarations needed
    public function __construct(
        private readonly UserRepository $repository,
        private readonly LoggerInterface $logger,
        private readonly EventDispatcherInterface $dispatcher,
    ) {}

    // Public interface first
    public function find(UserId $id): ?User
    {
        return $this->repository->find($id);
    }

    public function create(array $data): User
    {
        $this->guardCreateData($data);
        $user = User::create($data['name'], $data['email']);
        $this->repository->save($user);
        $this->dispatcher->dispatch(new UserCreated($user->getId(), new DateTimeImmutable()));
        $this->logger->info('User created', ['id' => $user->getId()->value]);
        return $user;
    }

    // Private helpers last
    private function guardCreateData(array $data): void
    {
        if (empty($data['email'])) {
            throw new InvalidArgumentException('Email is required');
        }
    }
}
```

## Exceptions / trade-offs
Bootstrap files (e.g., `index.php`, `artisan`) legitimately mix procedural setup with includes — they are entry points, not class files. Migration files and Doctrine fixtures are similar. These are the only acceptable exceptions to "one class per file."

## File structure order (reference)

```
1.  Opening tag:           <?php
2.  Strict types:          declare(strict_types=1);
3.  Blank line
4.  Namespace:             namespace App\Domain\User;
5.  Blank line
6.  Use statements:
    a. PHP native classes  (DateTimeImmutable, InvalidArgumentException…)
    b. External packages   (Psr\Log\LoggerInterface, Doctrine\…)
    c. Internal project    (App\Domain\…, App\Application\…)
    — alphabetised within each group, blank line between groups
7.  Blank line
8.  Class / interface / trait / enum declaration
    a. Traits              (use LoggableTrait;)
    b. Constants           (public → protected → private)
    c. Properties          (public → protected → private)
    d. Constructor
    e. Public methods
    f. Protected methods
    g. Private methods
```

## Use statement grouping

```php
<?php

declare(strict_types=1);

namespace App\Services;

// Group 1: PHP native
use DateTimeImmutable;
use InvalidArgumentException;
use RuntimeException;

// Group 2: External packages (alphabetised by vendor)
use Psr\Log\LoggerInterface;
use Stripe\StripeClient;

// Group 3: Internal project (alphabetised by namespace)
use App\Domain\User\User;
use App\Domain\User\UserRepository;
use App\Events\UserCreated;
use App\Exceptions\UserNotFoundException;
```

## Static-analysis notes
PHP-CS-Fixer with `ordered_imports`, `declare_strict_types`, `single_blank_line_before_namespace`, and `no_unused_imports` rules will enforce and autofix the ordering. PHPStan will catch undeclared namespaces and missing imports at level 0+.

## Related topics
- [psr-4-autoloading.md](psr-4-autoloading.md) — file structure must match namespace-to-path mapping
- [psr-namespace-usage.md](psr-namespace-usage.md) — how to structure and alias namespace imports
- [psr-12-coding-style.md](psr-12-coding-style.md) — formatting rules inside the file body
