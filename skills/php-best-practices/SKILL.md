---
name: php-best-practices
description: PHP 8.x modern patterns, PSR standards, and SOLID principles. Use when reviewing PHP code, checking type safety, auditing code quality, or ensuring PHP best practices. Triggers on "review PHP", "check PHP code", "audit PHP", or "PHP best practices".
license: MIT
metadata:
  author: agent-skills
  version: "1.0.0"
---

# PHP Best Practices

Modern PHP 8.x patterns, PSR standards, type system best practices, and SOLID principles. Contains 45+ rules for writing clean, maintainable PHP code.

## When to Apply

Reference these guidelines when:
- Writing or reviewing PHP code
- Implementing classes and interfaces
- Using PHP 8.x modern features
- Ensuring type safety
- Following PSR standards
- Applying design patterns

## Rule Categories by Priority

| Priority | Category | Impact | Prefix |
|----------|----------|--------|--------|
| 1 | Type System | CRITICAL | `type-` |
| 2 | Modern Features | CRITICAL | `modern-` |
| 3 | PSR Standards | HIGH | `psr-` |
| 4 | SOLID Principles | HIGH | `solid-` |
| 5 | Error Handling | HIGH | `error-` |
| 6 | Performance | MEDIUM | `perf-` |
| 7 | Security | CRITICAL | `sec-` |

## Quick Reference

### 1. Type System (CRITICAL)

- `type-strict-mode` - Declare strict types
- `type-return-types` - Always declare return types
- `type-parameter-types` - Type all parameters
- `type-property-types` - Type class properties
- `type-union-types` - Use union types effectively
- `type-intersection-types` - Use intersection types
- `type-nullable` - Handle nullable types properly
- `type-mixed-avoid` - Avoid mixed type when possible
- `type-void-return` - Use void for no-return methods
- `type-never-return` - Use never for non-returning functions

### 2. Modern Features (CRITICAL)

- `modern-constructor-promotion` - Use constructor property promotion
- `modern-readonly-properties` - Use readonly for immutable data
- `modern-readonly-classes` - Use readonly classes
- `modern-enums` - Use enums instead of constants
- `modern-attributes` - Use attributes for metadata
- `modern-match-expression` - Use match over switch
- `modern-named-arguments` - Use named arguments for clarity
- `modern-nullsafe-operator` - Use nullsafe operator (?->)
- `modern-arrow-functions` - Use arrow functions for simple closures
- `modern-first-class-callables` - Use first-class callable syntax

### 3. PSR Standards (HIGH)

- `psr-4-autoloading` - Follow PSR-4 autoloading
- `psr-12-coding-style` - Follow PSR-12 coding style
- `psr-naming-conventions` - Class and method naming
- `psr-file-structure` - One class per file
- `psr-namespace-declaration` - Proper namespace usage

### 4. SOLID Principles (HIGH)

- `solid-single-responsibility` - One reason to change
- `solid-open-closed` - Open for extension, closed for modification
- `solid-liskov-substitution` - Subtypes must be substitutable
- `solid-interface-segregation` - Small, focused interfaces
- `solid-dependency-inversion` - Depend on abstractions

### 5. Error Handling (HIGH)

- `error-custom-exceptions` - Create specific exceptions
- `error-exception-hierarchy` - Proper exception inheritance
- `error-try-catch-specific` - Catch specific exceptions
- `error-finally-cleanup` - Use finally for cleanup
- `error-never-suppress` - Don't suppress errors with @

### 6. Performance (MEDIUM)

- `perf-avoid-globals` - Avoid global variables
- `perf-lazy-loading` - Load resources lazily
- `perf-array-functions` - Use native array functions
- `perf-string-functions` - Use native string functions
- `perf-generators` - Use generators for large datasets

### 7. Security (CRITICAL)

- `sec-input-validation` - Validate all input
- `sec-output-escaping` - Escape output properly
- `sec-password-hashing` - Use password_hash/verify
- `sec-sql-prepared` - Use prepared statements
- `sec-file-uploads` - Validate file uploads

## Essential Guidelines

### Strict Types Declaration

```php
<?php

declare(strict_types=1);

// ❌ Without strict types - silent type coercion
function add($a, $b) {
    return $a + $b;
}
add("5", "3"); // Works but dangerous

// ✅ With strict types - type errors caught
function add(int $a, int $b): int {
    return $a + $b;
}
add("5", "3"); // TypeError thrown
```

### Constructor Property Promotion

```php
<?php

// ❌ Old verbose way
class User
{
    private string $name;
    private string $email;
    private bool $active;

    public function __construct(
        string $name,
        string $email,
        bool $active = true
    ) {
        $this->name = $name;
        $this->email = $email;
        $this->active = $active;
    }
}

// ✅ PHP 8.x constructor promotion
class User
{
    public function __construct(
        private string $name,
        private string $email,
        private bool $active = true,
    ) {}
}
```

### Readonly Properties and Classes

```php
<?php

// ✅ Readonly properties - immutable after initialization
class User
{
    public function __construct(
        public readonly string $id,
        public readonly string $email,
        private readonly DateTimeImmutable $createdAt,
    ) {}
}

// ✅ Readonly class - all properties are readonly
readonly class ValueObject
{
    public function __construct(
        public string $value,
        public string $type,
    ) {}
}
```

### Enums

```php
<?php

// ❌ Old way with constants
class Status
{
    public const PENDING = 'pending';
    public const ACTIVE = 'active';
    public const INACTIVE = 'inactive';
}

// ✅ PHP 8.1+ enums
enum Status: string
{
    case Pending = 'pending';
    case Active = 'active';
    case Inactive = 'inactive';

    public function label(): string
    {
        return match($this) {
            self::Pending => 'Awaiting Review',
            self::Active => 'Active',
            self::Inactive => 'Deactivated',
        };
    }

    public function color(): string
    {
        return match($this) {
            self::Pending => 'yellow',
            self::Active => 'green',
            self::Inactive => 'gray',
        };
    }
}

// Usage
$status = Status::Active;
$status->value;  // 'active'
$status->label(); // 'Active'
Status::from('pending'); // Status::Pending
Status::tryFrom('invalid'); // null
```

### Match Expression

```php
<?php

// ❌ Verbose switch statement
switch ($status) {
    case 'pending':
        $result = 'Waiting';
        break;
    case 'active':
        $result = 'Running';
        break;
    case 'inactive':
        $result = 'Stopped';
        break;
    default:
        $result = 'Unknown';
}

// ✅ Match expression - concise and returns value
$result = match($status) {
    'pending' => 'Waiting',
    'active' => 'Running',
    'inactive' => 'Stopped',
    default => 'Unknown',
};

// ✅ Match with multiple conditions
$category = match(true) {
    $age < 13 => 'child',
    $age < 20 => 'teenager',
    $age < 65 => 'adult',
    default => 'senior',
};
```

### Named Arguments

```php
<?php

// ❌ Positional arguments - unclear meaning
$user = new User('John', 'john@example.com', true, false, null);

// ✅ Named arguments - self-documenting
$user = new User(
    name: 'John',
    email: 'john@example.com',
    isActive: true,
    isAdmin: false,
    department: null,
);

// ✅ Skip optional parameters
function createNotification(
    string $message,
    string $type = 'info',
    bool $persistent = false,
    ?int $timeout = null,
): Notification {
    // ...
}

createNotification(
    message: 'Hello',
    persistent: true,  // Skip $type, use default
);
```

### Nullsafe Operator

```php
<?php

// ❌ Verbose null checks
$country = null;
if ($user !== null) {
    $address = $user->getAddress();
    if ($address !== null) {
        $country = $address->getCountry();
    }
}

// ✅ Nullsafe operator
$country = $user?->getAddress()?->getCountry();

// ✅ Combined with null coalescing
$countryCode = $user?->getAddress()?->getCountry()?->code ?? 'US';
```

### Union and Intersection Types

```php
<?php

// ✅ Union types - accepts multiple types
function process(string|int|null $value): string|false
{
    if ($value === null) {
        return false;
    }
    return (string) $value;
}

// ✅ Intersection types - must implement all
function processEntity(Countable&Traversable $collection): int
{
    return count($collection);
}

// ✅ DNF (Disjunctive Normal Form) types - PHP 8.2+
function handle((Countable&Traversable)|null $collection): void
{
    // ...
}
```

### Attributes

```php
<?php

use Attribute;

// Define custom attribute
#[Attribute(Attribute::TARGET_METHOD | Attribute::TARGET_PROPERTY)]
class Validate
{
    public function __construct(
        public string $rule,
        public ?string $message = null,
    ) {}
}

// Use attribute
class UserRequest
{
    #[Validate('required|email', 'Please provide a valid email')]
    public string $email;

    #[Validate('required|min:8')]
    public string $password;
}

// Read attributes via reflection
$reflection = new ReflectionClass(UserRequest::class);
foreach ($reflection->getProperties() as $property) {
    $attributes = $property->getAttributes(Validate::class);
    foreach ($attributes as $attribute) {
        $validator = $attribute->newInstance();
        // $validator->rule, $validator->message
    }
}
```

### First-Class Callable Syntax

```php
<?php

class Calculator
{
    public function add(int $a, int $b): int
    {
        return $a + $b;
    }

    public static function multiply(int $a, int $b): int
    {
        return $a * $b;
    }
}

$calc = new Calculator();

// ❌ Old way
$addFn = Closure::fromCallable([$calc, 'add']);
$multiplyFn = Closure::fromCallable([Calculator::class, 'multiply']);

// ✅ First-class callable syntax (PHP 8.1+)
$addFn = $calc->add(...);
$multiplyFn = Calculator::multiply(...);

// Use as callback
$numbers = [[1, 2], [3, 4], [5, 6]];
$results = array_map($calc->add(...), ...$numbers);
```

### Arrow Functions

```php
<?php

// ❌ Verbose closure
$multiplier = 2;
$multiply = function (int $n) use ($multiplier): int {
    return $n * $multiplier;
};

// ✅ Arrow function - auto-captures variables
$multiplier = 2;
$multiply = fn(int $n): int => $n * $multiplier;

// ✅ Array operations
$users = [/* ... */];
$names = array_map(fn(User $u) => $u->name, $users);
$active = array_filter($users, fn(User $u) => $u->isActive);
```

### Custom Exceptions

```php
<?php

// ✅ Domain-specific exception hierarchy
abstract class DomainException extends Exception {}

class EntityNotFoundException extends DomainException
{
    public function __construct(
        public readonly string $entityType,
        public readonly string|int $id,
    ) {
        parent::__construct(
            sprintf('%s with ID "%s" not found', $entityType, $id)
        );
    }
}

class ValidationException extends DomainException
{
    public function __construct(
        public readonly array $errors,
    ) {
        parent::__construct('Validation failed');
    }
}

// Usage
throw new EntityNotFoundException('User', $userId);
throw new ValidationException(['email' => 'Invalid email format']);
```

### Interface Segregation

```php
<?php

// ❌ Fat interface
interface UserRepositoryInterface
{
    public function find(int $id): ?User;
    public function findAll(): array;
    public function save(User $user): void;
    public function delete(User $user): void;
    public function findByEmail(string $email): ?User;
    public function findByRole(string $role): array;
    public function export(): string;
    public function import(string $data): void;
}

// ✅ Segregated interfaces
interface ReadableRepository
{
    public function find(int $id): ?User;
    public function findAll(): array;
}

interface WritableRepository
{
    public function save(User $user): void;
    public function delete(User $user): void;
}

interface UserQueryInterface
{
    public function findByEmail(string $email): ?User;
    public function findByRole(string $role): array;
}

interface Exportable
{
    public function export(): string;
    public function import(string $data): void;
}

// Implement only what's needed
class UserRepository implements ReadableRepository, WritableRepository, UserQueryInterface
{
    // ...
}
```

### Dependency Injection

```php
<?php

// ❌ Hard-coded dependencies
class OrderService
{
    public function process(Order $order): void
    {
        $mailer = new SmtpMailer(); // Hard-coded
        $logger = Logger::getInstance(); // Singleton
        // ...
    }
}

// ✅ Constructor injection with interfaces
class OrderService
{
    public function __construct(
        private readonly MailerInterface $mailer,
        private readonly LoggerInterface $logger,
        private readonly OrderRepositoryInterface $repository,
    ) {}

    public function process(Order $order): void
    {
        $this->repository->save($order);
        $this->mailer->send($order->customer->email, 'Order confirmed');
        $this->logger->info('Order processed', ['id' => $order->id]);
    }
}
```

### Generators for Large Datasets

```php
<?php

// ❌ Memory-heavy - loads all records
function getAllUsers(): array
{
    $users = [];
    $result = $db->query('SELECT * FROM users');
    while ($row = $result->fetch()) {
        $users[] = new User($row);
    }
    return $users; // Could be millions of records
}

// ✅ Generator - memory efficient
function getAllUsers(): Generator
{
    $result = $db->query('SELECT * FROM users');
    while ($row = $result->fetch()) {
        yield new User($row);
    }
}

// Usage - processes one at a time
foreach (getAllUsers() as $user) {
    processUser($user);
}
```

## Output Format

When auditing code, output findings in this format:

```
file:line - [category] Description of issue
```

Example:
```
src/Services/UserService.php:15 - [type] Missing return type declaration
src/Models/Order.php:42 - [modern] Use match expression instead of switch
src/Controllers/ApiController.php:28 - [solid] Class has multiple responsibilities
```

## How to Use

Read individual rule files for detailed explanations:

```
rules/modern-constructor-promotion.md
rules/type-strict-mode.md
rules/solid-single-responsibility.md
```
