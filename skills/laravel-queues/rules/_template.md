---
title: Rule Title Here
impact: CRITICAL|HIGH|MEDIUM
impactDescription: "Specific consequence — e.g., 'Jobs fail silently on retry; payment duplicates ship to customers'"
tags: tag1, tag2, tag3
---

## Rule Title Here

**Impact: LEVEL (impactDescription)**

1-2 sentences explaining why this rule matters in production.

## Incorrect

```php
// ❌ Bad pattern
```

**Why it breaks:**
- Reason 1
- Reason 2

## Correct

```php
// ✅ Good pattern
```

**Why it works:**
- Reason 1
- Reason 2

## Detection / Enforcement

```bash
# grep / phpstan / heuristic
```

Reference: [Laravel 13 — Section](https://laravel.com/docs/13.x/queues)
