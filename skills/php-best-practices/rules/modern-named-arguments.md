---
title: Named Arguments
impact: CRITICAL
impactDescription: Modern PHP 8.x language features
tags: php, modern-php, php8
---

# Named Arguments

## Why it matters
Positional boolean and integer flags at call sites are invisible — `setCookie('id', $val, 0, '', '', true, true)` communicates nothing without reading the signature. Named arguments embed that context directly in the call, eliminating a constant source of review mistakes and misread documentation.

## Rule
Use named arguments when the call site becomes materially clearer, and only for functions whose parameter names are under your control (not third-party APIs).

## Bad
```php
<?php

declare(strict_types=1);

// What does true, true mean here?
setcookie('token', $value, 0, '', '', true, true);

// Which argument is the needle and which is the haystack?
$pos = strpos($haystack, $needle, 0);
```

## Better
```php
<?php

declare(strict_types=1);

// Named args document the intent
setcookie(
    name: 'token',
    value: $value,
    secure: true,
    httponly: true,
);
```

## Best
```php
<?php

declare(strict_types=1);

final class Mailer
{
    public function send(
        string $to,
        string $subject,
        string $body,
        bool $html = false,
        int $priority = 3,
    ): void {}
}

$mailer = new Mailer();

// Skip defaults explicitly; self-documenting
$mailer->send(
    to: 'user@example.com',
    subject: 'Welcome',
    body: $html,
    html: true,
    // priority intentionally omitted — default is fine
);
```

## Exceptions / trade-offs
Do **not** use named arguments when calling third-party library functions or any public API where parameter names are not part of the versioned contract. A library rename of `$haystack` to `$string` (as PHP itself did in 8.0 for several functions) silently breaks named-argument call sites. Restrict named arguments to code you own and control.

## Static-analysis notes
PHPStan and Psalm validate named argument names against the function or method signature and report typos or mismatches as errors. This means named-argument call sites are safer in code you own than positional ones.

## Version notes
`PHP 8.0+`

## Related topics
- [modern-constructor-promotion.md](modern-constructor-promotion.md) — promoted parameters work naturally with named arguments
- [modern-attributes.md](modern-attributes.md) — attribute constructors benefit from named arguments
