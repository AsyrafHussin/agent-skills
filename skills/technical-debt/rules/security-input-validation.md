---
title: Missing Input Validation
impact: CRITICAL
impactDescription: "Foundational defense; injection vectors compound silently across endpoints"
tags: security, validation, injection
---

## Missing Input Validation

**Impact: CRITICAL (Foundational defense; injection vectors compound silently across endpoints)**

Untrusted input flowing into queries, templates, file paths, or shell commands is the source of most OWASP top-10 vulnerabilities. Every endpoint that accepts external input without explicit validation is debt — and it compounds because each new endpoint adds another opportunity.

## How to Detect

```bash
# Laravel: controllers reaching directly into request without FormRequest
grep -rEn '\$request->(input|get|all)\b' app/Http/Controllers/ | \
  grep -v 'FormRequest\|validated('

# Express / Node: handlers using req.body / req.query without zod/joi/yup
grep -rEn 'req\.(body|query|params)' src/ | \
  grep -vE 'parse\(|validate\(|safeParse'

# SQL string concatenation (always bad) — grep's --include uses fnmatch, not brace expansion
grep -rEn '(SELECT|INSERT|UPDATE|DELETE).*\+.*\$|\\?.*concat' \
  --include='*.ts' --include='*.tsx' --include='*.php' --include='*.js' .

# Shell-out from app code (path for command injection)
grep -rEn 'exec\(|shell_exec\(|proc_open\(|spawn\(' \
  --include='*.ts' --include='*.php' --include='*.js' .
```

Also look at audit log coverage: every controller endpoint should map to an explicit validation rule set.

## Incorrect

```typescript
// ❌ Direct use of request input — type cast is not validation
app.get('/users', async (req, res) => {
  const limit  = parseInt(req.query.limit as string);
  const search = req.query.q as string;
  const [rows] = await pool.query(
    `SELECT * FROM users WHERE name LIKE '%${search}%' LIMIT ${limit}`,    // SQL injection
  );
  res.json(rows);
});
```

```php
// ❌ Laravel: same problem, different stack
public function index(Request $request) {
    $sort = $request->input('sort');            // attacker controls
    $users = DB::select("SELECT * FROM users ORDER BY $sort");
    return response()->json($users);
}
```

**Problems:**
- SQL injection in both examples (no parameterization, no allowlist)
- `parseInt` of attacker input returns NaN for non-numbers — `LIMIT NaN` errors leak SQL structure
- The "cast as string" in TypeScript provides zero runtime validation

## Correct

```typescript
// ✅ Zod schema + parameterized query
import { z } from 'zod';

const ListUsersQuery = z.object({
  limit: z.coerce.number().int().min(1).max(100).default(20),
  q:     z.string().max(80).optional(),
});

// Using `mysql2/promise` — `?` placeholders, parameterized by the driver.
// We use `pool.query()` (client-side escaping) rather than `pool.execute()`
// (server-side prepared statements) because mysql2's prepared statements
// can fail to bind JS numbers to `LIMIT ?` (ER_WRONG_ARGUMENTS in some
// MySQL versions). With `query()` mysql2 escapes the number safely.
// Also: MySQL's default collation (`utf8mb4_0900_ai_ci`) is case-insensitive,
// so plain LIKE matches both 'Asyraf' and 'asyraf' without `LOWER(...)`.
import mysql from 'mysql2/promise';
const pool = mysql.createPool({ /* ... */ });

app.get('/users', async (req, res) => {
  const parsed = ListUsersQuery.safeParse(req.query);
  if (!parsed.success) return res.status(400).json(parsed.error.flatten());

  const { limit, q } = parsed.data;
  const [rows] = await pool.query(
    'SELECT * FROM users WHERE (? IS NULL OR name LIKE ?) LIMIT ?',
    [q ?? null, q ? `%${q}%` : null, limit],
  );
  res.json(rows);
});
```

```php
// ✅ Laravel FormRequest with allowlisted sort
final class ListUsersRequest extends FormRequest
{
    public function rules(): array {
        return [
            'sort' => ['nullable', Rule::in(['name', 'created_at'])],
            'limit' => ['integer', 'min:1', 'max:100'],
        ];
    }
}

public function index(ListUsersRequest $request) {
    $sort = $request->validated('sort', 'created_at');
    return DB::table('users')->orderBy($sort)->paginate($request->validated('limit', 20));
}
```

**Benefits:**
- Bad input → 400 with a clear message, never reaches the database
- SQL injection eliminated by parameterization + allowlist
- Validation is a single auditable location per endpoint

## Remediation Strategy

- **Effort:** S per endpoint
- **When to pay down:**
  - **NOW:** any endpoint that takes input into a raw SQL string, shell command, or file path
  - **This sprint:** all unvalidated endpoints in critical paths (auth, payment, profile)
  - **Then:** lint rules that fail PRs lacking validation schemas
- **Tip:** put validation at the boundary (controller / route handler), then trust the validated shape downstream. Don't re-validate the same fields in 5 places.

Reference: [OWASP — Input Validation Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html) · [Laravel Validation](https://laravel.com/docs/validation) · [Zod](https://zod.dev/)
