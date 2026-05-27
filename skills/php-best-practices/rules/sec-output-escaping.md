---
title: Escape by Output Context
impact: CRITICAL
impactDescription: Security best practices and input safety
tags: php, security, validation
---

# Escape by Output Context

## Why it matters
XSS attacks execute because data is inserted into a context without the escaping rules that context requires. Applying HTML escaping to a JavaScript string literal does not prevent XSS; it just changes which payload the attacker uses. Context-wrong escaping is functionally equivalent to no escaping.

## Rule
Escape by the context where the value will appear: HTML text, HTML attribute, JavaScript, URL query parameter, or shell argument. Apply the correct function for each context at the point of output.
Escaping is not an input-normalization strategy and not a storage format. Do not pre-escape values before persistence or business comparisons.

## Bad
```php
<?php

declare(strict_types=1);

// Raw user values in multiple contexts — XSS in all of them
echo "<h1>Hello {$user->name}</h1>";
echo "<input value='{$query}'>";
echo "<script>var q = '{$query}';</script>";
echo "<a href='/search?q={$query}'>Go</a>";
```

## Better
```php
<?php

declare(strict_types=1);

// HTML context escaped, but other contexts still raw
$safeName = htmlspecialchars($user->name, ENT_QUOTES, 'UTF-8');
echo "<h1>Hello {$safeName}</h1>";

// JavaScript and URL contexts still unsafe
echo "<script>var q = '{$query}';</script>";       // still vulnerable
echo "<a href='/search?q={$query}'>Go</a>";        // still vulnerable
```

## Best
```php
<?php

declare(strict_types=1);

// Convenience wrapper — use everywhere for HTML text/attributes
function e(string $value): string
{
    return htmlspecialchars($value, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8');
}

// HTML body text
echo '<h1>Hello ' . e($user->name) . '</h1>';

// HTML attribute (double-quoted)
echo '<input value="' . e($query) . '">';

// JavaScript string literal — json_encode is the correct escaper
$jsValue = json_encode($user->name, JSON_HEX_TAG | JSON_HEX_AMP | JSON_HEX_APOS | JSON_HEX_QUOT);
echo "<script>var name = {$jsValue};</script>";

// URL query parameter
echo '<a href="/search?q=' . urlencode($query) . '">Go</a>';

// Full URL — validate scheme before embedding
$url = filter_var($redirectUrl, FILTER_VALIDATE_URL);
if ($url === false || !str_starts_with($url, 'https://')) {
    throw new \InvalidArgumentException('Invalid redirect URL');
}
echo '<a href="' . e($url) . '">Continue</a>';

// Shell argument — escapeshellarg wraps in single quotes
$escaped = escapeshellarg($filename);
exec("convert {$escaped} output.png");

// Template engines handle this automatically when used correctly:
// Blade: {{ $name }}          → HTML-escaped
// Blade: {!! $trustedHtml !!} → raw (use only for pre-sanitized HTML)
// Twig:  {{ name }}           → HTML-escaped
```

## Exceptions / trade-offs
CMS systems that store and render rich HTML need to emit raw content — restrict that to a `{!! !!}` / `|raw` path and ensure the stored content passed through a trusted HTML sanitizer (e.g. HTMLPurifier) at save time, not at render time.

## Static-analysis notes
PHPStan and Psalm do not track taint by default; the `psalm/plugin-taint-analysis` package (Psalm) or Taint extensions add data-flow tracking that can flag unescaped output paths.

## Security notes
`htmlspecialchars` without `ENT_QUOTES` leaves single-quoted attributes vulnerable. Always pass `ENT_QUOTES | ENT_SUBSTITUTE` and an explicit encoding. Never use `htmlentities` — it encodes characters that do not need escaping and may behave inconsistently across encodings.

## Related topics
- [sec-input-validation.md](sec-input-validation.md) — validation and escaping are complementary layers
- [sec-sql-prepared.md](sec-sql-prepared.md) — context-specific protection for SQL
