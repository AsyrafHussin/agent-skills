---
title: Long Parameter Lists
impact: MEDIUM
impactDescription: "Hard to call correctly; signals mixed responsibilities"
tags: parameters, srp, refactoring
---

## Long Parameter Lists

**Impact: MEDIUM (Hard to call correctly; signals mixed responsibilities)**

A function with 7+ parameters is almost impossible to call correctly without re-reading the signature each time. Positional arguments get swapped (`createUser(name, email, ...)` vs `createUser(email, name, ...)`), and the function is usually doing too many things.

## How to Detect

```bash
# JavaScript / TypeScript
npx eslint . --rule 'max-params: ["error", 4]'

# PHP
vendor/bin/phpmd app text codesize  # ExcessiveParameterList (default 10; lower via ruleset)
```

Threshold: **> 4 parameters** (3 or fewer is fine, 4 is borderline, 5+ is debt).

## Incorrect

```typescript
// ❌ Eight positional parameters — easy to swap, hard to call
export function createBooking(
  userId: string,
  hotelId: string,
  roomTypeId: string,
  startDate: Date,
  endDate: Date,
  guestCount: number,
  promoCode: string | null,
  notes: string,
): Booking {
  // ...
}

// At the call site:
createBooking(u, h, r, s, e, 2, null, 'Late check-in');   // is 2 the count or roomTypeId?
```

```php
// ❌ Same anti-pattern in PHP
public function process(
    int $orderId,
    int $userId,
    int $shippingMethodId,
    string $address,
    string $city,
    string $postalCode,
    string $country,
    bool $expedited,
    bool $giftWrap,
): Order { /* ... */ }
```

**Problems:**
- Reordering parameters in a refactor silently breaks every caller (same type → no compile error)
- Optional parameters force you to pass `null` for arguments you don't care about
- The signature is doing the work of a value object

## Correct

```typescript
// ✅ Introduce Parameter Object — named, optional fields make intent explicit
interface BookingRequest {
  userId: string;
  hotelId: string;
  roomTypeId: string;
  stay: { from: Date; to: Date };
  guestCount: number;
  promoCode?: string;
  notes?: string;
}

export function createBooking(req: BookingRequest): Booking { /* ... */ }

// Call site:
createBooking({
  userId: u,
  hotelId: h,
  roomTypeId: r,
  stay: { from: s, to: e },
  guestCount: 2,
  notes: 'Late check-in',
});
```

```php
// ✅ PHP equivalent: a DTO / value object
final readonly class BookingRequest
{
    public function __construct(
        public int $userId,
        public int $hotelId,
        public int $roomTypeId,
        public DateRange $stay,
        public int $guestCount,
        public ?string $promoCode = null,
        public ?string $notes = null,
    ) {}
}

public function process(BookingRequest $req): Order { /* ... */ }
```

**Benefits:**
- Named arguments are self-documenting
- Adding a field doesn't break callers
- The object becomes a natural place to add validation or behaviour later

## Remediation Strategy

- **Effort:** S–M (Introduce Parameter Object is a well-known refactor; most IDEs automate it)
- **When to pay down:** When you need to add yet another parameter to an already-long signature, or when a bug is traced to swapped arguments at a call site.

**Anti-pattern:** "Boolean flag parameters" — `process(order, true, false, true)` is unreadable. Replace with enum values or split into separate functions.

Reference: [Refactoring — Introduce Parameter Object](https://refactoring.guru/introduce-parameter-object) · [Refactoring — Replace Parameter with Method Call](https://refactoring.guru/replace-parameter-with-method-call)
