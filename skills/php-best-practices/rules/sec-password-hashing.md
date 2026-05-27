---
title: Hash Passwords with `password_hash()` and Verify with `password_verify()`
impact: CRITICAL
impactDescription: Security best practices and input safety
tags: php, security, validation
---

# Hash Passwords with `password_hash()` and Verify with `password_verify()`

## Why it matters
Passwords stored with MD5 or SHA1 can be reversed for the entire user base within hours using precomputed tables and consumer GPU hardware. Every breach that exposes an MD5 password database results in mass credential compromise, cross-site account takeover, and regulatory liability. The PHP password API was designed to make the correct choice the easy choice.

## Rule
Use `password_hash()` with `PASSWORD_ARGON2ID` (or `PASSWORD_DEFAULT`) to hash, and `password_verify()` to compare. Never use MD5, SHA1, or bare SHA-256. Rehash on successful login when `password_needs_rehash()` returns true.

## Bad
```php
<?php

declare(strict_types=1);

// Plain text — worst possible
$db->insert('users', ['password' => $_POST['password']]);

// MD5 — trivially reversed via rainbow tables
$hash = md5($_POST['password']);

// SHA1 — equally weak
$hash = sha1($_POST['password']);

// SHA-256 without a per-user salt — still vulnerable to precomputation
$hash = hash('sha256', $_POST['password']);

// Timing-unsafe comparison
if ($stored === md5($input)) { /* login */ }
```

## Better
```php
<?php

declare(strict_types=1);

// bcrypt is better than SHA1, but PASSWORD_DEFAULT is preferred
// over hardcoding PASSWORD_BCRYPT so the algorithm can evolve
$hash = password_hash($_POST['password'], PASSWORD_BCRYPT);

if (password_verify($_POST['password'], $stored)) {
    // logged in — but no rehash check
}
```

## Best
```php
<?php

declare(strict_types=1);

// Hash on registration
function hashPassword(string $plaintext): string
{
    if (mb_strlen($plaintext) < 8) {
        throw new ValidationException(['password' => 'Minimum 8 characters required']);
    }
    // bcrypt truncates at 72 bytes; validate to avoid silent truncation
    if (strlen($plaintext) > 72) {
        throw new ValidationException(['password' => 'Password must not exceed 72 characters']);
    }

    return password_hash($plaintext, PASSWORD_ARGON2ID, [
        'memory_cost' => 65536, // 64 MB
        'time_cost'   => 4,
        'threads'     => 2,
    ]);
}

// Verify and rehash on login
function verifyAndRehash(
    string $plaintext,
    string $stored,
    callable $persistHash,
): bool {
    if (!password_verify($plaintext, $stored)) {
        return false;
    }

    if (password_needs_rehash($stored, PASSWORD_ARGON2ID)) {
        $persistHash(password_hash($plaintext, PASSWORD_ARGON2ID));
    }

    return true;
}

// Session fixation protection after successful login
session_regenerate_id(true);
```

## Exceptions / trade-offs
`PASSWORD_DEFAULT` is a valid alternative to hardcoding `PASSWORD_ARGON2ID` — it picks the best available algorithm and will change in future PHP versions, triggering automatic rehashing via `password_needs_rehash()`. If the deployment environment lacks the `sodium` or `argon2` extensions, `PASSWORD_BCRYPT` is the fallback.

## Static-analysis notes
PHPStan and Psalm cannot detect weak hashing algorithms automatically. Enforce this via a custom PHPStan rule or Rector rule that flags calls to `md5`, `sha1`, and `hash` with weak algorithm strings in security-sensitive contexts.

## Security notes
Argon2id is memory-hard, making GPU/ASIC attacks significantly more expensive than against bcrypt. Tune `memory_cost` upward as hardware improves. `password_verify` is timing-safe by design; never compare hashes with `==` or `===`.

## Version notes
`PASSWORD_ARGON2ID` — PHP 7.3+

## Related topics
- [sec-input-validation.md](sec-input-validation.md) — validate password length before hashing
- [error-never-suppress.md](error-never-suppress.md) — do not suppress errors from `password_hash()`
