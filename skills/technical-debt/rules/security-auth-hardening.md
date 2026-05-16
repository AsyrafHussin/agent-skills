---
title: Auth and Hardening Gaps
impact: HIGH
impactDescription: "Outdated auth defaults are tomorrow's breach disclosures"
tags: security, authentication, authorization, hardening
---

## Auth and Hardening Gaps

**Impact: HIGH (Outdated auth defaults are tomorrow's breach disclosures)**

Auth choices made years ago — password hashing algorithm, session lifetime, missing MFA, missing rate limits, missing security headers — accrue as silent debt. They only become visible during pen tests or incidents, where they're suddenly the biggest finding.

## How to Detect

Audit each of:
1. **Password hashing** — bcrypt is OK, argon2id is preferred; MD5/SHA-1 are unacceptable
2. **Session lifetime** — infinite or multi-month sessions are debt
3. **MFA support** — is it offered? Is it enforced for admins?
4. **Rate limiting** — login, password-reset, API endpoints
5. **Security headers** — `Content-Security-Policy`, `Strict-Transport-Security`, `X-Content-Type-Options`, `Referrer-Policy`
6. **Authorization checks** — every endpoint enforces who can call it
7. **CSRF protection** — present on every state-changing endpoint that uses cookies

```bash
# Check security headers
curl -sI https://your.app | grep -iE 'content-security|strict-transport|x-content-type|x-frame|referrer'

# Mozilla Observatory CLI
# https://observatory.mozilla.org/
# securityheaders.com

# Laravel: scan controllers for missing authorization
grep -rEn 'function (index|show|store|update|destroy)\(' app/Http/Controllers/ | \
  while IFS=: read -r FILE LINE REST; do \
    grep -q 'authorize\|Gate::\|middleware' "$FILE" || echo "MISSING AUTHZ: $FILE:$LINE"; \
  done
```

## Incorrect

```php
// ❌ Multiple hardening gaps
// User registration with MD5 (catastrophic)
public function register(Request $request) {
    User::create([
        'email' => $request->email,
        'password' => md5($request->password),     // hash algorithm from 1992
    ]);
}

// Session config (config/session.php)
'lifetime' => 525600,     // 1 year sessions — every stolen device is forever

// No rate limit on login → credential stuffing trivial
Route::post('/login', [AuthController::class, 'login']);

// No CSP — XSS gets full DOM access
// (no header set in middleware)

// Authorization missing — anyone with a session can hit admin endpoints
Route::get('/admin/users', [AdminController::class, 'users']);
```

## Correct

```php
// ✅ Password hashing via Hash::make (bcrypt by default in Laravel 11+;
//    argon2id is opt-in via config/hashing.php — preferred for new projects)
User::create([
    'email' => $request->validated('email'),
    'password' => Hash::make($request->validated('password')),
]);

// Reasonable session lifetime, secure flags
'lifetime' => 60 * 8,           // 8h
'secure' => true,               // HTTPS only
'http_only' => true,
'same_site' => 'lax',

// Login rate limited
Route::post('/login', [AuthController::class, 'login'])
    ->middleware('throttle:5,1');   // 5 attempts per minute per IP

// Authorization on every admin endpoint
Route::middleware(['auth', 'can:viewAdmin'])->group(function () {
    Route::get('/admin/users', [AdminController::class, 'users']);
});

// Security headers via middleware (or a package like spatie/laravel-csp)
return $next($request)
    ->header('Content-Security-Policy', "default-src 'self'; ...")
    ->header('Strict-Transport-Security', 'max-age=63072000; includeSubDomains; preload')
    ->header('X-Content-Type-Options', 'nosniff')
    ->header('Referrer-Policy', 'strict-origin-when-cross-origin');
```

**Benefits:**
- Hashing upgrade path supported by `Hash::needsRehash()` — old MD5 hashes can be transparently re-hashed on next successful login (after one-time migration to bcrypt/argon2id)
- Session theft window is bounded
- Authorization is explicit and uniformly applied via middleware

## Remediation Strategy

- **Effort:**
  - **S** — security headers, rate limits, session lifetime
  - **M** — adding MFA, enforcing authz across all endpoints
  - **L** — password hash migration (must be done on next login per user; takes weeks of natural traffic)
- **When to pay down:**
  - **NOW:** any MD5/SHA-1 password hashing, missing CSRF, public admin endpoints
  - **This quarter:** MFA, security headers, rate limits
  - **Ongoing:** authz coverage in CI (e.g., test that every authenticated route asserts a policy)

**Tip:** run `https://securityheaders.com` against staging at least once per quarter — it's free, fast, and surfaces missing headers immediately.

Reference: [OWASP — Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html) · [OWASP — Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html) · [Mozilla Observatory](https://observatory.mozilla.org/)
