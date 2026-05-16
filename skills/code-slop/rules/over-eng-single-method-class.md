---
title: Single-Method Class That Should Be a Function
impact: HIGH
impactDescription: "Wraps a free function in class ceremony for no reason"
tags: over-engineering, classes, ai-fingerprint
---

## Single-Method Class That Should Be a Function

**Impact: HIGH (Wraps a free function in class ceremony for no reason)**

A class with exactly one public method, no state, and no dependencies should usually be a function. AI generates these because its training data is enterprise Java/C# where classes are mandatory for everything. PHP and TypeScript both support free functions / static methods / single-purpose action classes — the wrapping ceremony is pure slop.

The exception: **invokable action classes** (Laravel convention) using `__invoke()` are legitimate when they have constructor-injected dependencies and represent a named domain action. The slop variant is the class with no constructor, no state, one static method, and no clear domain identity.

## Incorrect

```php
// ❌ Class wrapping a single pure function

class StringFormatter
{
    public static function slugify(string $input): string
    {
        return Str::slug($input);
    }
}

class EmailValidator
{
    public function isValid(string $email): bool
    {
        return filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
    }
}

class TimestampHelper
{
    public static function toIso(DateTime $dt): string
    {
        return $dt->format(DateTimeInterface::ATOM);
    }
}
```

```typescript
// ❌ Same in TS
class StringUtil {
  static capitalize(s: string): string {
    return s.charAt(0).toUpperCase() + s.slice(1);
  }
}

class ResponseFormatter {
  static format(data: unknown): ApiResponse {
    return { data, timestamp: Date.now() };
  }
}
```

**Why it's slop:**
- Callers write `StringFormatter::slugify(...)` instead of `slugify(...)`
- `new EmailValidator()->isValid($email)` is six tokens for what should be one
- The class adds no encapsulation (no state to encapsulate)
- Importing/autoloading the class wastes bytes for zero benefit
- Pattern is recognisably "Java port" — PHP and TS have first-class functions

## Correct

```php
// ✅ Free function or method on the existing domain type

// helpers.php (or composer autoload "files")
function slugify(string $input): string
{
    return Str::slug($input);
}

function isValidEmail(string $email): bool
{
    return filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
}

// Or extension methods on Carbon
class CustomCarbon extends Carbon
{
    public function toIso(): string { return $this->format(DateTimeInterface::ATOM); }
}
```

```typescript
// ✅ Free exported functions
export function capitalize(s: string): string {
  return s.charAt(0).toUpperCase() + s.slice(1);
}

export function formatResponse<T>(data: T): ApiResponse<T> {
  return { data, timestamp: Date.now() };
}
```

**Why it reads human:**
- Caller writes `slugify(input)` — one token, no ceremony
- No autoloader hit, no class instantiation, no dependency to mock in tests
- Functions are testable directly with no setup

## Single-method class is OK when…

These are legitimate, NOT slop:

```php
// ✅ Invokable action with injected dependencies — Laravel idiom
final class PlaceOrder
{
    public function __construct(
        private PaymentGateway $payments,
        private InventoryService $inventory,
    ) {}

    public function __invoke(OrderRequest $request): Order { /* ... */ }
}

// ✅ Job / command — meant to be queued
final class SendWeeklyDigest implements ShouldQueue
{
    public function handle(MailerService $mailer): void { /* ... */ }
}

// ✅ Form Request — Laravel pattern
final class StoreUserRequest extends FormRequest { /* ... */ }
```

The test: **does it have state, dependencies, or a domain identity beyond "I wrap a function"?** If yes, it's a class. If no, it should be a function.

## Detection

```bash
# PHP — classes with exactly one public method and no constructor injection
# (rough heuristic: file has 'class X' + exactly one 'public function')
for f in $(find app/ -name '*.php'); do
  PUBLIC=$(grep -cE '^\s+public function ' "$f")
  CTOR=$(grep -cE '^\s+public function __construct' "$f")
  PROPS=$(grep -cE '^\s+(private|protected) (readonly )?[a-zA-Z]+ \$' "$f")
  if [ "$PUBLIC" = "1" ] && [ "$CTOR" = "0" ] && [ "$PROPS" = "0" ]; then
    echo "SUSPECT (single-method, no state): $f"
  fi
done

# TS — classes with one method and no constructor or fields
grep -rln 'class\s\+[A-Z]' --include='*.ts' src/ | while read f; do
  METHODS=$(grep -cE '^\s+(public\s+|private\s+|protected\s+)?[a-zA-Z]+\s*\(' "$f")
  CTOR=$(grep -c 'constructor' "$f")
  if [ "$METHODS" = "1" ] && [ "$CTOR" = "0" ]; then
    echo "SUSPECT: $f"
  fi
done
```

Reference: Internal: [`naming-suffix-abuse`](naming-suffix-abuse.md), [`over-eng-useless-wrapper`](over-eng-useless-wrapper.md)
