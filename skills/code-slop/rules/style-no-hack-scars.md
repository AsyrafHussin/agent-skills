---
title: No HACK Scars — Suspiciously Pristine Code
impact: MEDIUM
impactDescription: "Real codebases have geology; absence of any '// HACK:' / '// XXX:' markers is itself a tell"
tags: style, geology, ai-fingerprint
---

## No HACK Scars — Suspiciously Pristine Code

**Impact: MEDIUM (Real codebases have geology; absence of any '// HACK:' / '// XXX:' markers is itself a tell)**

A real codebase has *scars*. `// HACK:` comments where someone went around the type system to ship. `// XXX:` markers where the engineer noticed something fishy but had to ship. Files with a "we tried it the right way; this is the workaround until Stripe v2" note. These markers signal:

- A human engineered the code, hit a real constraint, and acknowledged the trade-off
- The team agreed to ship with the trade-off rather than over-engineer
- A future engineer is being warned

AI-generated codebases are flat: no `// HACK:`, no `// XXX:`, no "we know this is wrong but" markers. Every line presented as equally polished. The *absence* of scars is the slop signal — real systems don't look like this.

## The pattern

You can audit a repo's "humanity" by looking for these markers:

```bash
git grep -E '// (HACK|XXX|NOTE|GOTCHA|HMMMM|WTF)' | wc -l
```

A repo with 50 of these is alive. A repo with zero across a 50K-line codebase is either:
- A library that was rigorously reviewed across many years (rare)
- An AI-generated codebase passing as human (more common)

You're not looking for high counts — you're looking for **some**. Two or three `// HACK:` markers in 5K LoC is healthy.

## What healthy scars look like

```php
// ✅ Real human marker — explains the constraint
public function chargeCustomer(int $cents, string $token): string
{
    // HACK: Stripe v1 SDK doesn't expose retry headers; manually parse from Response
    // until we migrate to v2 (tracked in #1842).
    try {
        return $this->stripe->charges->create([...])->id;
    } catch (RateLimitException $e) {
        $retryAfter = (int) ($e->getResponse()->headers['Retry-After'] ?? 30);
        sleep($retryAfter);
        return $this->stripe->charges->create([...])->id;
    }
}
```

```typescript
// ✅ XXX marker that warns the next reader
function parseEnvNumber(key: string): number {
  const raw = process.env[key];
  // XXX: parseInt('1.5') === 1, parseFloat('1.5e10') === 15000000000 — fine for us
  // because we only use this for ports and integer caps. Audit before reusing.
  return parseInt(raw ?? '0', 10);
}
```

```php
// ✅ NOTE marker capturing tribal knowledge
class OrderObserver
{
    public function creating(Order $order): void
    {
        // NOTE: tax_amount must be set BEFORE Stripe charge; the webhook handler
        // re-reads this row and expects it to be non-zero.
        $order->tax_amount = TaxCalculator::for($order);
    }
}
```

These tell the next engineer: *"someone thought about this; the workaround is intentional; if you change it, here's the context."*

## What absence looks like

Five files, 600 lines, all in a PR:

- Every method has the same length and shape
- Every error path is the same `try { ... } catch (e) { console.error(e); }`
- No comments about edge cases, browser quirks, library bugs, race conditions
- No `// HACK:` / `// XXX:` / `// NOTE:` anywhere
- Every variable has a "perfect" name

That's not human. Real production code dealing with real third parties has scars.

## What to do when you find pristine-looking code

This rule is harder to action than to detect. Recommended response:

1. **Pair-review with the author** — ask them to walk you through one of the "perfect" sections. If they can articulate the *why* (and the trade-offs they considered), it's fine. If they can't, treat it as AI-assisted code that needs deeper review.
2. **Run mutation testing on the new code** — Stryker (TS) / Infection (PHP). Mirror-tests die fast.
3. **Sample edge cases** — try inputs the AI wouldn't have thought of (empty string, very long string, Unicode, negative numbers, future dates, leap years). If the code falls over on basics, the polish was cosmetic.

## Detection

```bash
# Count scars in the existing repo (the baseline)
git grep -cE '// (HACK|XXX|NOTE|GOTCHA|WTF|FIXME)' -- '*.ts' '*.tsx' '*.php' '*.js' '*.jsx' 2>/dev/null | head

# Same, but for the PR diff only — if a 1000-line PR adds zero scars, suspicious
git diff origin/main...HEAD -- '*.ts' '*.tsx' '*.php' | \
  grep -cE '^\+.*// (HACK|XXX|NOTE|GOTCHA|WTF)'
```

A PR adding > 500 LoC with zero `HACK:` / `XXX:` / `NOTE:` is unusual. Real engineering on real systems leaves these.

**Note:** this rule cuts both ways. Don't *add* a `// HACK:` just to look human. Add them when you actually have a hack to mark. The signal is genuine, not performative.

Reference: Internal: [`style-hyper-consistent`](style-hyper-consistent.md)
