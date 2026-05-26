---
title: Deprecated Framework APIs
impact: HIGH
impactDescription: "Deprecations become breakages on the next major upgrade"
tags: framework, deprecations, upgrade
---

## Deprecated Framework APIs

**Impact: HIGH (Deprecations become breakages on the next major upgrade)**

Every deprecation warning is the framework telling you exactly what will break in the next major release. Ignoring them doesn't postpone the cost — it concentrates it on whoever does the upgrade, who is now blocked by hundreds of issues at once.

## How to Detect

```bash
# Laravel
php artisan about                           # framework version
grep -rn '@deprecated' vendor/laravel/      # known deprecated APIs
# Run tests with deprecation reporting on:
APP_ENV=testing E_DEPRECATED=on php artisan test

# React
# React DevTools logs deprecations in console
# eslint-plugin-react flags many deprecated APIs:
npx eslint . --rule 'react/no-deprecated: error'

# Node
node --pending-deprecation app.js          # surface upcoming deprecations
node --throw-deprecation app.js            # treat them as errors in CI

# Generic: parse build/test logs
npm test 2>&1 | grep -iE 'deprecat|warning' | sort -u
```

## Incorrect

```php
// ❌ Laravel deprecated API usage left in code
use Illuminate\Support\Facades\Input;       // deprecated in Laravel 5.x; removed in 6.0
$name = Input::get('name');

// ❌ Method signature that's been overhauled
public function failed(Exception $e)        // Laravel 10+ expects ?Throwable
{
    // ...
}

// ❌ Deprecated React lifecycle method
class UserList extends React.Component {
  componentWillMount() {                    // deprecated since React 16.3
    this.fetch();
  }
}
```

**Problems:**
- Next Laravel major: `Input` gone; `failed(Throwable)` enforced — both break at once
- React 18 strict mode logs warnings; React 19 may remove them entirely
- Deprecation log noise hides genuine warnings

## Correct

```php
// ✅ Use current APIs
$name = request()->input('name');

public function failed(?Throwable $e): void
{
    // ...
}
```

```typescript
// ✅ Hooks or current lifecycle methods
function UserList() {
  useEffect(() => { fetch(); }, []);
}
```

Add a CI gate:

```yaml
- name: No deprecation warnings
  run: |
    OUTPUT=$(npm test 2>&1 || true)
    echo "$OUTPUT" | grep -qiE 'deprecat' && { echo "Deprecations found"; exit 1; }
    exit 0
```

**Benefits:**
- Next major upgrade is hours, not weeks
- Test output is signal again
- Framework authors' migration notes apply directly without rediscovery

## Remediation Strategy

- **Effort:** S–M per deprecation (the migration path is usually documented by the framework)
- **When to pay down:**
  - **Read the framework's upgrade guide** when they ship a deprecation — pay down what you can immediately
  - **CI gate:** zero new deprecation warnings allowed; old ones whittled down per sprint
- **Order of operations:** fix deprecations *before* attempting the major upgrade — never together

Reference: [Laravel Upgrade Guide](https://laravel.com/docs/upgrade) · [React Strict Mode](https://react.dev/reference/react/StrictMode) · [Node Deprecations](https://nodejs.org/api/deprecations.html)
