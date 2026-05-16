# Technical Debt - Complete Reference

**Version:** 1.0.0
**Organization:** Agent Skills Contributors
**Date:** May 2026
**License:** MIT

## Abstract

Technical debt audit and prioritization framework for **PHP/Laravel (MySQL) and Node/TypeScript/React** projects. Contains 42 rules across 10 categories covering code, security, design, dependency, test, performance, data, documentation, infrastructure, and process debt. Produces a ranked debt ledger (effort × impact) so teams know what to pay down first — not just what's broken. Supports audit mode with PASS/FAIL/N/A checklist output and priority ranking (P0–P3). Each rule includes detection commands, incorrect and correct examples, and a remediation strategy with effort estimate.

## How to Audit

When asked to "audit technical debt", "find tech debt", or "what should we refactor first", run through each rule in this document as a checklist. For each item output **PASS**, **FAIL** (with `file:line` or command output, **effort** S/M/L, and **impact** LOW/MED/HIGH/CRITICAL), or **N/A**. End with a prioritized debt ledger (sorted by impact then effort) and a summary of pass/fail counts.

## References

- [Martin Fowler — Technical Debt Quadrant](https://martinfowler.com/bliki/TechnicalDebtQuadrant.html)
- [Ward Cunningham — The Debt Metaphor](https://www.youtube.com/watch?v=pqeJFYwnkjE)
- [SonarQube Technical Debt Model](https://docs.sonarsource.com/sonarqube/latest/user-guide/metric-definitions/)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [OWASP Dependency-Check](https://owasp.org/www-project-dependency-check/)
- [Snyk Vulnerability Database](https://security.snyk.io/)
- [gitleaks](https://github.com/gitleaks/gitleaks)
- [Use the Index, Luke](https://use-the-index-luke.com/)
- [MySQL Reference Manual](https://dev.mysql.com/doc/refman/8.4/en/)
- [Google SRE Book](https://sre.google/sre-book/table-of-contents/)
- [OpenTelemetry](https://opentelemetry.io/)

## Step 1: Detect Project Stack

**Always detect the stack before running tooling.** This skill targets PHP / Laravel (with MySQL) and Node / TypeScript / React projects; detection commands below assume one or both are present.

| Signal | Stack | Tooling |
|--------|-------|---------|
| `composer.json` present | PHP / Laravel | `composer outdated`, `composer audit`, `phpstan`, `phpcs`, `phpmd`, `deptrac` |
| `package.json` present | Node / JS / TS / React | `npm outdated`, `npm audit`, `eslint`, `tsc --noEmit`, `knip`, `madge` |
| MySQL connection available | Database | `EXPLAIN`, `sys.schema_tables_with_full_table_scans`, `sys.schema_unused_indexes`, `sys.statement_analysis` |
| any repo | Secrets scan | `gitleaks git`, `trufflehog` |

If both stacks are present (e.g., Laravel + Inertia + React), run audits for each.

---

# Sections

This file defines all sections, their ordering, impact levels, and descriptions.
The section ID (in parentheses) is the filename prefix used to group rules.

---

## 1. Code Debt (code)

**Impact:** CRITICAL
**Description:** Debt within the code itself — duplication, complex methods, oversized classes, dead code, magic literals, and long parameter lists. These are the most direct drivers of bug density and slow feature work, and they compound fastest as the codebase grows.

## 2. Security Debt (security)

**Impact:** CRITICAL
**Description:** Accumulated security gaps — secrets in code, missing input validation, outdated auth defaults, and missing hardening. Security debt is the category most likely to become a *public* problem; what's tolerable as a backlog item today is tomorrow's breach disclosure.

## 3. Design Debt (design)

**Impact:** HIGH
**Description:** Structural debt across modules — tight coupling, circular dependencies, leaky abstractions, and changes that ripple through many files. Design debt makes refactoring expensive and concentrates risk in small changes.

## 4. Dependency Debt (deps)

**Impact:** HIGH
**Description:** Debt in third-party packages — outdated versions, abandoned libraries, known CVEs, and unused dependencies. Dependency debt is the cheapest debt to detect (tooling does it for you) and the most dangerous to ignore (it grows by itself even when you don't touch the code).

## 5. Test Debt (test)

**Impact:** HIGH
**Description:** Gaps in test coverage, flaky or disabled tests, and slow suites that erode confidence in CI. Test debt directly slows the team — a flaky or slow suite costs every engineer every day.

## 6. Performance Debt (perf)

**Impact:** HIGH
**Description:** N+1 queries, unbounded result sets, and bundle bloat. Performance debt looks fine in development and explodes proportionally to your most successful customer. It is the category most likely to convert directly into lost revenue.

## 7. Data Debt (data)

**Impact:** HIGH
**Description:** Schema drift, missing indexes, and referential-integrity gaps. Data debt is the slowest to detect and the hardest to fix — every migration after the drift is a roll of the dice, and orphaned records propagate into reports nobody trusts.

## 8. Documentation Debt (docs)

**Impact:** MEDIUM
**Description:** Stale comments, outdated READMEs, and undocumented public APIs. Documentation debt is invisible until onboarding or incident response, where it suddenly becomes the bottleneck.

## 9. Infrastructure Debt (infra)

**Impact:** MEDIUM
**Description:** EOL runtimes, deprecated framework APIs, accumulated build warnings, insecure secrets handling, and observability gaps. Infrastructure debt has hard deadlines (CVEs, vendor EOL dates) and is non-negotiable once they hit.

## 10. Process Debt (process)

**Impact:** MEDIUM
**Description:** Aging TODO/FIXME comments, `@deprecated` markers without removal plans, untracked debt, and code without owners. Process debt is about visibility and accountability — debt you can't see, or that nobody owns, cannot be prioritized.

---


## Code Duplication

**Impact: CRITICAL (Bug fixes multiply across copies; behaviour drifts silently)**

Duplicated logic means every bug fix and behaviour change must be made in N places. Copies drift over time, producing inconsistent behaviour that is hard to detect and harder to test.

## How to Detect

```bash
# Find duplicated blocks across the repo (multi-language)
npx jscpd --min-lines 30 --min-tokens 100 src/
npx jscpd --languages php --min-lines 30 app/         # PHP support via jscpd

# PHP-specific (note: sebastian/phpcpd was archived in 2023 — prefer jscpd above)
# vendor/bin/phpcpd app/                              # legacy projects only

# Language-agnostic
git ls-files | xargs sha1sum | sort | uniq -d -w 40   # exact file dupes
```

Threshold: any duplicated block longer than **30 lines** or appearing in **3+ locations** counts as debt.

## Incorrect

```php
// ❌ Bad: same calculation duplicated in two controllers
// app/Http/Controllers/OrderController.php
public function summary(Order $order) {
    $subtotal = $order->items->sum(fn($i) => $i->price * $i->quantity);
    $tax = $subtotal * 0.06;
    $shipping = $subtotal > 100 ? 0 : 15;
    return ['total' => $subtotal + $tax + $shipping];
}

// app/Http/Controllers/CartController.php
public function checkout(Cart $cart) {
    $subtotal = $cart->items->sum(fn($i) => $i->price * $i->quantity);
    $tax = $subtotal * 0.06;
    $shipping = $subtotal > 100 ? 0 : 15;
    return ['total' => $subtotal + $tax + $shipping];
}
```

**Problems:**
- Tax rate change requires editing both files (and any others that copy this)
- One copy can drift (e.g., free-shipping threshold raised in one but not the other)
- No single place to add tests for pricing rules

## Correct

```php
// ✅ Single source of truth
final class PricingCalculator
{
    public function __construct(private TaxRate $tax, private ShippingPolicy $shipping) {}

    public function total(iterable $items): Money
    {
        $subtotal = collect($items)->sum(fn($i) => $i->price * $i->quantity);
        return $subtotal + $this->tax->for($subtotal) + $this->shipping->for($subtotal);
    }
}

// Both controllers inject and call PricingCalculator::total()
```

**Benefits:**
- One place to change tax/shipping rules
- One target for unit tests
- Behaviour cannot drift between call sites

## Remediation Strategy

- **Effort:** S–M (depends on number of duplicates and parameter variation)
- **When to pay down:** Before the next behaviour change touches *any* of the copies — the next edit pays for the refactor.

Reference: [Martin Fowler — Duplicated Code](https://refactoring.com/catalog/extractFunction.html)

---


## Cyclomatic and Cognitive Complexity

**Impact: CRITICAL (High-complexity methods harbor most production bugs)**

Methods with cyclomatic complexity > 10 (or cognitive complexity > 15) are statistically the strongest predictor of bug density in a codebase. They are also the slowest to test, review, and modify.

## How to Detect

```bash
# PHP
vendor/bin/phpmd app text codesize --reportfile complexity.txt

# JavaScript / TypeScript
npx eslint . --rule 'complexity: ["error", 10]'
```

Threshold: **cyclomatic complexity > 10** OR **cognitive complexity > 15** OR **nesting depth > 4**.

## Incorrect

```typescript
// ❌ Cyclomatic complexity 14 — every branch combination is a separate path
function calculateDiscount(user: User, order: Order): number {
  if (user.tier === 'gold') {
    if (order.total > 1000) {
      if (order.itemCount > 10) {
        if (user.yearsActive > 5) return 0.30;
        else return 0.25;
      } else if (order.itemCount > 5) {
        return 0.20;
      }
      return 0.15;
    } else if (order.total > 500) {
      return user.yearsActive > 2 ? 0.12 : 0.10;
    }
    return 0.05;
  } else if (user.tier === 'silver') {
    // ... another 20 lines of nested ifs
  }
  return 0;
}
```

**Problems:**
- 14+ independent code paths — minimum 14 tests to cover them all
- Adding a new tier requires understanding every nested branch
- Reviewers cannot hold the state in their head

## Correct

```typescript
// ✅ Replace nested conditions with a lookup table + small predicates
type DiscountRule = { match: (u: User, o: Order) => boolean; rate: number };

const DISCOUNT_RULES: DiscountRule[] = [
  { match: (u, o) => u.tier === 'gold' && o.total > 1000 && o.itemCount > 10 && u.yearsActive > 5, rate: 0.30 },
  { match: (u, o) => u.tier === 'gold' && o.total > 1000 && o.itemCount > 10,                       rate: 0.25 },
  { match: (u, o) => u.tier === 'gold' && o.total > 1000 && o.itemCount > 5,                        rate: 0.20 },
  // ...
];

function calculateDiscount(user: User, order: Order): number {
  return DISCOUNT_RULES.find(r => r.match(user, order))?.rate ?? 0;
}
```

**Benefits:**
- Each rule is independently testable
- New tiers/rules added by appending — no nested-branch surgery
- Cyclomatic complexity of `calculateDiscount` drops to 1

## Remediation Strategy

- **Effort:** S–M per method
- **When to pay down:** The next time you need to add a branch to a complexity-flagged method. Extract first, modify after.

Reference: [SonarSource — Cognitive Complexity](https://www.sonarsource.com/resources/cognitive-complexity/)

---


## Long Functions and Methods

**Impact: CRITICAL (Long methods hide multiple responsibilities and resist testing)**

A function longer than ~50 lines almost always does more than one thing. It cannot be unit-tested in isolation, cannot be named accurately, and is the most common location for "scary code nobody touches".

## How to Detect

```bash
# Find functions over 50 lines (rough heuristic)
grep -rEn 'function |def |func ' --include='*.{php,ts,js,py,go}' | \
  awk -F: '{ print $1 ":" $2 }' | \
  while read line; do : ; done   # combine with editor stats or:

# PHP
vendor/bin/phpmd app text codesize  # reports ExcessiveMethodLength (default 100; lower to 50 via custom ruleset)
#   <property name="minimum" value="50"/>   in the ExcessiveMethodLength rule reference

# TS / JS
npx eslint . --rule 'max-lines-per-function: ["error", 50]'
```

Threshold: **method > 50 lines** OR **function > 50 lines** (excluding comments and blank lines).

## Incorrect

```php
// ❌ 180-line controller action doing fetch, validate, transform, persist, notify
public function checkout(Request $request)
{
    // 20 lines of input validation
    $data = $request->validate([ /* ... */ ]);

    // 30 lines of cart calculation
    $cart = Cart::findOrFail($data['cart_id']);
    $subtotal = 0;
    foreach ($cart->items as $item) { /* ... */ }
    $tax = /* ... */;

    // 25 lines of payment processing
    $charge = Stripe::charges()->create(/* ... */);
    if ($charge->status !== 'succeeded') { /* ... */ }

    // 40 lines of order creation + side effects
    $order = Order::create(/* ... */);
    foreach ($cart->items as $item) { /* ... */ }

    // 20 lines of notifications
    Mail::to($cart->user)->send(new OrderConfirmation($order));
    Slack::notify('#sales', /* ... */);

    return response()->json(/* ... */);
}
```

**Problems:**
- Cannot test calculation logic without mocking Stripe and Mail
- Reviewer must understand the entire flow to verify a one-line change
- Side effects (mail, Slack) hidden inside a request handler

## Correct

```php
// ✅ Each responsibility extracted; controller orchestrates only
public function checkout(CheckoutRequest $request, CheckoutService $service)
{
    $order = $service->process(
        cart: Cart::findOrFail($request->validated('cart_id')),
        paymentToken: $request->validated('payment_token'),
    );
    return new OrderResource($order);
}

// CheckoutService::process() is itself a short orchestrator that calls:
//   PricingCalculator, PaymentGateway, OrderRepository, NotificationDispatcher
```

**Benefits:**
- Controller is 5 lines, trivially testable as an HTTP wrapper
- Pricing, payment, persistence, notifications each have a focused unit test
- Adding a new notification channel changes one class

## Remediation Strategy

- **Effort:** S–M (Extract Method is mechanical; the hard part is naming)
- **When to pay down:** Whenever you need to add a feature inside a long method, extract first.

Reference: [Refactoring — Extract Function](https://refactoring.com/catalog/extractFunction.html)

---


## God Classes

**Impact: CRITICAL (One-class kingdoms become merge-conflict and bug magnets)**

A class with too many responsibilities (often called a "god class") attracts every change in the system. It becomes the file with the most commits, the most authors, and the most bugs — and it blocks parallel work.

## How to Detect

```bash
# Files larger than 300 lines (rough heuristic)
find . -type f \( -name '*.php' -o -name '*.ts' \) -exec wc -l {} \; | \
  awk '$1 > 300 { print }' | sort -rn

# PHP
vendor/bin/phpmd app text codesize  # reports ExcessiveClassLength, TooManyMethods
#   PHPMD defaults are looser (1000 LoC, 25 methods); for stricter 300/15 limits, use a custom
#   ruleset overriding `<property name="minimum" .../>` on those rules.

# Hotspot detection — files changed most often
git log --since='12 months ago' --name-only --pretty=format: | \
  sort | uniq -c | sort -rn | head -20
```

Thresholds: **> 300 lines**, **> 15 public methods**, OR **> 7 dependencies in the constructor**.

## Incorrect

```php
// ❌ OrderService has 820 lines, 27 public methods, depends on 12 services
final class OrderService
{
    public function __construct(
        private OrderRepo $orders,
        private CartRepo $carts,
        private PaymentGateway $payments,
        private TaxCalculator $tax,
        private ShippingService $shipping,
        private InventoryService $inventory,
        private NotificationService $notifications,
        private AnalyticsService $analytics,
        private FraudCheckService $fraud,
        private LoyaltyService $loyalty,
        private InvoiceService $invoices,
        private RefundService $refunds,
    ) {}

    public function create(...)    { /* 80 lines */ }
    public function update(...)    { /* 60 lines */ }
    public function cancel(...)    { /* 70 lines */ }
    public function refund(...)    { /* 90 lines */ }
    public function ship(...)      { /* ... */ }
    public function calculateTax(...) { /* ... */ }
    // ... 21 more methods
}
```

**Problems:**
- Every team that touches orders edits this one file → constant merge conflicts
- Unit tests must mock 12 collaborators just to instantiate it
- A bug in refund logic puts the entire order flow at risk to deploy

## Correct

```php
// ✅ Split by lifecycle stage / responsibility
final class OrderPlacement       { /* create() */ }
final class OrderFulfillment     { /* ship(), markDelivered() */ }
final class OrderCancellation    { /* cancel() */ }
final class OrderRefund          { /* refund() */ }
final class OrderTaxCalculation  { /* calculateTax() */ }

// Each has 2–4 dependencies and 30–100 LoC
```

**Benefits:**
- Teams can work on different stages in parallel without conflicts
- Each class has a focused test suite with minimal mocking
- Bug in refund logic blocks only the refund deploy, not new orders

## Remediation Strategy

- **Effort:** L (almost always — break into multiple PRs)
- **When to pay down:** Identify the highest-churn god class first (`git log` hotspots). Carve off one responsibility per PR — do not attempt a single big-bang refactor.

Reference: [Martin Fowler — Large Class](https://refactoring.com/catalog/extractClass.html)

---


## Dead Code

**Impact: HIGH (Unused code misleads readers and inflates maintenance surface)**

Dead code — unused exports, unreachable branches, commented-out blocks — costs nothing to delete and costs a lot to keep. Readers assume code that exists is code that runs; dead code wastes attention and creates phantom dependencies that block upgrades.

## How to Detect

```bash
# TypeScript / JavaScript
npx knip                     # unused files, exports, deps (preferred; ts-prune is archived)
npx eslint . --rule 'no-unreachable: error'

# PHP
vendor/bin/phpstan analyse --level=9  # detects unreachable code and unused private elements
#   For broader dead-code detection, add: tomasvotruba/unused-public, or use Rector's DeadCodeSetList

# Commented-out code (one --include per extension; grep doesn't expand braces)
grep -rEn '^\s*//.*[;{}]$' --include='*.php' --include='*.ts' --include='*.tsx' --include='*.js' .
```

## Incorrect

```typescript
// ❌ Dead imports, dead helper, dead branch, commented-out block
import { legacyFormatter } from './legacy';   // never used after v2 rewrite
import { format } from './format';

function formatPrice(p: number, currency: string) {
  // const oldImpl = (p) => `$${p.toFixed(2)}`;     // kept "just in case"
  // if (currency === 'BTC') return formatBtc(p);  // BTC support removed 2023

  if (currency === 'USD') return format(p, 'USD');
  if (currency === 'EUR') return format(p, 'EUR');
  return format(p, 'USD');
  return formatLegacy(p);   // unreachable
}

export function formatLegacy() { /* called nowhere */ }
```

**Problems:**
- Reader has to puzzle out whether the commented BTC branch is coming back
- `formatLegacy` blocks deleting the `./legacy` module
- The unreachable `return` raises false suspicion during reviews

## Correct

```typescript
// ✅ Delete it. Git remembers.
import { format } from './format';

function formatPrice(p: number, currency: string): string {
  if (currency === 'EUR') return format(p, 'EUR');
  return format(p, 'USD');
}
```

**Benefits:**
- No phantom dependency on the legacy module
- Reader sees only what runs
- Diff in `git log` documents *when* and *why* BTC was removed — better than a stale comment

## Remediation Strategy

- **Effort:** S (deletion is mechanical; trust git history)
- **When to pay down:** Immediately on detection — there is no reason to keep dead code in main.

**Note:** Resist the urge to keep "might-be-useful-later" code commented out. If you genuinely need it later, restore it from git history. The cost of a `git revert` is far less than the cost of confusing every future reader.

Reference: [Refactoring — Remove Dead Code](https://refactoring.guru/smells/dead-code)

---


## Magic Numbers and Hardcoded Literals

**Impact: MEDIUM (Obscure intent; require coordinated edits across files when changed)**

A `0.06` in tax code is invisible business knowledge. The next time the tax rate changes — or the next reader who needs to understand the rule — pays the cost. Magic numbers also make the same value drift across copies (one file uses `0.06`, another `0.065`).

## How to Detect

```bash
# TypeScript / JavaScript
npx eslint . --rule 'no-magic-numbers: ["error", { "ignore": [0, 1, -1] }]'

# PHP — PHPMD has no built-in magic-number rule; use a Psalm/PHPStan extension
# or a custom PHPCS sniff. Closest built-ins:
vendor/bin/phpstan analyse --level=8                       # catches some via type-aware analysis
# Custom: a project-local PHPCS sniff for hardcoded literals in *Service* / *Calculator* classes

# Cross-language grep for suspicious literals in business logic
grep -rEn '\b[0-9]+\.[0-9]+\b' app/Services/ src/services/ | grep -v test
```

Threshold: any non-trivial literal (anything except 0, 1, -1, and indexes used for slicing) appearing in business logic — especially if it appears more than once.

## Incorrect

```typescript
// ❌ Bare numbers and strings scattered through business logic
export function calculateOrder(items: Item[], user: User): Order {
  const subtotal = items.reduce((s, i) => s + i.price * i.qty, 0);
  const tax = subtotal * 0.06;                          // what's 0.06?
  const shipping = subtotal > 100 ? 0 : 15;             // why 100? why 15?
  const cacheKey = `order:${user.id}:v3`;               // why v3?
  redis.set(cacheKey, JSON.stringify({ subtotal, tax, shipping }), 'EX', 3600);  // 3600 what?
  return { subtotal, tax, shipping };
}
```

**Problems:**
- "0.06" appears in 4 other files — tax rate change requires hunting them all
- A reader can't tell whether `3600` is seconds, milliseconds, or a row count
- `'v3'` is silent invariant — changing cache format requires knowing about every caller

## Correct

```typescript
// ✅ Named constants with units and intent
const TAX_RATE = 0.06;
const FREE_SHIPPING_THRESHOLD = 100;
const FLAT_SHIPPING_FEE = 15;
const CACHE_VERSION = 'v3';
const CACHE_TTL_SECONDS = 60 * 60;

export function calculateOrder(items: Item[], user: User): Order {
  const subtotal = items.reduce((s, i) => s + i.price * i.qty, 0);
  const tax = subtotal * TAX_RATE;
  const shipping = subtotal > FREE_SHIPPING_THRESHOLD ? 0 : FLAT_SHIPPING_FEE;
  const cacheKey = `order:${user.id}:${CACHE_VERSION}`;
  redis.set(cacheKey, JSON.stringify({ subtotal, tax, shipping }), 'EX', CACHE_TTL_SECONDS);
  return { subtotal, tax, shipping };
}
```

**Benefits:**
- Tax-rate change is one line
- Units are explicit (`60 * 60` reads as "seconds in an hour")
- Constants are searchable; renames are mechanical

## Remediation Strategy

- **Effort:** S per file (extract constant, replace references)
- **When to pay down:** When you next change a value (the change is now one line), or when you spot the same literal in 2+ places.
- **Where to put constants:** as module-level `const` for local values; in a shared `pricing/Config.ts` or `config/billing.php` for cross-module business values.

**Tip:** for genuinely tunable values (rates, thresholds, feature flags), put them in environment-driven config so changes don't require a deploy.

Reference: [Refactoring — Replace Magic Number with Symbolic Constant](https://refactoring.guru/replace-magic-number-with-symbolic-constant)

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

---


## Secrets in Source Code

**Impact: CRITICAL (Once committed, a secret must be rotated AND scrubbed — git history is forever)**

A secret committed to git is compromised the moment the commit lands, even if you delete it in the next commit. Public repos are crawled by bots within minutes; private repos leak through forks, backups, and CI logs. Rotation is mandatory — deletion alone is theater.

## How to Detect

```bash
# Scan current tree and full history for secrets
gitleaks git                                  # scan repo history
gitleaks git --pre-commit --staged            # pre-commit hook form (v8.19+)
trufflehog filesystem .                       # alternative scanner
trufflehog git file://. --only-verified       # verified live secrets

# Targeted grep
grep -rEn '(aws_secret|api_key|password|token)\s*=\s*["\047][A-Za-z0-9/+=_-]{16,}' .

# Pre-commit hook (gitleaks) — pin to the latest stable release tag
# .pre-commit-config.yaml
# - repo: https://github.com/gitleaks/gitleaks
#   rev: v8.30.1
#   hooks: [ { id: gitleaks } ]
```

## Incorrect

```php
// ❌ Hardcoded API keys, database passwords, signing keys
// config/services.php
return [
    'stripe' => [
        'secret' => 'sk_live_51Hxxxxxxxxxxxxxxxxxxx',         // committed to repo
    ],
    'aws' => [
        'key' => 'AKIAIOSFODNN7EXAMPLE',
        'secret' => 'wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY',
    ],
];

// .env.example with REAL values that got copied to .env and committed
DB_PASSWORD=ProductionDb2024!
JWT_SECRET=hardcoded-jwt-signing-key-do-not-use
```

**Problems:**
- The Stripe key is now public — Stripe will eventually rotate it, but not before charges go through
- The AWS key allows full account access until rotated; bots will find it
- "Look, I deleted it in the next commit" — irrelevant. It's in git history.

## Correct

```php
// ✅ Read from environment / secret manager
return [
    'stripe' => ['secret' => env('STRIPE_SECRET')],
    'aws' => [
        'key'    => env('AWS_ACCESS_KEY_ID'),
        'secret' => env('AWS_SECRET_ACCESS_KEY'),
    ],
];
```

```bash
# .env.example contains only placeholders
STRIPE_SECRET=
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=

# Real values live in:
#   - Doppler / Vault / AWS Secrets Manager / GCP Secret Manager (production)
#   - Local .env (gitignored)
#   - CI: GitHub Actions secrets, with OIDC for AWS where possible
```

CI gate:

```yaml
- uses: gitleaks/gitleaks-action@v2     # fails PR if any secret pattern detected
```

**Benefits:**
- Secrets can be rotated without code changes
- Audit logs show every access (with a real secret manager)
- New engineers cannot accidentally leak production credentials

## Remediation Strategy

- **Effort:**
  - **S** — move forward: env vars + pre-commit hook + CI gate
  - **M** — clean active code paths to use env / secret manager
  - **L** — scrub git history if secrets are old (use `git filter-repo` or BFG; coordinate with team)
- **When to pay down:**
  - **NOW:** any secret committed to a public repo — rotate first, then clean
  - **This sprint:** any committed secret in private repos
  - **Then:** install gitleaks pre-commit + CI gate to prevent regression

**Rotation checklist for any discovered secret:**
1. Revoke the secret at the issuer (Stripe, AWS, etc.)
2. Generate a replacement
3. Update production via secret manager
4. Remove from code + commit replacement source
5. Optionally scrub history (cost-benefit; sometimes rotation is enough)
6. Add a regex rule to gitleaks to prevent the same shape from re-entering

Reference: [GitGuardian — State of Secrets Sprawl](https://www.gitguardian.com/state-of-secrets-sprawl-report-2024) · [gitleaks](https://github.com/gitleaks/gitleaks) · [trufflehog](https://github.com/trufflesecurity/trufflehog)

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

---


## Tight Coupling

**Impact: HIGH (Changes ripple unpredictably; modules cannot be replaced)**

Tight coupling means modules depend on each other's internals. A change in one ripples into many, and you can't swap an implementation without rewriting consumers. The symptom: small features take days, simple bug fixes break unrelated tests.

## How to Detect

- Count cross-module imports — modules importing > 10 files from other modules are suspect
- Look for direct instantiation (`new X()`) of cross-module dependencies in business logic
- Search for reaches into private/internal namespaces

```bash
# Find imports from too many other modules
grep -rE "^(import|use|require)" src/orders/ | \
  awk -F'[\"\\\\\\047]' '{print $2}' | sort -u | wc -l
```

## Incorrect

```typescript
// ❌ OrderService reaches deep into Payment, Inventory, and Email internals
import { StripeClient } from '../payments/stripe/client';
import { StripeWebhookSecret } from '../payments/stripe/config';
import { InventoryDB } from '../inventory/db/connection';
import { SesTransport } from '../email/transports/ses';

export class OrderService {
  async place(order: Order) {
    const stripe = new StripeClient(StripeWebhookSecret.value);
    const charge = await stripe.charges.create({ amount: order.total });

    const conn = await InventoryDB.connect();
    await conn.query('UPDATE inventory SET stock = stock - ? WHERE sku = ?', [order.qty, order.sku]);

    const ses = new SesTransport(process.env.AWS_KEY!);
    await ses.send({ to: order.email, body: '...' });
  }
}
```

**Problems:**
- Cannot test without a real Stripe key, DB, and SES credentials
- Switching from Stripe → Adyen means rewriting `place()`
- A schema change in Inventory breaks orders

## Correct

```typescript
// ✅ Depend on interfaces, inject implementations
export class OrderService {
  constructor(
    private payments: PaymentGateway,
    private inventory: InventoryRepository,
    private notifications: NotificationSender,
  ) {}

  async place(order: Order) {
    await this.payments.charge(order.total, order.token);
    await this.inventory.reserve(order.sku, order.qty);
    await this.notifications.orderConfirmed(order);
  }
}
```

**Benefits:**
- Each dependency is a stable interface — internals can change freely
- Tests use in-memory fakes; no credentials needed
- Stripe → Adyen swap is a one-line wiring change

## Remediation Strategy

- **Effort:** M per module boundary
- **When to pay down:** When two modules' release schedules need to decouple, or when a swap is on the roadmap.

Reference: [Robert Martin — Stable Dependencies Principle (Package Principles)](https://en.wikipedia.org/wiki/Package_principles)

---


## Circular Dependencies

**Impact: HIGH (Indicates broken module boundaries; breaks tree-shaking and isolation)**

Module A imports B, B imports A. Either the boundary is wrong, or one module is misplaced. Cycles break dead-code elimination, make modules impossible to test in isolation, and cause runtime initialization order bugs in many languages.

## How to Detect

```bash
# JavaScript / TypeScript
npx madge --circular --extensions ts,tsx src/

# PHP (architectural rules, including cycle detection between layers/modules)
vendor/bin/deptrac analyse                  # qossmic/deptrac
# or:  vendor/bin/phparkitect check
```

Threshold: **zero cycles** is the only acceptable target. Even one cycle indicates a layering problem.

## Incorrect

```typescript
// ❌ user/index.ts imports order/, order/index.ts imports user/
// src/user/index.ts
import { Order } from '../order';
export class User {
  orders: Order[] = [];
  totalSpent() { return this.orders.reduce((s, o) => s + o.total, 0); }
}

// src/order/index.ts
import { User } from '../user';
export class Order {
  customer: User;
  total: number;
}
```

**Problems:**
- Either module fails to initialize cleanly under some bundlers (one side is `undefined` at import time)
- Cannot extract `user` or `order` into a separate package
- A test of `user` necessarily pulls in `order`

## Correct

```typescript
// ✅ Option A: extract shared types to a third module
// src/shared/types.ts
export interface UserRef { id: string; }
export interface OrderRef { id: string; total: number; }

// src/user/index.ts → imports types only
// src/order/index.ts → imports types only

// ✅ Option B: invert the dependency — let one own the relationship
// src/order/index.ts owns the customer reference;
// user no longer knows about order. totalSpent() lives in an OrderService.
```

**Benefits:**
- No initialization-order bugs
- Each module can be packaged independently
- Tests load the minimum surface

## Remediation Strategy

- **Effort:** M (mechanical once you decide the direction)
- **When to pay down:** As soon as `madge`/equivalent reports a new cycle. Adding tests across the boundary first protects the refactor.

Reference: [Madge — Circular Dependencies](https://github.com/pahen/madge)

---


## Leaky Abstractions

**Impact: HIGH (Implementation details leak past layer boundaries, blocking change)**

An abstraction leaks when its consumers depend on details it was supposed to hide — ORM models in controllers, framework types in domain logic, HTTP concerns in repositories. Once leaked, the abstraction can no longer be changed independently.

## How to Detect

- Grep for ORM/framework types in inner layers (`Eloquent\Model`, `Request`, `HttpClient`) where they should not appear
- Look for return types like `Builder`, `Collection<Model>`, or `Response` crossing service boundaries

```bash
# Laravel: Eloquent leaking out of repositories
grep -rEn 'extends Model|Eloquent\\Builder' app/Services/ app/Domain/

# Express: Request/Response leaking into services
grep -rEn '\\bRequest\\b|\\bResponse\\b' src/services/
```

## Incorrect

```php
// ❌ Repository returns an Eloquent Builder — controller chains ORM calls
final class OrderRepository
{
    public function forCustomer(int $customerId): Builder
    {
        return Order::query()->where('customer_id', $customerId);
    }
}

// Controller does ORM chaining directly:
$orders = $this->repo->forCustomer($id)
    ->where('status', 'paid')
    ->with(['items', 'shipments'])
    ->orderByDesc('created_at')
    ->paginate(20);
```

**Problems:**
- Repository's promised abstraction ("get orders for customer") is gone — controllers issue arbitrary queries
- Cannot swap Eloquent for another data source without rewriting every controller
- N+1 risk now lives in controllers, not in one auditable place

## Correct

```php
// ✅ Repository returns plain DTOs or a paginated value object — no Builder leak
final class OrderRepository
{
    /** @return Paginated<OrderSummary> */
    public function paidForCustomer(int $customerId, int $page = 1): Paginated
    {
        $query = Order::query()
            ->where('customer_id', $customerId)
            ->where('status', 'paid')
            ->with(['items', 'shipments'])
            ->orderByDesc('created_at');

        return Paginated::from($query->paginate(20, page: $page), OrderSummary::class);
    }
}
```

**Benefits:**
- Controllers receive a stable type; database choice is hidden
- Eager-loading and ordering are owned by the repository (one auditable place)
- Repository can be replaced with an HTTP gateway, gRPC client, or in-memory fake

## Remediation Strategy

- **Effort:** M–L (each leak is local but there are usually many)
- **When to pay down:** When a swap or split is on the roadmap, OR when you find yourself fixing N+1 bugs in multiple controllers — the leak is now causing concrete pain.

Reference: [Joel Spolsky — The Law of Leaky Abstractions](https://www.joelonsoftware.com/2002/11/11/the-law-of-leaky-abstractions/)

---


## Shotgun Surgery

**Impact: HIGH (One conceptual change forces edits across many files)**

A change is "shotgun surgery" when adding one concept (a new payment method, a new locale, a new currency) requires edits in 5+ files. The information is fragmented; whoever makes the change is guaranteed to miss a spot.

## How to Detect

```bash
# Find files commonly changed together (co-change hotspots)
git log --since='6 months ago' --name-only --pretty=format:'COMMIT' | \
  awk '/COMMIT/{print "---"; next} {print}' | \
  # process: count file pairs appearing in the same commit
  # any pair > 10 co-changes is a shotgun-surgery suspect

# Search for switch/if chains on the same enum across multiple files
grep -rEn "case PaymentMethod::|=== ['\"]stripe['\"]" --include='*.{php,ts}'
```

## Incorrect

```typescript
// ❌ Adding a new payment method "klarna" requires editing 6 files

// src/payments/types.ts
type PaymentMethod = 'stripe' | 'paypal' | 'wire';

// src/payments/validator.ts
if (m === 'stripe') { /* */ } else if (m === 'paypal') { /* */ } else if (m === 'wire') { /* */ }

// src/payments/feeCalculator.ts
const FEES = { stripe: 0.029, paypal: 0.034, wire: 0 };

// src/payments/iconUrl.ts
const ICONS = { stripe: '...', paypal: '...', wire: '...' };

// src/payments/displayName.ts
const NAMES = { stripe: 'Card', paypal: 'PayPal', wire: 'Bank wire' };

// src/payments/router.ts
switch (m) { case 'stripe': return new StripeClient(); /* ... */ }
```

**Problems:**
- Adding Klarna means hunting through 6+ files — easy to miss one
- New engineers cannot find "what makes a payment method valid"
- Tests don't catch a forgotten file until production

## Correct

```typescript
// ✅ One Strategy per method — adding Klarna is one new file
// src/payments/methods/StripeMethod.ts
export const StripeMethod: PaymentMethod = {
  id: 'stripe',
  displayName: 'Card',
  iconUrl: '/icons/stripe.svg',
  feeRate: 0.029,
  validate: (payload) => /* ... */,
  charge: (amount, token) => new StripeClient().charge(amount, token),
};

// src/payments/methods/index.ts
export const METHODS = [StripeMethod, PayPalMethod, WireMethod /*, KlarnaMethod */];
```

**Benefits:**
- Adding Klarna = one new file + one export — nothing else changes
- All knowledge about a method lives in one place
- TypeScript types prevent "forgot a branch" bugs

## Remediation Strategy

- **Effort:** M (collect scattered knowledge into one type per concept)
- **When to pay down:** Before the *next* new variant is added — the pain is highest right when adding one.

Reference: [Refactoring Guru — Shotgun Surgery](https://refactoring.guru/smells/shotgun-surgery)

---


## Outdated Dependency Versions

**Impact: HIGH (Each major version skipped exponentially raises upgrade cost)**

Skipping major versions doesn't save effort — it just defers and compounds it. Two majors behind is roughly 4× the upgrade work of one major behind, because deprecations from intermediate versions stack.

## How to Detect

```bash
# Node / TypeScript
npm outdated                       # shows current vs wanted vs latest
npx npm-check-updates              # interactive upgrade tool

# PHP / Laravel
composer outdated --direct         # direct deps only
composer outdated --direct --major-only
```

Threshold: any dependency **more than 2 major versions behind** OR **more than 18 months behind on minor/patch**.

## Incorrect

```json
// ❌ package.json — multiple dependencies 3+ majors behind
{
  "dependencies": {
    "react": "^16.8.0",           // current major: 19
    "express": "^4.17.0",         // current major: 5
    "webpack": "^4.46.0",         // current major: 5
    "jest": "^26.6.3",            // current major: 29
    "@types/node": "^14.0.0"      // current: 22
  }
}
```

**Problems:**
- React 16 → 19 means a full upgrade path through legacy mode, automatic batching, new JSX transform, etc.
- Webpack 4 → 5 requires polyfill changes, ESM handling, persistent caching adoption
- Jest 26 → 29 changes test environment and ESM behaviour
- Each upgrade *separately* is now too big to fit in one sprint

## Correct

```json
// ✅ Upgraded incrementally; pin policies documented
{
  "dependencies": {
    "react": "^19.0.0",
    "express": "^5.0.0",
    "webpack": "^5.95.0",
    "jest": "^29.7.0",
    "@types/node": "^22.0.0"
  }
}
```

Establish a **monthly upgrade rhythm** rather than letting deps drift for a year.

**Benefits:**
- Each upgrade fits in a small PR
- Security patches and deprecation warnings land while context is fresh
- Avoids the "we can't upgrade React because of 5 transitive blockers" situation

## Remediation Strategy

- **Effort:** S per minor upgrade, M–L per major upgrade
- **When to pay down:**
  - **Patch/minor:** weekly or biweekly automated PRs (Renovate, Dependabot)
  - **Major:** scheduled, one dep at a time, with a release plan
- **Order of operations:** upgrade dev tools (TypeScript, ESLint, Jest) before frameworks; framework before app code

Reference: [Renovate Bot](https://docs.renovatebot.com/) · [Dependabot](https://docs.github.com/en/code-security/dependabot)

---


## Abandoned and Unmaintained Packages

**Impact: HIGH (No upstream fixes for bugs, CVEs, or runtime upgrades)**

A package without a release in 24+ months, with open critical issues, or with the maintainer publicly stepping away is **abandonware**. The next CVE, the next Node/PHP version, or the next breaking dep change becomes *your* problem to fix.

## How to Detect

Indicators to check on each direct dependency:

- **Last release date** (`npm view <pkg> time.modified`, `composer info <pkg>`)
- **Open issues vs closed** ratio (high open count, no recent triage)
- **Maintainer activity** (last commit > 2 years ago)
- **Explicit deprecation** (`npm view <pkg> deprecated`)
- **Known alternatives community has migrated to**

```bash
# Node
npm view <pkg> time.modified deprecated
npx npm-check                       # flags deprecated packages
npx snyk test                       # warns on unmaintained packages

# PHP
composer info <pkg>                 # shows abandoned status from packagist
composer audit                      # also reports abandonment
```

## Incorrect

```json
// ❌ Depending on packages flagged as abandoned or deprecated
{
  "dependencies": {
    "request": "^2.88.0",            // deprecated 2020 by maintainer
    "node-uuid": "^1.4.8",           // replaced by `uuid` years ago
    "moment": "^2.29.0",             // maintenance-only since 2020, deprecated by author
    "babel-eslint": "^10.1.0"        // replaced by @babel/eslint-parser
  }
}
```

**Problems:**
- `request` has an open CVE with no upstream fix coming
- `moment` ships ~290KB of timezone data — `date-fns` or `dayjs` do it in 10KB
- Future engineer cannot tell whether these are "trusted core deps" or graveyard residents

## Correct

```json
// ✅ Migrated to maintained alternatives
{
  "dependencies": {
    "undici": "^6.0.0",              // replaces `request`
    "uuid": "^9.0.0",                // replaces `node-uuid`
    "date-fns": "^3.0.0",            // replaces `moment`
    "@babel/eslint-parser": "^7.23.0"
  }
}
```

**Benefits:**
- CVEs in maintained packages get upstream fixes — you only patch
- Bundle size and runtime characteristics improve
- New engineers don't waste time on packages they "shouldn't have learned"

## Remediation Strategy

- **Effort:** S–M per package (depends on API surface used)
- **When to pay down:**
  1. **Now:** any abandoned dep with a known CVE
  2. **This quarter:** any abandoned dep blocking a runtime upgrade
  3. **Opportunistically:** the rest, when you're already touching that code path

**Tip:** When forced to keep an abandoned dep temporarily, lock the version exactly, document why in a comment in the manifest, and create a tracking issue.

Reference: [npm Deprecation Policy](https://docs.npmjs.com/policies/deprecation) · [Packagist Abandoned Packages](https://packagist.org/about#abandoning-a-package)

---


## Known Security Advisories

**Impact: CRITICAL (Public CVEs are pre-published attack instructions)**

Once a CVE is public, exploit attempts start within hours. A HIGH/CRITICAL advisory in your dependency tree is not "tech debt to schedule" — it's an unmitigated security incident you haven't responded to yet.

## How to Detect

```bash
# Node
npm audit                                # full report
npm audit --audit-level=high             # CI-friendly threshold
npm audit fix                            # auto-fix non-breaking

# PHP
composer audit                           # built-in since Composer 2.4
composer audit --format=json

# Cross-stack
snyk test            # https://snyk.io
```

## Incorrect

```bash
# ❌ Pre-commit and CI ignore audit results
$ npm audit
12 vulnerabilities (3 moderate, 7 high, 2 critical)
$ git push   # CI passes — audit isn't a gate
```

**Problems:**
- CRITICAL CVEs sitting in main are public attack surface
- No paper trail of when each was acknowledged
- Each new dep adds more without anyone noticing

## Correct

```yaml
# ✅ CI gate that fails on high+ vulnerabilities
# .github/workflows/security.yml
- name: Audit dependencies
  run: |
    npm audit --audit-level=high
    composer audit --abandoned=fail

# ✅ Renovate / Dependabot configured for security PRs
# renovate.json
{
  "vulnerabilityAlerts": { "enabled": true, "labels": ["security"] },
  "osvVulnerabilityAlerts": true
}
```

**Benefits:**
- New CVEs auto-generate PRs within hours of disclosure
- CI fails the moment a high-severity advisory lands
- Audit log of every advisory acknowledgement and fix

## Remediation Strategy

- **Effort:**
  - **S** — patch/minor bump available, no breaking change
  - **M** — requires upgrade across multiple deps
  - **L** — vulnerable code is in an abandoned dep; replacement needed
- **When to pay down:**
  - **CRITICAL / HIGH:** within 24–72 hours
  - **MEDIUM:** within the current sprint
  - **LOW:** opportunistically with other dep work
- If a fix is genuinely blocked, document the **compensating control** (WAF rule, input validation, feature disable) and the **target unblocking date**.

Reference: [GitHub Advisory Database](https://github.com/advisories) · [OSV.dev](https://osv.dev/)

---


## Unused Dependencies

**Impact: MEDIUM (Inflate install size, supply-chain surface, and audit noise)**

A dependency you don't use is one you still ship, audit, and trust. Each unused dep is a potential supply-chain footgun (compromised maintainer, malicious post-install script) for zero benefit.

## How to Detect

```bash
# Node / TypeScript
npx depcheck                         # unused + missing deps
npx knip                             # also finds unused files and exports

# PHP / Composer
composer-unused                      # https://github.com/composer-unused/composer-unused
vendor/bin/composer-unused
```

## Incorrect

```json
// ❌ package.json declares deps no longer imported
{
  "dependencies": {
    "lodash": "^4.17.21",        // grep shows zero `from 'lodash'` imports
    "axios": "^1.6.0",           // migrated to fetch 6 months ago
    "moment": "^2.29.4",         // migrated to date-fns; one stale import left
    "node-fetch": "^3.3.0"       // only used in a deleted script
  }
}
```

**Problems:**
- Each `npm install` downloads code that does nothing
- Each `npm audit` reports advisories you can't act on (you don't even use the affected code paths)
- New engineers see them and assume they're load-bearing

## Correct

```bash
# ✅ Remove unused deps
$ npx depcheck
Unused dependencies: lodash, axios, moment, node-fetch
$ npm uninstall lodash axios moment node-fetch
$ npm audit            # quieter report
```

```yaml
# Add to CI to keep it clean
- run: npx depcheck --ignores="@types/*,eslint-*"
```

**Benefits:**
- Smaller `node_modules`, faster installs, faster CI
- Audit reports are signal, not noise
- Reduced supply-chain attack surface

## Remediation Strategy

- **Effort:** S (almost always)
- **When to pay down:** Immediately on detection. Add a depcheck/composer-unused step to CI to prevent regression.

**Watch out for:**
- **Transitive usage only:** some deps are loaded by tooling (e.g., babel plugins listed in `babel.config.js`). Verify before removing.
- **Type-only packages:** `@types/*` packages are used by the compiler but invisible to import scanners — configure your tool to ignore them.

Reference: [depcheck](https://github.com/depcheck/depcheck) · [composer-unused](https://github.com/composer-unused/composer-unused)

---


## Coverage Gaps on Critical Paths

**Impact: HIGH (Uncovered critical paths fail in production, not in CI)**

Aggregate coverage percentage is a vanity metric. What matters is whether the **critical paths** (checkout, auth, payment, signup) have integration tests. A repo at 85% line coverage with zero tests on payment has more risk than one at 60% with a full checkout suite.

## How to Detect

Identify critical paths from the product (signup, login, checkout, refund, etc.), then check:

```bash
# Node
npx jest --coverage --coverageReporters=text
npx vitest run --coverage

# PHP
vendor/bin/phpunit --coverage-html=coverage/

# Targeted: are there any integration tests for the checkout flow?
grep -rln 'test.*checkout\|describe.*checkout' tests/
```

For each critical path, look for:
- An end-to-end / integration test that exercises the happy path
- Tests for failure modes (declined card, out-of-stock, expired session)
- At least one test that runs against the real DB / real network adapter

## Incorrect

```
// ❌ 92% line coverage — but all of it is on getters, mappers, and trivial helpers.
// The actual checkout pipeline has NO integration test.

src/
├── utils/             — 100% covered (10 tests)
├── formatters/        — 100% covered (15 tests)
├── checkout/
│   ├── pricing.ts     —  20% covered
│   ├── inventory.ts   —   0% covered
│   ├── payment.ts     —   0% covered
│   └── orchestrate.ts —   0% covered   ← THE checkout flow
```

**Problems:**
- Coverage metric is "green" → team feels safe → bugs land in checkout
- No safety net for refactoring the orchestrator
- On-call has no automated regression check before deploys

## Correct

```typescript
// ✅ One integration test per critical-path scenario, hitting real adapters where feasible
// tests/checkout.integration.test.ts
describe('checkout', () => {
  it('places an order with valid card', async () => {
    const order = await checkoutClient.place({ /* ... */ });
    expect(order.status).toBe('confirmed');
    expect(inventory.stockFor('SKU-1')).toBe(initial - 1);
  });

  it('rejects when card is declined', async () => {
    await expect(checkoutClient.place({ token: 'tok_chargeDeclined' }))
      .rejects.toThrow(PaymentDeclined);
    expect(inventory.stockFor('SKU-1')).toBe(initial);  // no leak
  });

  it('rejects when item is out of stock', async () => { /* ... */ });
  it('issues idempotent retries safely', async () => { /* ... */ });
});
```

**Benefits:**
- Regression on checkout fails CI, not customers
- Refactoring the orchestrator is safe
- New scenarios (e.g., new payment method) extend a known suite

## Remediation Strategy

- **Effort:** M per critical path (the first test costs the most; subsequent are cheap)
- **When to pay down:** **Before** the next behaviour change on that path. The change itself is your justification.
- **Target:** one integration test per critical path, covering happy + 2–3 failure modes. Don't chase a coverage number.

Reference: [Martin Fowler — Test Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html)

---


## Flaky Tests

**Impact: HIGH (Erode trust in CI; teach the team to ignore failures)**

A flaky test — one that passes and fails on the same code — is worse than no test. The first few times, you re-run. After that, the team learns to retry-and-merge, and the suite stops catching real regressions.

## How to Detect

```bash
# Run the suite N times against the same commit
for i in {1..20}; do npm test -- --silent || echo "Failed run $i"; done

# Better: track flakiness across CI runs
# - GitHub Actions: re-run-on-failure stats
# - CircleCI / Buildkite: built-in flaky-test reports
# - https://github.com/jonny-improbable/jest-circus-flaky-retry

# Identify suspect tests by name patterns
grep -rEn 'sleep|setTimeout|Date\.now|Math\.random' tests/
```

Common flaky-test smells:
- `sleep(N)` / `setTimeout` instead of waiting for a condition
- Order-dependent tests (depend on previous test's state)
- Time-dependent assertions (`expect(date).toBe(today)`)
- Tests against shared mutable resources (real Redis without cleanup, real DB without transactions)

## Incorrect

```typescript
// ❌ Three different forms of flakiness
test('debounced search', async () => {
  searchInput.value = 'foo';
  await sleep(300);                          // race: timing-dependent
  expect(results).toHaveLength(5);
});

test('order created today', () => {
  const order = createOrder();
  expect(order.createdAt.toDateString())
    .toBe(new Date().toDateString());        // fails when run at midnight
});

test('user can log in', async () => {
  await db.query('INSERT INTO users ...');  // depends on previous test's cleanup
  // ...
});
```

**Problems:**
- Sleep races: works on fast machines, fails on busy CI runners
- Date-dependent: fails on DST changes, midnight, leap-day
- Shared-state: passes alone, fails in suite

## Correct

```typescript
// ✅ Deterministic alternatives
test('debounced search', async () => {
  searchInput.value = 'foo';
  await waitFor(() => expect(results).toHaveLength(5));  // wait on condition
});

test('order created at the expected time', () => {
  vi.setSystemTime(new Date('2026-01-15T10:00:00Z'));    // freeze time
  const order = createOrder();
  expect(order.createdAt).toEqual(new Date('2026-01-15T10:00:00Z'));
});

beforeEach(async () => {
  await db.transaction(async (t) => { /* setup, rolled back after each test */ });
});
```

**Benefits:**
- Test passes deterministically regardless of machine speed, clock, or order
- Suite can run in parallel without contention
- CI failures become signal again

## Remediation Strategy

- **Effort:** S per test (the fix is usually local — replace sleep with waitFor, freeze the clock, use transactions)
- **When to pay down:**
  1. **Quarantine** the flaky test immediately (mark as `.skip` with a tracking issue) so it stops eroding trust
  2. **Fix** within the sprint — quarantine is a deferral, not a destination
  3. **Delete** if it can't be made deterministic in reasonable effort

**Policy:** Re-running a failed CI without a root-cause is anti-pattern. Always file an issue.

Reference: [Google Testing Blog — Flaky Tests](https://testing.googleblog.com/2016/05/flaky-tests-at-google-and-how-we.html)

---


## Skipped and Disabled Tests

**Impact: HIGH (Dark coverage — code looks tested, isn't)**

Skipped tests are worse than missing tests because they create a false sense of safety. A `test.skip(...)` or `markTestSkipped()` left in main without an owner, issue, or deadline is debt that grows in silence.

## How to Detect

```bash
# JavaScript / TypeScript (Jest / Vitest)
grep -rEn '\\.skip|xdescribe|xit|test\\.todo|describe\\.skip' tests/ src/

# Pest / PHPUnit
grep -rEn 'markTestSkipped|markTestIncomplete|@group\\s+skip|->skip\\(' tests/
```

Cross-reference each hit with:
- Is there a linked issue?
- Is there a comment explaining why?
- Is there a date or condition for re-enabling?

## Incorrect

```typescript
// ❌ Bare skips with no context
describe.skip('checkout', () => { /* ... */ });

test.skip('refunds work', () => { /* ... */ });

test('payment webhook', () => {
  if (process.env.CI) return;          // silent skip on CI
  // ...
});
```

**Problems:**
- Why are checkouts skipped? Nobody remembers
- The webhook test runs only locally — production behaviour is untested
- Coverage report shows them as "executed" but with zero assertions

## Correct

```typescript
// ✅ Every skip has an owner, reason, and re-enable trigger
test.skip(
  'refunds work — DISABLED 2026-02-10 (#1247) re-enable after Stripe webhook v2 migration',
  () => { /* ... */ }
);

// ✅ Or: delete and replace if the test cannot be repaired
//   git log will remember it ever existed.
```

Add CI checks:

```yaml
# .github/workflows/test-hygiene.yml
- name: Disallow new bare skips
  run: |
    NEW_SKIPS=$(git diff origin/main...HEAD -- 'tests/**' \
      | grep -E '^\+.*\.skip\(' | grep -v '#[0-9]')
    test -z "$NEW_SKIPS" || { echo "Bare skip without ticket"; exit 1; }
```

**Benefits:**
- Every skip is auditable and assigned
- New skips require a ticket — prevents quiet accumulation
- The team has a count of "real" coverage

## Remediation Strategy

- **Effort:** S per skip (decide: fix, delete, or document)
- **When to pay down:**
  - **Now:** audit existing skips → add owner + ticket OR delete
  - **Ongoing:** CI gate prevents new bare skips
- **Default action:** if a skip is older than 90 days with no movement, delete the test. If it's worth keeping, it's worth re-enabling.

Reference: [PHPUnit docs](https://docs.phpunit.de/) (see the "Incomplete and Skipped Tests" chapter in the current major version)

---


## Slow Test Suites

**Impact: HIGH (Slow feedback compounds across every engineer every day)**

A test suite that takes 20 minutes costs every engineer 20 minutes per push. Multiplied across team size and PR count, it becomes the single biggest tax on velocity — and pushes the team to stop running tests locally.

## How to Detect

```bash
# Show slowest tests
npx jest --verbose                                    # per-test timing in output
npx jest --reporters=jest-slow-test-reporter          # 3rd-party reporter (recommended)
npx vitest --reporter=verbose | sort -k4 -rn | head -20

# PHPUnit
vendor/bin/phpunit --log-junit=junit.xml
# Then sort junit.xml by time

# Track total wall-clock time per CI run
# - Watch for trends: "is it growing 10% per quarter?"
```

Targets to aim for:
- **Unit suite:** < 30 seconds
- **Integration suite:** < 5 minutes
- **Total CI per PR:** < 10 minutes

## Incorrect

```typescript
// ❌ Common slow-test smells

// 1. Real network calls in unit tests
test('user can fetch profile', async () => {
  const data = await fetch('https://api.production.example.com/users/1');
});

// 2. Real sleeps for "wait for something"
test('debounced search', async () => {
  search('foo');
  await sleep(2000);          // 2s × 50 tests = 100s wasted
});

// 3. Full DB rebuild per test instead of transaction rollback
beforeEach(async () => {
  await execSync('npm run db:migrate:fresh');   // 10s × 200 tests
});
```

## Correct

```typescript
// ✅ Fast alternatives
// 1. Mock the network at the boundary for unit tests (MSW v2)
import { http, HttpResponse } from 'msw';
import { setupServer } from 'msw/node';
const server = setupServer(
  http.get('/users/1', () => HttpResponse.json(USER)),
);

// 2. Wait on conditions, not on time
await waitFor(() => expect(results).toHaveLength(5));   // returns in ms

// 3. Use transactions or schema snapshots
beforeEach(async () => {
  await db.beginTransaction();
});
afterEach(async () => {
  await db.rollback();
});
```

Parallelize where safe:

```bash
# Jest: --maxWorkers=50%
# Vitest: vitest --pool=threads        (Vitest v1+; `--threads` is deprecated)
# PHPUnit: vendor/bin/paratest -p 8
```

**Benefits:**
- Engineers run tests locally → faster feedback before push
- CI cost drops linearly with wall time
- Slow-by-design tests (E2E) can be quarantined to a nightly job

## Remediation Strategy

- **Effort:** S–M per hotspot (mostly mechanical: mock, fake, paratest, transactionalize)
- **When to pay down:**
  1. **First win:** identify and fix the 5 slowest tests — usually 50%+ of total time
  2. **Then:** enable parallel execution
  3. **Then:** budget. Document a per-suite wall-clock budget and fail CI if exceeded.

**Budget enforcement:**

```yaml
# Hard cap test duration in CI
- run: timeout 600 npm test    # fails CI if > 10 minutes
```

Reference: [Martin Fowler — Test Suite Speed](https://martinfowler.com/articles/practical-test-pyramid.html#TheImportanceOf(Test)Speed)

---


## N+1 Query Patterns

**Impact: HIGH (Linear request → quadratic database load; latency scales with data, not traffic)**

An N+1 happens when fetching a list of N records issues 1 query for the list + N follow-up queries for each row's relations. Latency looks fine in dev (small N) and explodes in prod (large N). N+1 is the single most common database debt in ORM-driven codebases.

## How to Detect

```bash
# Laravel: Telescope panel "Queries" — sort by request, count per page
php artisan telescope:install

# Laravel: detect N+1s in dev by failing on excessive queries
composer require beyondcode/laravel-query-detector --dev

# Node / TypeScript: enable query logging in your ORM (Prisma `log: ['query']`,
# TypeORM `logging: 'all'`, Drizzle `logger: true`) and count queries per request.

# Heuristic: > ~10 queries for a typical list endpoint is suspect.
```

Pattern signal: query count grows with result count instead of staying constant.

## Incorrect

```php
// ❌ Laravel: relation accessed inside a loop → one query per order
$orders = Order::where('status', 'paid')->get();        // 1 query

foreach ($orders as $order) {
    $customer = $order->customer;                       // 1 query each (N more)
    $shipping = $order->shipments;                      // 1 query each (another N)
    echo "{$customer->name}: " . $shipping->count();
}
// Total: 1 + 2N queries for 100 orders → 201 queries
```

```typescript
// ❌ TypeORM equivalent
const users = await userRepo.find();                    // 1 query
for (const user of users) {
  const orders = await user.orders;                     // N queries (lazy relation)
  console.log(user.email, orders.length);
}
```

**Problems:**
- A list endpoint that returns 200 rows takes 401 round-trips to the database
- Latency is invisible at low traffic; suddenly catastrophic with real data
- Database connection pool saturates; queue depth spikes for unrelated requests

## Correct

```php
// ✅ Eager-load with `with()` — fixed query count regardless of N
$orders = Order::where('status', 'paid')
    ->with(['customer', 'shipments'])
    ->get();   // 3 queries total: orders, customers, shipments

foreach ($orders as $order) {
    echo "{$order->customer->name}: " . $order->shipments->count();
}
```

```typescript
// ✅ Specify relations in the find call
const users = await userRepo.find({ relations: { orders: true } });
```

For large result sets, also paginate:

```php
$orders = Order::with(['customer', 'shipments'])
    ->where('status', 'paid')
    ->cursorPaginate(50);   // memory + DB bounded
```

**Benefits:**
- Query count becomes constant per request, regardless of result size
- Latency is predictable in production
- Connection pool stays healthy under load

## Remediation Strategy

- **Effort:** S per endpoint (add `with(...)` or equivalent eager-load)
- **When to pay down:**
  - **NOW:** any endpoint that emits > 10× more queries than its result count
  - **Then:** add a CI test that asserts query counts on critical endpoints
- **Detection workflow:**
  1. Install Telescope / Bullet / Silk
  2. Walk the top 10 most-hit endpoints in a representative env
  3. Sort by query count per request — N+1s are at the top
  4. Add `with(...)` and re-measure

**Anti-pattern:** "let's add a cache" before fixing N+1. Caching hides the problem but doesn't fix it — the first request after expiry still does 201 queries.

Reference: [Laravel Eager Loading](https://laravel.com/docs/eloquent-relationships#eager-loading) · [beyondcode/laravel-query-detector](https://github.com/beyondcode/laravel-query-detector)

---


## Unbounded Result Sets

**Impact: HIGH (One large customer kills latency for everyone)**

An endpoint that returns "all" records works fine until one customer has 100,000 of them. At that point, the request times out, exhausts memory, and (in shared-tenancy systems) takes down the database for everyone else. Unbounded result sets are time bombs proportional to your most successful customer.

## How to Detect

```bash
# Find list endpoints / repository methods without LIMIT or pagination
grep -rEn '\\->all\\(|\\->get\\(\\)|findAll|fetchAll' --include='*.php' --include='*.ts'

# Frontend: full-result-set components without "load more" / virtualization
grep -rEn 'map\\(|forEach\\(' --include='*.tsx' src/ | head -50  # review for unbounded lists

# Check actual prod metrics: max rows returned by each endpoint over the last 30 days
# (most APMs / DB monitors expose this)
```

Heuristic: any list endpoint **without an explicit limit** is debt. Any endpoint that returns objects with nested collections (orders → items, users → posts) is doubly so.

## Incorrect

```php
// ❌ Returns every order ever placed by the customer
public function index(Customer $customer) {
    return OrderResource::collection($customer->orders);   // 50,000 rows → 8MB response
}

// ❌ "Export all users" endpoint loads entire table into memory
public function export() {
    $users = User::all();                                  // OOM at scale
    return Excel::download(new UsersExport($users), 'users.xlsx');
}
```

```typescript
// ❌ Frontend asks for everything and filters client-side
const allTransactions = await fetch('/api/transactions').then(r => r.json());
const recent = allTransactions.filter(t => isThisMonth(t.date));   // 100,000 rows → 200,000 ms parse
```

## Correct

```php
// ✅ Cursor-based pagination (preferred for "infinite scroll" / large datasets)
public function index(Customer $customer, Request $request) {
    return OrderResource::collection(
        $customer->orders()->latest()->cursorPaginate(50)
    );
}

// ✅ Streaming export — never loads the full set into memory
public function export() {
    return response()->streamDownload(function () {
        User::query()->orderBy('id')->lazy()->each(function ($user) {
            // write one row at a time
        });
    }, 'users.csv');
}
```

```typescript
// ✅ Server filters; client requests only what it needs
const { data, nextCursor } = await fetch('/api/transactions?since=2026-05-01&limit=50')
  .then(r => r.json());
```

**Benefits:**
- Memory and latency stay bounded regardless of customer size
- Database returns fewer rows over the wire
- Frontend can render results progressively

## Remediation Strategy

- **Effort:** S–M per endpoint (cursor pagination is more invasive than offset; both are mechanical)
- **When to pay down:**
  - **NOW:** any endpoint where the result count is user-controlled and unbounded
  - **NOW:** any export endpoint loading the entire result set into memory
  - **Then:** add a max-result-count assertion in CI for list endpoints
- **Pagination choice:**
  - **Cursor** — best for "next page" UX, large datasets, real-time-ish data (no skip cost)
  - **Offset** — easier to implement, OK for small/medium datasets; expensive at high page numbers
  - **Keyset** — similar to cursor but using a real column (id, created_at)

**Tip:** when retrofitting pagination on a public API, support both old (full response) and new (paginated) shapes during a deprecation window, then remove the unbounded form.

Reference: [Laravel Pagination](https://laravel.com/docs/pagination) · [Slack — Cursor Pagination](https://docs.slack.dev/apis/web-api/pagination) · [Use the Index, Luke — Paging Through Results](https://use-the-index-luke.com/sql/partial-results/fetch-next-page)

---


## Frontend Bundle Bloat

**Impact: HIGH (Bundle size directly drives bounce rate, INP, and conversion)**

Every kilobyte the browser must parse and execute slows down first paint, interaction-readiness, and on mobile networks, page abandonment. Bundle bloat creeps in invisibly — a moment-import here, a `lodash` import there, an unused route bundled with the entry point — and the team only notices when Lighthouse drops a grade.

## How to Detect

```bash
# Vite
npx vite-bundle-visualizer

# Webpack
npx webpack-bundle-analyzer dist/stats.json

# Any bundler — source map visualization
npx source-map-explorer 'dist/**/*.js'

# CI bundle-size budget (size-limit)
npm install --save-dev size-limit @size-limit/preset-app
# package.json:
#   "size-limit": [{ "path": "dist/index.js", "limit": "200 KB" }]
npx size-limit
```

Budget targets (gzipped, mobile-first):
- **Initial bundle:** < 200 KB
- **Route bundles:** < 100 KB
- **Single dep:** anything > 50 KB deserves justification

## Incorrect

```typescript
// ❌ Default import → ships the entire library
import _ from 'lodash';                     // ~70 KB min / ~24 KB gzip
import moment from 'moment';                // ~290 KB min / ~70 KB gzip with locales

const debounced = _.debounce(handler, 200);
const formatted = moment().format('YYYY-MM-DD');

// ❌ No code splitting — admin pages bundled with the public site
import AdminDashboard from './admin/Dashboard';
import AdminUsers from './admin/Users';
// ... all eagerly imported in the entry file
```

**Problems:**
- Importing all of lodash to use `debounce` is like buying a truck to carry one tomato
- Moment with all locales ships ~70 KB gzip of timezone data nobody uses
- Admin code bundled with the public site triples the entry-bundle for 99% of visitors who never visit `/admin`

## Correct

```typescript
// ✅ Named imports / smaller libraries
import { debounce } from 'lodash-es';        // tree-shaken via ESM
import { format } from 'date-fns';           // ~10 KB gzip for the single `format` import

const debounced = debounce(handler, 200);
const formatted = format(new Date(), 'yyyy-MM-dd');

// ✅ Route-level code splitting
const AdminDashboard = lazy(() => import('./admin/Dashboard'));
const AdminUsers     = lazy(() => import('./admin/Users'));

// In your router:
<Route path="/admin" element={<Suspense fallback={<Spinner />}><AdminDashboard /></Suspense>} />
```

Add a CI guard:

```yaml
- name: Bundle-size budget
  run: npx size-limit
# Fails the build if any tracked bundle exceeds its budget
```

**Benefits:**
- Initial bundle shrinks dramatically; LCP and INP both improve
- Admin code only loads for admin users
- A regression (someone adds `import * as everything`) fails CI

## Remediation Strategy

- **Effort:** S–M per dep (swap imports, lazy-load routes)
- **When to pay down:**
  - **First:** run the bundle visualizer and target the top 5 contributors
  - **Then:** install a bundle-size budget in CI to prevent regression
  - **Ongoing:** every PR that adds a dep > 20KB should be reviewed for alternatives
- **Common wins:**
  - `moment` → `date-fns` or `dayjs` (–80–90% size)
  - `lodash` → `lodash-es` + named imports, or native equivalents
  - Route-level code splitting (10× reduction on admin-heavy apps)
  - Drop polyfills for unsupported browsers
  - Replace heavy SVG icon sets with on-demand icon components

Reference: [web.dev — Apply Instant Loading](https://web.dev/articles/apply-instant-loading-with-prpl) · [BundlePhobia](https://bundlephobia.com/) · [size-limit](https://github.com/ai/size-limit)

---


## Missing Caching Opportunities

**Impact: HIGH (Repeated work on every request — predictable latency you're paying for indefinitely)**

Caching is the highest-ROI performance work for read-heavy systems: a single Redis lookup replaces a 200ms aggregation query, and an HTTP cache header lets the browser skip the round-trip entirely. Missing caching is invisible — the system "works" — but every page load pays the full computation cost.

## How to Detect

Look for these signals in any read-heavy path:

1. **Expensive aggregations recomputed per request** (dashboard counters, reports, leaderboards)
2. **External API calls without caching** (currency rates, geolocation, third-party catalogs)
3. **Static-ish responses with no `Cache-Control` headers** (asset metadata, config, public lists)
4. **Database queries that join 5+ tables to return the same shape repeatedly**
5. **Same query fired by N concurrent requests, no request coalescing**

```bash
# Look at HTTP response headers — missing Cache-Control on static-ish endpoints
curl -sI https://your.app/api/categories | grep -iE 'cache-control|etag|last-modified'

# Find APIs that re-compute the same shape across requests
# (look in your APM for endpoints with consistently high mean latency + low variance)

# Laravel — endpoints that hit expensive accessors / relations without remember()
grep -rEn '\\->withCount\\(|\\->withSum\\(' app/Http/Controllers/ | head
```

## Incorrect

```php
// ❌ Recomputed on every request — even though categories change ~once a week
public function categories() {
    $categories = Category::query()
        ->withCount('products')
        ->with(['parent', 'translations'])
        ->orderBy('sort_order')
        ->get();

    return CategoryResource::collection($categories);
}

// ❌ Third-party rate fetched per request — 200ms latency on every checkout
public function checkout(Order $order) {
    $rate = Http::get('https://api.fx.example.com/rates/USD-MYR')->json('rate');
    return ['total' => $order->total * $rate];
}

// ❌ No HTTP caching → browser revalidates on every navigation
return response()->json($publicConfig);
```

**Problems:**
- Database scans for `categories` happen N times per second across the whole fleet
- Every checkout pays 200ms for an FX rate that updates hourly
- Browser fetches `publicConfig` on every page load even though it changes daily

## Correct

```php
// ✅ Tag-keyed Redis cache with explicit invalidation
// Note: Cache::tags() only works with the `redis` or `memcached` driver.
// On `file` / `database` / `array` stores it throws BadMethodCallException —
// fall back to plain keys + manual invalidation on those drivers.
public function categories() {
    $categories = Cache::tags(['categories'])->remember(
        'categories:tree:v2',
        now()->addHour(),
        fn () => Category::query()
            ->withCount('products')
            ->with(['parent', 'translations'])
            ->orderBy('sort_order')
            ->get(),
    );
    return CategoryResource::collection($categories);
}

// In Category::saved / Category::deleted listeners:
//   Cache::tags(['categories'])->flush();
```

```php
// ✅ External API result cached for an hour
$rate = Cache::remember('fx:USD-MYR', now()->addHour(), function () {
    return Http::get('https://api.fx.example.com/rates/USD-MYR')->json('rate');
});
```

```php
// ✅ HTTP cache headers for safely-cacheable public responses
return response()->json($publicConfig)
    ->header('Cache-Control', 'public, max-age=3600, stale-while-revalidate=86400')
    ->header('ETag', md5(json_encode($publicConfig)));
```

**Benefits:**
- Mean latency drops orders of magnitude for cacheable endpoints
- Origin server, database, and third-party APIs all see reduced load
- Browser short-circuits revalidation for unchanged resources

## Remediation Strategy

- **Effort:**
  - **S** — add `remember()` / HTTP `Cache-Control` to a single endpoint
  - **M** — design a cache-key scheme + invalidation strategy for a domain
  - **L** — distributed caching with proper invalidation across services
- **When to pay down:**
  - **NOW:** any endpoint hitting an expensive query AND showing high traffic AND data is read-mostly
  - **Then:** look at top-10 requests by total time in APM; cache the easy wins first
- **Layer order (cheapest first):**
  1. HTTP cache headers (browser does the work)
  2. CDN / edge cache (`Cache-Control: public, s-maxage=N`)
  3. Application memory cache (per-process, fastest, no network)
  4. Redis / Memcached (shared across processes, sub-ms)
  5. Database query cache / materialized views (last resort)

**Anti-patterns:**
- **Caching everything by default** — cache invalidation is hard; cache only what hurts
- **TTL-only invalidation when freshness matters** — combine with event-based busts (`Cache::flush` on writes)
- **Caching personalized data with a public key** — leaks one user's data to another
- **Caching error responses indefinitely** — always exclude 4xx/5xx from caches

Reference: [Laravel — Cache](https://laravel.com/docs/cache) · [MDN — HTTP Caching](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching) · [RFC 9111 — HTTP Caching](https://www.rfc-editor.org/rfc/rfc9111)

---


## Database Schema Drift

**Impact: HIGH (Production schema diverges from code; migrations break unpredictably)**

Schema drift is the gap between what the migrations say the database looks like and what it actually looks like. Once it exists, every new migration is a roll of the dice — it might apply cleanly, fail halfway through, or succeed but leave the schema in a state neither environment expects.

## How to Detect

```bash
# Laravel: compare current schema to migration plan
php artisan migrate:status
php artisan schema:dump --prune          # snapshot current schema
# Apply on a fresh DB and diff against production schema

# MySQL: schema-only dump for diffing across environments
mysqldump --no-data --routines --triggers -u root -p prod_db    > prod.sql
mysqldump --no-data --routines --triggers -u root -p staging_db > staging.sql
diff -u staging.sql prod.sql

# Drift signals:
# - Tables that exist in prod but not in any migration
# - Columns/indexes added manually via ALTER TABLE outside migration
# - Migrations marked "ran" in different orders across environments
# - Migrations that have been edited in place (hashes differ)
```

Tooling: `atlas migrate diff`, `bytebase`, `liquibase diff`, `flyway info` for managed migration platforms.

## Incorrect

```
❌ Common drift scenarios:

1. "Quick fix" applied directly to prod
   DBA runs:  ALTER TABLE orders ADD COLUMN priority INT DEFAULT 0;
   …without a corresponding migration. New migrations now run against a schema
   the codebase has never seen.

2. Migration edited after being applied somewhere
   Original: CREATE TABLE refunds (id, order_id, amount);
   Edited:   CREATE TABLE refunds (id, order_id, amount, reason TEXT NOT NULL);
   Some environments have the column; others don't. Same migration hash, two realities.

3. Models with attributes not in any migration
   class Order extends Model {
     protected $fillable = ['status', 'priority'];  // 'priority' has no migration
   }
```

**Problems:**
- New environments (CI, staging, new dev laptops) can't reach the same schema
- The next migration may fail at 50% completion, leaving the schema half-done
- Reports and queries silently rely on columns that "are there in prod" but not in code

## Correct

```php
// ✅ Every schema change goes through a migration
// One migration per change, never edit a merged migration

// database/migrations/2026_05_16_000000_add_priority_to_orders.php
return new class extends Migration {
    public function up(): void {
        Schema::table('orders', fn (Blueprint $t) =>
            $t->unsignedTinyInteger('priority')->default(0)->after('status')
        );
    }
    public function down(): void {
        Schema::table('orders', fn (Blueprint $t) => $t->dropColumn('priority'));
    }
};
```

CI gate:

```yaml
- name: Schema is migration-derivable
  run: |
    php artisan migrate --pretend --database=ci_clone   # ensure all migrations apply
    php artisan schema:dump --prune
    git diff --exit-code database/schema/               # fail if dump differs
```

**Benefits:**
- Any environment can be reconstructed from migrations alone
- Code and schema move together, atomically reviewable in PRs
- A failed migration in CI catches drift before it hits prod

## Remediation Strategy

- **Effort:** M–L (depends on how far drift has progressed)
- **When to pay down:**
  - **NOW:** any drift discovered during incident response — fix during the postmortem
  - **As a cleanup project:** snapshot prod schema → generate a "consolidation migration" that brings empty databases to current state → mark all prior migrations as "ran" in environments that already match
- **Anti-patterns:**
  - Editing applied migrations (always create a new one)
  - "DBA runs prod ALTERs directly" without a corresponding migration
  - Squashing migrations in a way that breaks existing environments

**Tip:** in long-lived projects, periodically generate a "consolidated migration" from the current schema (`schema:dump`) so new environments don't have to replay 5 years of migrations. Keep the consolidated dump and historical migrations both checked in.

Reference: [Laravel — Schema Dumping](https://laravel.com/docs/migrations#squashing-migrations) · [Atlas — Migration Diff](https://atlasgo.io/) · [Liquibase Diff](https://docs.liquibase.com/commands/inspection/diff.html)

---


## Missing Database Indexes

**Impact: HIGH (Query time grows linearly with data; locks and connection pool compound)**

A missing index turns a 5ms query into a 5-second query as the table grows. Worse, because slow queries hold connections longer, missing indexes cascade into connection-pool exhaustion and 503s for unrelated traffic. Indexing decisions made early are usually right; indexes never added at all are silent debt.

## How to Detect

```sql
-- MySQL (requires sys schema, enabled by default in 5.7+): tables doing frequent full scans
SELECT * FROM sys.schema_tables_with_full_table_scans
WHERE rows_full_scanned > 1000
ORDER BY rows_full_scanned DESC;

-- MySQL: query plan for a hot query (type=ALL means full table scan)
EXPLAIN     SELECT * FROM orders WHERE customer_id = 123;
EXPLAIN ANALYZE  SELECT * FROM orders WHERE customer_id = 123;   -- MySQL 8.0+

-- MySQL: indexes that exist but are never read
SELECT * FROM sys.schema_unused_indexes;

-- MySQL: top queries by total latency (performance_schema must be ON)
SELECT * FROM sys.statement_analysis
ORDER BY total_latency DESC LIMIT 20;
```

Application-side:
- **Laravel:** Telescope's "Queries" panel — sort by duration
- **Node ORMs:** enable query logging (Prisma `log: ['query', 'warn']`, TypeORM `logging: 'all'`) and review slow entries

Look for: queries on foreign keys, status columns, date filters, and `ORDER BY` columns without supporting indexes.

## Incorrect

```php
// ❌ Foreign key columns without indexes
Schema::create('orders', function (Blueprint $t) {
    $t->id();
    $t->unsignedBigInteger('customer_id');     // FK column — no index!
    $t->string('status');                       // queried often — no index!
    $t->timestamp('created_at');
});

// Query: SELECT * FROM orders WHERE customer_id = ? AND status = 'paid' ORDER BY created_at DESC
//   → Seq Scan of the entire orders table for every customer profile load
```

**Problems:**
- Customer-profile page becomes O(N) of total orders, not the customer's orders
- Status-based dashboards lock the table during long scans
- Connection pool saturates under modest load

## Correct

```php
// ✅ Index foreign keys, status columns, and ORDER BY columns
Schema::create('orders', function (Blueprint $t) {
    $t->id();
    $t->foreignId('customer_id')->constrained()->index();    // index on FK
    $t->string('status')->index();
    $t->timestamp('created_at');

    // Composite index for the common (customer_id, status, created_at) query path
    $t->index(['customer_id', 'status', 'created_at']);
});
```

Verify the plan after indexing:

```sql
EXPLAIN
SELECT * FROM orders
WHERE customer_id = 123 AND status = 'paid'
ORDER BY created_at DESC LIMIT 50;

-- Want to see in `key`: orders_customer_id_status_created_at_index
-- NOT:                  NULL  (or `type` = ALL → full table scan)
```

**Benefits:**
- Query time becomes O(log N) instead of O(N)
- Connection pool stays healthy under load
- The index pays for itself many times over per request

## Remediation Strategy

- **Effort:**
  - **S** — add a single index (online index creation in MySQL/InnoDB minimizes downtime)
  - **M** — add several indexes; analyze query patterns first
  - **L** — large tables (100M+ rows) require careful online build + monitoring
- **When to pay down:**
  - **NOW:** any query on a hot path showing `type=ALL` in EXPLAIN over a > 10k-row table
  - **NOW:** any FK column without an index (defaults vary by ORM — Eloquent does not auto-index FKs)
  - **Then:** monitor `sys.statement_analysis` weekly and add indexes for the top long-runners

**Anti-patterns:**
- **Indexing everything** — wastes write speed and disk; index the columns you actually filter/sort by
- **Adding indexes blindly** without `EXPLAIN ANALYZE` — verify the planner actually uses them
- **Forgetting to remove old indexes** — duplicate or unused indexes cost disk + write speed

**Online index creation (MySQL/InnoDB):**
```sql
ALTER TABLE orders
  ADD INDEX orders_customer_status_created_at_index (customer_id, status, created_at),
  ALGORITHM=INPLACE, LOCK=NONE;
```

For huge tables, prefer `pt-online-schema-change` (Percona Toolkit) or `gh-ost` (GitHub) — both run truly non-blocking schema changes.

Reference: [Use the Index, Luke](https://use-the-index-luke.com/) · [MySQL — EXPLAIN Output Format](https://dev.mysql.com/doc/refman/8.4/en/explain-output.html) · [MySQL — sys Schema](https://dev.mysql.com/doc/refman/8.4/en/sys-schema.html) · [pt-online-schema-change](https://docs.percona.com/percona-toolkit/pt-online-schema-change.html)

---


## Orphaned Records and Referential Drift

**Impact: MEDIUM (Bugs, broken reports, and inconsistent state compound over time)**

Orphaned records — child rows whose parent no longer exists — are usually invisible until a report breaks or a query joins fail mysteriously. The root cause is almost always missing foreign-key constraints, ad-hoc deletes that bypass the ORM, or "soft-delete the parent, leave children active" semantics.

## How to Detect

```sql
-- Orphans on a single relation
SELECT child.id, child.parent_id
FROM order_items child
LEFT JOIN orders parent ON parent.id = child.parent_id
WHERE parent.id IS NULL;

-- MySQL: columns ending in _id that are NOT part of any FK constraint.
-- Note: `_` is a single-char wildcard in LIKE — escape it (use `#` as ESCAPE char to
-- avoid backslash-escape confusion in MySQL string literals). Exclude the PK `id` column.
SELECT c.TABLE_NAME, c.COLUMN_NAME
FROM information_schema.COLUMNS c
WHERE c.TABLE_SCHEMA = DATABASE()
  AND c.COLUMN_NAME LIKE '%#_id' ESCAPE '#'
  AND c.COLUMN_NAME <> 'id'
  AND NOT EXISTS (
    SELECT 1
    FROM information_schema.KEY_COLUMN_USAGE kcu
    JOIN information_schema.TABLE_CONSTRAINTS tc
      ON tc.CONSTRAINT_NAME   = kcu.CONSTRAINT_NAME
     AND tc.CONSTRAINT_SCHEMA = kcu.CONSTRAINT_SCHEMA
     AND tc.TABLE_NAME        = kcu.TABLE_NAME
    WHERE tc.CONSTRAINT_TYPE   = 'FOREIGN KEY'
      AND kcu.TABLE_SCHEMA     = c.TABLE_SCHEMA
      AND kcu.TABLE_NAME       = c.TABLE_NAME
      AND kcu.COLUMN_NAME      = c.COLUMN_NAME
  );

-- Soft-deleted parents with active children
SELECT COUNT(*) FROM orders
WHERE deleted_at IS NOT NULL
  AND EXISTS (
    SELECT 1 FROM order_items
    WHERE order_id = orders.id AND deleted_at IS NULL
  );
```

## Incorrect

```php
// ❌ No FK constraints; manual delete bypasses cascades
Schema::create('order_items', function (Blueprint $t) {
    $t->id();
    $t->unsignedBigInteger('order_id');          // no constraint, no index
    $t->unsignedBigInteger('product_id');
});

// Deletion via raw SQL — children become orphans
DB::table('orders')->where('status', 'cancelled')->delete();
// 200 order rows gone; their 1,800 order_items now point to nothing.

// Soft-delete parent, hard-active children — reports show ghost orders
class Order extends Model { use SoftDeletes; }
class OrderItem extends Model {}   // does NOT respect parent's deleted_at
```

**Problems:**
- Joins return rows but with `NULL` parents, silently breaking sums and counts
- Cleanup jobs run forever ("delete order_items with no order" finds millions)
- Restore-from-backup workflows can't trust the data they restore

## Correct

```php
// ✅ FK constraints with explicit cascade behaviour
Schema::create('order_items', function (Blueprint $t) {
    $t->id();
    $t->foreignId('order_id')
        ->constrained()                     // creates FK to orders(id)
        ->cascadeOnDelete();                // delete items when order is deleted
    $t->foreignId('product_id')
        ->constrained()
        ->restrictOnDelete();               // can't delete a product still referenced
});

// ✅ Soft-deletes coordinated across parent and children
//    Both models must use SoftDeletes for soft-cascade to work (otherwise
//    `$order->items()->delete()` will hard-delete the children).
class OrderItem extends Model {
    use SoftDeletes;
}

class Order extends Model {
    use SoftDeletes;
    protected static function booted() {
        static::deleted(fn($order) => $order->items()->delete());      // soft-deletes children
        static::restored(fn($order) => $order->items()->withTrashed()->restore());
    }
}
```

```sql
-- ✅ Add missing FKs to legacy tables (MySQL/InnoDB; clean orphans first)
-- Note: adding a FK with `ALGORITHM=INPLACE` is only supported when
-- `foreign_key_checks=OFF`, and `LOCK=NONE` is NOT supported for FK adds.
-- With default settings MySQL forces `ALGORITHM=COPY`, which rewrites
-- the table. For large tables, use `pt-online-schema-change` or `gh-ost`
-- to add the FK without blocking writes.
ALTER TABLE order_items
  ADD CONSTRAINT fk_order_items_order
  FOREIGN KEY (order_id) REFERENCES orders(id) ON DELETE CASCADE;
```

**Benefits:**
- Database enforces integrity even when application code is buggy
- Reports and joins are trustworthy
- Cleanup jobs become unnecessary (or trivial)

## Remediation Strategy

- **Effort:**
  - **S** — adding constraints to greenfield tables
  - **M** — backfilling constraints + cleanup on small/medium tables
  - **L** — adding constraints to a 100M+ row table (must clean orphans first, then add constraint online)
- **When to pay down:**
  - **NOW:** any data integrity issue traced back to orphans
  - **As cleanup project:** audit tables for `_id` columns without FK constraints
  - **Then:** make FKs required for all new tables (lint a migration template)

**Cleanup workflow:**
1. Run the orphan-detection query above for each suspected relation
2. Decide policy: hard-delete orphans, mark them archived, or attach to a placeholder parent
3. Apply the cleanup in batches (avoid one giant transaction)
4. Add the FK constraint
5. Add a CI test that runs the orphan-detection query against the test DB after each test run

Reference: [MySQL — FOREIGN KEY Constraints](https://dev.mysql.com/doc/refman/8.4/en/create-table-foreign-keys.html) · [Laravel — Foreign Key Constraints](https://laravel.com/docs/migrations#foreign-key-constraints)

---


## Stale Comments

**Impact: MEDIUM (Wrong information is worse than no information)**

A comment that contradicts the code it describes actively misleads readers. They either trust the comment and write a bug, or they distrust all comments and miss the load-bearing ones. Stale comments are negative-value documentation.

## How to Detect

There is no perfect tool — stale comments are read-and-judge. Useful starting points:

```bash
# Find comments referencing functions, classes, files that no longer exist
grep -rEn '@see |@deprecated |TODO\\(|see also' src/ | \
  while IFS= read -r line; do
    REF=$(echo "$line" | grep -oE '[A-Z][a-zA-Z]+::[a-zA-Z]+|[a-z_]+\\.[a-z]+')
    [ -n "$REF" ] && ! grep -rq "$REF" src/ && echo "STALE: $line"
  done

# Find comments referencing removed parameters
# (compare comment params with current function signature)
```

Hotspot: comments next to code that has been git-touched more recently than the comment.

## Incorrect

```typescript
// ❌ Comment lies about behaviour
/**
 * Returns the user's full name in "Last, First" format.
 */
function displayName(user: User): string {
  return `${user.firstName} ${user.lastName}`;   // actually "First Last"
}

// ❌ Comment references removed parameter
/**
 * @param userId - the user to load
 * @param includeArchived - whether to include archived records
 */
function loadUser(userId: string) {              // includeArchived removed last year
  return db.users.findOne({ id: userId, archived: false });
}

// ❌ "Temporary" workaround that became permanent
// HACK: workaround for Stripe API bug — remove after their 2022 fix
const tax = Math.round(subtotal * 0.06 * 100) / 100;
```

**Problems:**
- A reader follows the comment, writes integration code expecting "Last, First", ships a bug
- IDE autocomplete picks up the stale `@param`, suggesting a parameter that no longer exists
- "Temporary" workaround now load-bearing; nobody dares remove it

## Correct

```typescript
// ✅ Comment matches reality, or is deleted
/**
 * Returns "First Last" — used in user-facing greetings.
 */
function displayName(user: User): string {
  return `${user.firstName} ${user.lastName}`;
}

// ✅ Parameter doc removed when parameter removed
function loadUser(userId: string) {
  return db.users.findOne({ id: userId, archived: false });
}

// ✅ Either remove the workaround, or update the comment with current rationale
// Stripe's 2022 fix shipped; this rounding is kept because our DB stores 4 decimal places
// and accounting reports require 2-decimal-place reconciliation. (#TAX-431)
const tax = Math.round(subtotal * 0.06 * 100) / 100;
```

**Benefits:**
- Comments become trusted again
- IDE hints align with reality
- "Why does this exist" is captured at the right level of fidelity

## Remediation Strategy

- **Effort:** S per comment
- **When to pay down:** On every PR — if you touch code, read the surrounding comments and verify or delete. Reviewers should call out stale comments next to changed lines.
- **Heuristic:** when in doubt, delete. Code documents *what*; the commit message documents *why*. A stale comment is rarely the right tool to keep.

Reference: [John Ousterhout — A Philosophy of Software Design, Ch. 13](https://web.stanford.edu/~ouster/cgi-bin/aposd.php)

---


## Outdated Architecture Documentation

**Impact: MEDIUM (Onboarding and incident response collapse without accurate maps)**

Architecture docs that don't match the running system cause the worst kind of mistake: confident wrong decisions. The cost shows up at the worst moments — onboarding a new engineer or debugging a 2am incident.

## How to Detect

For each architecture doc in the repo:
- **README "Getting Started":** can a brand-new engineer follow it end-to-end *today* and end with a working dev environment?
- **Architecture diagrams:** do all named services still exist with the same names?
- **Database schema diagrams:** do all tables and columns exist?
- **Sequence diagrams:** do the request paths still match the code?

```bash
# Find architecture docs and check last-modified vs the code they describe
find . \( -name 'ARCHITECTURE.md' -o -name 'README.md' -o -path '*/docs/*' \) | \
  while read doc; do
    DOC_DATE=$(git log -1 --format=%ct -- "$doc")
    SRC_DATE=$(git log -1 --format=%ct -- src/ app/)
    if [ $((SRC_DATE - DOC_DATE)) -gt 7776000 ]; then       # 90 days
      echo "STALE: $doc (last touched $(date -r $DOC_DATE +%Y-%m-%d))"
    fi
  done
```

## Incorrect

```markdown
<!-- ❌ README from 18 months ago -->
# CheckoutService

## Architecture
- Node.js 14 (LTS)
- MongoDB for orders
- Redis for sessions
- Stripe for payments

## Getting Started
1. Clone the repo
2. Copy `.env.example` to `.env` and fill in the DB credentials
3. `composer install && npm install`
4. `php artisan migrate && php artisan serve`

## Services
The service is split into:
- /api    — REST endpoints
- /jobs   — background workers
- /web    — admin dashboard
```

Reality:
- Migrated to Node 22 last year
- Moved from MongoDB to MySQL 8 months ago
- Stripe was replaced with Adyen
- `.env.example` no longer exists; uses Doppler
- `/web` was extracted to its own repo 6 months ago

**Problems:**
- New engineer spends 2 days debugging "why won't MongoDB connect"
- Incident responder pages the wrong on-call (the old `/web` team)
- Architectural decisions get made against an imaginary system

## Correct

```markdown
<!-- ✅ README that matches reality, with a freshness signal -->
# CheckoutService

_Last verified against running system: 2026-05-10_

## Architecture
- Node.js 22 (LTS)
- MySQL 8.4 for orders
- Redis for sessions
- Adyen for payments (migrated from Stripe — see `docs/adr/0007-adyen.md`)

## Getting Started
1. `gh repo clone org/checkout && cd checkout`
2. `cp .env.example .env`; fill in DB and Adyen credentials (or pull via `doppler setup`)
3. `composer install && npm install`
4. `php artisan migrate --seed && php artisan serve` (in another shell: `npm run dev`)

## Services
- `/api`  — REST endpoints
- `/jobs` — background workers
- (Admin dashboard lives in `org/checkout-admin`)
```

**Benefits:**
- Onboarding works end-to-end without hidden tribal knowledge
- Incident responders trust the doc to find the right team
- Each ADR captures the *why* for architectural changes

## Remediation Strategy

- **Effort:** S–M per doc
- **When to pay down:**
  - Whenever a PR changes architecture (new service, new datastore, new dependency): **update the doc in the same PR**
  - Quarterly: "doc walk" — pair an engineer with a brand-new hire, follow the README, fix what breaks
- **Tip:** add a `Last verified` date to architecture docs; treat doc-aging like dep-aging

Reference: [Architecture Decision Records (ADR)](https://adr.github.io/) · [The Diátaxis Framework](https://diataxis.fr/)

---


## Undocumented Public APIs

**Impact: MEDIUM (Consumers reverse-engineer behaviour; breaking changes blindside everyone)**

A public API without documentation forces consumers to read source or guess. Once they guess wrong and ship, that "wrong guess" becomes the de facto contract — you can no longer change the implementation without breaking them.

## How to Detect

```bash
# TypeScript: public exports without TSDoc
# Requires eslint-plugin-jsdoc installed and registered in your eslint config; the
# rule below is a config-file rule (not a CLI one-liner override):
#   { "plugins": ["jsdoc"], "rules": { "jsdoc/require-jsdoc": ["error", {"publicOnly": true}] } }

# PHP: public methods on public classes without docblocks
# PHPStan does NOT natively flag missing docblocks — use PHP_CodeSniffer with a
# docblock-aware standard (e.g. Squiz.Commenting.FunctionComment), or phpDocumentor's
# validator. PHPStan can still catch missing parameter/return *types* via higher levels.
vendor/bin/phpcs --standard=Squiz --sniffs=Squiz.Commenting.FunctionComment app/

# OpenAPI / REST APIs
# Compare routes/controllers to swagger.json / openapi.yaml — drift is debt
```

## Incorrect

```typescript
// ❌ Exported function with non-obvious behaviour and no docs
export function calculateRefund(order: Order, items: Item[]): RefundResult {
  // ... 80 lines including special cases for partial refunds,
  // restocking fees, expired return windows, original-payment-method routing,
  // and tax recalculation under different jurisdictions
}
```

Questions a consumer can't answer without reading the body:
- Does `items` default to all items if empty? Or refund nothing?
- Are restocking fees deducted? How are they configured?
- What happens if the original payment method is invalidated?
- Are taxes recalculated, or refunded proportionally?

## Correct

```typescript
/**
 * Calculate a refund for some or all items in an order.
 *
 * - If `items` is empty, refunds the entire order.
 * - Restocking fees (configured per category) are deducted from the refund amount.
 * - Taxes are recalculated based on the remaining order subtotal (not refunded proportionally).
 * - Refunds route back to the original payment method; if invalidated, returns
 *   `RefundResult.requiresAlternativeMethod = true` and the caller must collect new instrument.
 * - Refunds are not allowed past the return window (`order.returnsCloseAt`).
 *
 * @throws RefundWindowClosed   — if `order.returnsCloseAt` has passed
 * @throws PartialRefundNotAllowed — if the order's policy disallows partial returns
 *
 * @see docs/refunds.md for business rules and worked examples.
 */
export function calculateRefund(order: Order, items: Item[]): RefundResult {
  // ...
}
```

**Benefits:**
- Consumers depend on documented contract, not observed behaviour
- You can change implementation freely as long as docs hold
- Edge cases are surfaced — usually catching a bug or missing test in the process

## Remediation Strategy

- **Effort:** S per public function (write the doc; if it's hard to write, the function is doing too much)
- **When to pay down:**
  - **Now:** add CI rule (`jsdoc/require-jsdoc`, PHPStan public-API check) so new APIs ship with docs
  - **Gradually:** document existing APIs as you touch them. Don't try to backfill all at once.
- **Side benefit:** writing the doc is the cheapest way to find an API with too many concerns. If you need 3 paragraphs to describe a single function, split it.

Reference: [TSDoc](https://tsdoc.org/) · [PHPDoc](https://docs.phpdoc.org/) · [OpenAPI Specification](https://swagger.io/specification/)

---


## End-of-Life Runtime Versions

**Impact: CRITICAL (EOL runtimes stop receiving security patches)**

Once a language runtime hits EOL, no more CVE fixes ship — the next disclosed vulnerability is unpatched and forever yours. EOL deadlines are public, hard, and immovable; treating them as "we'll deal with it later" is treating a deadline as optional.

## How to Detect

```bash
# Check installed versions
node --version
php --version

# Check declared versions
cat .nvmrc .node-version 2>/dev/null
grep -E '"engines"|"node":' package.json
grep -E '"php"' composer.json

# Server-side check (Forge / Vapor / shared hosting)
ssh user@server 'php -v && node -v'

# Check against EOL dates
# - https://endoflife.date/nodejs
# - https://endoflife.date/php
# - https://www.php.net/supported-versions.php
```

| Runtime | EOL Pattern |
|---|---|
| Node.js | Every 6 months; LTS for 30 months |
| PHP | 2 years active + 2 years security (since 2024) |

Threshold: **flag any version < 6 months from EOL** as P1, **EOL today** as P0.

## Incorrect

```json
// ❌ package.json declares EOL Node
{
  "engines": { "node": "14.x" }
}
```

```json
// ❌ composer.json requires EOL PHP
{
  "require": { "php": "^7.4" }     // PHP 7.4 EOL: 2022-11-28
}
```

```
// ❌ .nvmrc still pins to Node 14
14.21.3
```

**Problems:**
- Any disclosed CVE in Node 14 / PHP 7.4 stays exploitable indefinitely
- Modern dependencies start dropping support — package upgrades become impossible
- Forge / Vapor / shared-hosting providers eventually remove EOL versions entirely → forced emergency migration

## Correct

```json
// ✅ package.json — Node 22 LTS
{
  "engines": { "node": ">=22 <23" }
}
```

```json
// ✅ composer.json — PHP 8.3 (LTS, supported through 2027-12)
{
  "require": { "php": "^8.3" }
}
```

```
// ✅ .nvmrc
22.6.0
```

Automate the watch:

```yaml
# CI step
- name: Check runtime EOL
  run: |
    NODE_MAJOR=$(node --version | sed 's/v\([0-9]*\).*/\1/')
    EOL_NODE=20    # update annually
    test $NODE_MAJOR -ge $EOL_NODE || { echo "Node $NODE_MAJOR is EOL"; exit 1; }
```

**Benefits:**
- Security patches keep arriving for free
- Modern dep ecosystem stays compatible
- No emergency "runtime EOL was last week" migration

## Remediation Strategy

- **Effort:**
  - **S** — minor runtime bump within supported major
  - **M** — major bump (Node 18 → 22, PHP 8.1 → 8.3)
  - **L** — multiple majors at once (Node 14 → 22) — break into steps
- **When to pay down:**
  - **6 months before EOL:** start the upgrade
  - **3 months before EOL:** be done
  - Schedule a calendar reminder when you upgrade — the next deadline is already on the clock

Reference: [endoflife.date](https://endoflife.date/) · [Node Release Schedule](https://nodejs.org/en/about/previous-releases) · [PHP Supported Versions](https://www.php.net/supported-versions.php)

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

---


## Ignored Build and Lint Warnings

**Impact: MEDIUM (Warning noise hides real failures and trains the team to ignore output)**

A build that emits dozens of warnings teaches every engineer that warnings are normal. The day a critical warning appears (a deprecation, a type-narrowing issue, a circular import), nobody sees it. Clean output is a precondition for noticing problems.

## How to Detect

```bash
# Capture and count warnings from build/test/lint
npm run build 2>&1 | grep -ciE 'warning|deprecat'
npx tsc --noEmit 2>&1 | wc -l
npx eslint . 2>&1 | grep -c 'warning'

# PHP
vendor/bin/phpstan analyse --no-progress
vendor/bin/phpcs --report=summary

# Webpack / Vite
# Look at the bundler output for "compiled with N warnings"
```

Threshold: **zero warnings tolerated**. Either fix or explicitly suppress with a comment explaining why.

## Incorrect

```bash
# ❌ Build "passes" but emits a wall of warnings
$ npm run build
[tsc] src/orders/index.ts(42,5): warning TS6133: 'unused' is declared but never used.
[tsc] src/orders/index.ts(55,3): warning TS2532: Object is possibly undefined.
[eslint] src/payment/stripe.ts:18:1 warning  no-explicit-any
[eslint] src/payment/stripe.ts:34:5 warning  react-hooks/exhaustive-deps
[webpack] WARNING in ./node_modules/some-pkg/dist/index.js
       Critical dependency: the request of a dependency is an expression
... 87 more warnings
Compiled with 92 warnings.
```

**Problems:**
- A new genuine warning ("X will be removed in vNext") buries in the noise
- "Compiled successfully" with 92 warnings is a lie that erodes trust
- New engineers conclude "warnings don't matter here"

## Correct

```bash
$ npm run build
Compiled successfully (0 warnings).
```

CI gates:

```yaml
- run: npx tsc --noEmit                          # fails on any type error
- run: npx eslint . --max-warnings 0             # zero warnings
- run: npm run build -- --no-warnings            # bundler warnings → errors
```

When a warning genuinely must be suppressed:

```typescript
// ✅ Targeted suppression with reason
// eslint-disable-next-line @typescript-eslint/no-explicit-any -- Stripe SDK types are too narrow; tracked in #2034
function configureStripe(opts: any) { /* ... */ }
```

**Benefits:**
- Build output is signal — every line means something
- Reviews can ask "does this PR add any new warning?" → easy answer
- Genuine deprecations and CVE-related warnings are noticed immediately

## Remediation Strategy

- **Effort:** S–M (most warnings are mechanical fixes; a few require small refactors)
- **When to pay down:**
  1. **Snapshot the current count** in CI: `--max-warnings $CURRENT_COUNT`
  2. **Ratchet down** — every PR can only equal or decrease the count
  3. Reach zero, then flip to `--max-warnings 0`
- **Anti-pattern:** disabling the lint rule entirely instead of fixing the underlying issues. The rule exists for a reason; suppress with context, don't disable globally.

Reference: [ESLint — Disabling Rules](https://eslint.org/docs/latest/use/configure/rules#disabling-rules) · [TypeScript Strict Mode](https://www.typescriptlang.org/tsconfig#strict)

---


## Insecure Secrets Management

**Impact: HIGH (.env files leak, plain env vars surface in logs, no rotation = compounding risk)**

Even when secrets stay out of source control, ad-hoc handling — `.env` files copied between machines, plain env vars echoed into CI logs, indefinite credential lifetimes — leaves a long tail of leakage paths. This rule covers everything *outside* the repo: secret-manager adoption, rotation, scope, and audit.

## How to Detect

For each environment (dev, staging, prod), check:

1. Where are secrets stored? (encrypted secret manager vs. plain files vs. shell history)
2. Are they versioned and audit-logged?
3. Is rotation automated or manual?
4. Are CI/CD secrets scoped per repo/job, or org-wide?
5. Are workload identities (OIDC / IAM roles) used instead of long-lived keys?

```bash
# Audit any .env files committed historically
gitleaks git                                  # full history scan
git log --all --diff-filter=A --name-only | grep -E '\\.env$'

# CI: scan for echoes of secrets in logs (common when devs add `set -x`)
grep -E 'API_KEY=|TOKEN=|PASSWORD=' .github/workflows/*.yml

# Cloud: list long-lived access keys (AWS example)
aws iam list-access-keys --user-name <iam-user>   # any keys > 90 days old?
```

## Incorrect

```bash
# ❌ Anti-patterns

# 1. Plain .env file committed in history (even if removed later)
$ git log -p --all -- '.env'
... (compromised)

# 2. CI workflow echoing secrets via debugging
- run: echo "AWS_KEY=$AWS_ACCESS_KEY_ID"      # ends up in build logs

# 3. Single shared IAM access key used by all CI jobs, 4 years old, never rotated
$ aws iam list-access-keys --user-name ci-deploy
CreateDate: 2022-03-15   # 4 years; full admin permissions

# 4. Slack/Notion link sharing for "DB credentials" — anyone with the link sees the password
```

**Problems:**
- A leaked CI log exposes the entire production environment
- Long-lived credentials with broad scope = compromise = full account access
- No audit trail of who accessed what credential when

## Correct

```yaml
# ✅ GitHub Actions: OIDC to AWS — no long-lived keys needed
permissions:
  id-token: write
  contents: read
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: aws-actions/configure-aws-credentials@v6
        with:
          role-to-assume: arn:aws:iam::123456789012:role/deploy-role
          aws-region: ap-southeast-1
      # AWS calls now use short-lived session tokens scoped to this workflow
```

```bash
# ✅ Production secrets in a dedicated manager (one of these):
#   - HashiCorp Vault       (self-hosted, comprehensive)
#   - AWS Secrets Manager   (with rotation Lambdas)
#   - GCP Secret Manager
#   - Doppler / 1Password Secrets Automation (SaaS)
#
# Apps read at startup via SDK; rotation is automated.

# ✅ Local dev: .env files exist but are gitignored and contain dev-only credentials
echo '.env' >> .gitignore
```

**Benefits:**
- No long-lived credentials to leak; access is scoped to job + duration
- Audit logs record every secret access
- Rotation is automatic and predictable

## Remediation Strategy

- **Effort:**
  - **S** — gitignore .env, install pre-commit gitleaks hook
  - **M** — migrate one application from env-file → secret manager
  - **L** — full org migration to OIDC + secret manager + rotation
- **When to pay down:**
  - **NOW:** any committed .env, any 90-day-old long-lived key, any secrets in plain CI logs
  - **This quarter:** migrate CI/CD to OIDC where the cloud provider supports it (AWS, GCP, Azure all do)
  - **Then:** automated rotation for all production secrets, with audit alerts on access spikes

**Hierarchy of secret handling, from worst to best:**
1. Hardcoded in repo (CRITICAL — must rotate immediately)
2. Untracked `.env` file emailed/Slacked around (high leak risk)
3. Long-lived credentials in CI/CD secrets store
4. Short-lived credentials issued via OIDC workload identity (best)

Reference: [GitHub Actions OIDC](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments) · [AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/) · [HashiCorp Vault](https://developer.hashicorp.com/vault) · [Doppler](https://docs.doppler.com/)

---


## Observability and Monitoring Gaps

**Impact: HIGH (Debt you can't measure can't be paid down; incidents take hours longer to diagnose)**

A system without structured logs, metrics, traces, and alerts is a system that breaks silently. The "we'll add monitoring later" decision is a tax paid during every incident, every customer-reported bug, and every capacity-planning conversation. Observability debt has the unique property of being most expensive to fix *during* an incident.

## How to Detect

For each service in scope, audit:

1. **Structured logs** — JSON, with correlation IDs, request IDs, user IDs (when appropriate)
2. **Application metrics** — request rates, latency percentiles (p50/p95/p99), error rates, queue depths
3. **Tracing** — distributed traces with span IDs spanning service boundaries
4. **Alerts** — paging only on user-facing symptoms; non-paging for early signals
5. **SLOs** — explicit service-level objectives with error budgets

```bash
# Find log statements still using non-structured output
grep -rEn 'echo |print(|print_r(|var_dump(|console\.log\(' app/ src/ | wc -l

# Laravel: is logging configured to JSON channel?
grep -A5 "'channels' =>" config/logging.php

# Are there any Prometheus / OpenTelemetry imports/integrations?
grep -rEn 'prometheus|opentelemetry|datadog|sentry|new-relic' composer.json package.json

# Sentry / Bugsnag / equivalent error tracking installed?
grep -rE 'sentry|bugsnag|rollbar|honeybadger' composer.lock package-lock.json
```

## Incorrect

```php
// ❌ Observability black hole

// 1. Print-style logging — unstructured, no correlation
public function pay(Order $order) {
    echo "Processing payment for order " . $order->id . "\n";
    try {
        $result = $this->stripe->charge($order);
    } catch (Exception $e) {
        echo "FAILED: " . $e->getMessage() . "\n";          // lost on next request
    }
}

// 2. No request IDs
// 3. No latency metrics — "the app feels slow" is the only signal
// 4. Pager rule: "send a Slack message if a single 500 occurs"
//    → wakes people up for every transient hiccup; signal-to-noise = 0
// 5. No SLOs → no shared definition of "the service is broken"
```

**Problems:**
- During an incident, you can't tell which user, which request, or which span failed
- Capacity planning is guesswork ("we think we can handle 2× traffic")
- Slow regressions (p95 creeping from 200ms → 800ms over a quarter) go unnoticed
- On-call burnout from low-quality alerts

## Correct

```php
// ✅ Structured logging with correlation ID + context
public function pay(Order $order): PaymentResult {
    $log = Log::withContext([
        'order_id'    => $order->id,
        'user_id'     => $order->user_id,
        'request_id'  => request()->header('X-Request-ID') ?? Str::uuid(),
    ]);

    $log->info('payment.start', ['amount' => $order->total]);
    $start = microtime(true);

    try {
        $result = $this->stripe->charge($order);
        $log->info('payment.success', [
            'charge_id' => $result->id,
            'duration_ms' => (int) ((microtime(true) - $start) * 1000),
        ]);
        return $result;
    } catch (\Throwable $e) {
        $log->error('payment.failure', [
            'exception' => $e::class,
            'message'   => $e->getMessage(),
            'duration_ms' => (int) ((microtime(true) - $start) * 1000),
        ]);
        report($e);                                          // → Sentry / Bugsnag
        throw $e;
    }
}
```

Minimum viable observability stack to install:
- **Logs:** JSON channel + central log store (CloudWatch, Loki, Datadog Logs)
- **Errors:** Sentry / Bugsnag / Rollbar
- **Metrics + tracing:** OpenTelemetry SDK → vendor or self-hosted (Grafana stack, Datadog, Honeycomb)
- **Uptime:** external prober (UptimeRobot, Pingdom, Datadog Synthetics)
- **Alerts:** routed to a real pager system (PagerDuty, Opsgenie); thresholds based on SLO burn rate, not single events

Define SLOs explicitly (illustrative — real Sloth uses `version: prometheus/v1` plus an
`sli.events` block with `error_query`/`total_query`; Pyrra ships as a Kubernetes CRD —
consult each tool's schema before adopting):

```yaml
# slo.yaml — illustrative shape only
service: checkout
slos:
  - name: availability
    objective: 99.9%
    sli: error_rate < 1% over 28d
  - name: latency
    objective: 99% of requests < 500ms over 28d
```

**Benefits:**
- Incidents resolve faster (mean MTTR drops 50%+ with traces)
- Slow regressions are caught when they're small
- Alerts wake people up only for user-facing problems
- Engineering and product share a quantitative definition of "the service is healthy"

## Remediation Strategy

- **Effort:**
  - **S** — add Sentry + JSON logging to one service
  - **M** — add OpenTelemetry + APM dashboards
  - **L** — full SLO program (objectives, alerts on burn rate, error budgets, blameless postmortems)
- **When to pay down:**
  - **NOW:** any production service without an error-tracker (Sentry-class)
  - **NOW:** any service whose only outage signal is "a user complained"
  - **This quarter:** structured logging + request IDs + p95 latency dashboards
  - **Then:** SLO program; alerts based on burn rates, not raw thresholds

**Anti-patterns:**
- **Alert on everything** — alarms drown signal; only page on customer-impacting symptoms
- **Logs-only observability** — logs are expensive to query at scale; metrics + traces complement them
- **No retention policy** — logs at 1TB/day with infinite retention becomes its own debt
- **Tool sprawl** — one logs vendor, one APM, one error tracker is plenty

Reference: [Google SRE — Monitoring Distributed Systems](https://sre.google/sre-book/monitoring-distributed-systems/) · [OpenTelemetry](https://opentelemetry.io/) · [Sloth — SLO Generator](https://sloth.dev/)

---


## Aging TODO, FIXME, and HACK Comments

**Impact: MEDIUM (Untracked promises that compound silently)**

A `// TODO` is a promise to do something later, written by someone who has now forgotten. After 6 months, nobody remembers what the TODO meant, whether it still applies, or what the consequences are. These comments accumulate as untracked technical debt invisible to product and management.

## How to Detect

```bash
# All TODO/FIXME/HACK with the introducing commit's date (via git blame)
# Note: grep's --include uses fnmatch, not brace expansion — pass one --include per ext.
grep -rEn '(TODO|FIXME|HACK|XXX|BUG)' \
  --include='*.php' --include='*.ts' --include='*.tsx' \
  --include='*.js' --include='*.jsx' . | \
  while IFS=: read -r file line content; do
    DATE=$(git blame -L "$line,$line" --date=short -- "$file" 2>/dev/null | awk '{print $3}')
    echo "$DATE | $file:$line | $content"
  done | sort

# Quick count
grep -rEn '(TODO|FIXME|HACK)' src/ | wc -l
```

Threshold rules:
- **TODO older than 6 months** without ticket reference → debt
- **FIXME of any age** without ticket → debt (FIXME implies known broken)
- **HACK of any age** without ticket → debt (HACK implies known wrong)

## Incorrect

```typescript
// ❌ Naked TODOs / FIXMEs accumulated over years
function calculateShipping(order: Order): number {
  // TODO: support international shipping
  // TODO: discount for premium members
  // FIXME: this is wrong for orders > $500
  // HACK: hardcoded $15 for now

  return 15;
}
```

**Problems:**
- "Wrong for orders > $500" is a bug nobody is tracking
- International shipping was demanded a year ago; product team doesn't know it's blocked here
- Each TODO is an island — no estimate, no owner, no priority

## Correct

```typescript
// ✅ TODO with ticket, date, and owner (or no TODO at all)
function calculateShipping(order: Order): number {
  // Hardcoded $15 — international + premium discounts tracked in #1842 (asyraf, 2026-03)
  return 15;
}
```

Or — preferred when the work is real:

1. **Open a ticket** in the issue tracker
2. **Reference it** in code: `// see #1842`
3. **Delete bare TODOs** — if they're worth keeping, they're worth tracking

CI enforcement:

```yaml
- name: No new bare TODOs
  run: |
    BARE=$(git diff origin/main...HEAD \
      | grep -E '^\+.*\b(TODO|FIXME|HACK)\b' \
      | grep -v -E '#[0-9]+')
    test -z "$BARE" || { echo "Add a ticket reference to TODOs"; exit 1; }
```

**Benefits:**
- Every promise has an owner and a tracker entry
- Product can see and prioritise the backlog
- Code is honest about what's known-broken

## Remediation Strategy

- **Effort:** S per comment (decision: ticket, fix, or delete)
- **When to pay down:**
  - **Now:** triage existing TODOs → ticket, fix immediately, or delete
  - **Ongoing:** CI gate prevents new bare TODOs
- **Triage flow:**
  1. Still relevant? If no, delete.
  2. Worth doing? If no, delete.
  3. Worth doing this quarter? File a ticket, reference it.
  4. Worth doing now? Just do it — don't write a TODO.

Reference: [Steve McConnell — Code Complete, Ch. 32 on Self-Documenting Code](https://www.microsoftpressstore.com/store/code-complete-9780735619678)

---


## @deprecated Markers Without Removal Plan

**Impact: MEDIUM (Indefinite deprecations become permanent — they never actually go away)**

A `@deprecated` tag without a replacement, removal date, or migration plan is just a polite "I wish you wouldn't use this." Consumers keep using it, the deprecation message persists for years, and the codebase carries dead-but-not-dead APIs forever.

## How to Detect

```bash
# Find all @deprecated markers (one --include per extension; grep doesn't expand braces)
grep -rEn '@deprecated' \
  --include='*.php' --include='*.ts' --include='*.tsx' \
  --include='*.js' --include='*.jsx' src/ app/

# For each, check whether it includes:
#   - Replacement / `@see` pointer
#   - Removal version or date
#   - Reason

# Find @deprecated callers — these are migration targets
grep -rEn '@deprecated' -A2 src/ | grep -oE 'function [a-zA-Z]+|method [a-zA-Z]+'
```

Flag any `@deprecated` without:
1. **What to use instead** (`@see` or "use X instead")
2. **When it will be removed** (version, date, or trigger)
3. **At least one PR removing internal usage** since the deprecation was added

## Incorrect

```typescript
// ❌ Vague, unactionable deprecations
/** @deprecated */
export function getUserById(id: string) { /* ... */ }

/** @deprecated do not use */
export function legacyFormat(date: Date) { /* ... */ }

/** @deprecated use the new API */               // which "new API"?
export function fetchOrders() { /* ... */ }
```

**Problems:**
- Consumers see "deprecated" but cannot act
- No deadline → no urgency → no migration
- 5 years later still in code, still warning, still used everywhere

## Correct

```typescript
/**
 * @deprecated Since v4.2 (2025-09). Use {@link findUserById} which supports
 * batching and returns `Result<User, NotFound>`. Will be removed in v5.0
 * (target: 2026-Q3). Internal callers: 0 (migration complete).
 *
 * @see findUserById
 */
export function getUserById(id: string): User | null { /* ... */ }
```

Add CI to enforce:

```yaml
- name: No undocumented @deprecated
  run: |
    BAD=$(grep -rEn '@deprecated\b' src/ | grep -v -E '@deprecated.*[0-9]{4}')
    test -z "$BAD" || { echo "@deprecated needs a date/version"; exit 1; }
```

**Benefits:**
- Consumers see exactly what to do and when
- Tracking the deprecation → removal cycle is mechanical (grep + count)
- Old code actually leaves the codebase

## Remediation Strategy

- **Effort:** S per marker (audit), M per deprecation cycle (migrate internal callers, then remove)
- **When to pay down:**
  - **Audit existing markers:** add date + replacement, OR upgrade to "will be removed in next major" + start migration
  - **Removal:** when a deprecation hits its target version, *actually remove the code*. A deprecation that doesn't end is a lie.
- **Cycle:**
  1. Mark `@deprecated since vX (date), use Y, removed in vZ`
  2. Migrate internal callers (creates the proof the replacement works)
  3. Wait one major or N months for external callers
  4. Remove

Reference: [Semantic Versioning — Major changes](https://semver.org/) · [PHPDoc @deprecated](https://docs.phpdoc.org/3.0/guide/references/phpdoc/tags/deprecated.html)

---


## Untracked Technical Debt

**Impact: MEDIUM (Debt that isn't tracked can't be prioritized or budgeted)**

Debt that exists only in engineers' heads (or in scattered TODOs and Slack messages) competes with product features by stealth — engineers slow down, but leadership can't see why. A debt register makes the cost visible, so it can be funded properly instead of being paid in invisible overtime.

## How to Detect

Check the project for an explicit debt-tracking mechanism:

- **Issue tracker label** (e.g., GitHub `tech-debt`, Linear `Debt` project)
- **Dedicated debt board** (Trello, Notion, or a `DEBT.md` in the repo)
- **ADRs** for major debt accumulation decisions
- **Quarterly debt-paydown allocation** (e.g., 20% capacity)

```bash
# Quick repo check
ls DEBT.md TECH_DEBT.md docs/debt/ 2>/dev/null
gh issue list --label "tech-debt" --state=all
grep -rE 'tech.?debt' .github/ docs/
```

If none of these exist, the project has **process debt about technical debt** — meta-debt.

## Incorrect

```
❌ Debt situation in a typical repo:
- 47 TODO comments, no tickets
- 12 known "we should rewrite that" conversations on Slack
- 3 engineers each have a mental list of "things that scare me"
- Last debt-paydown sprint: never
- When asked "what's our biggest debt?": four engineers give four different answers
```

**Problems:**
- Debt accrues invisibly — leadership only sees velocity drop
- Same problem gets discovered repeatedly by new hires
- No paydown budget because no list to justify the budget

## Correct

Pick **one** lightweight mechanism and use it consistently:

```markdown
<!-- ✅ Option A: DEBT.md in the repo (low ceremony) -->
# Technical Debt Register

| # | Category | Item | Effort | Impact | Owner | Linked |
|---|----------|------|--------|--------|-------|--------|
| 1 | deps     | guzzle 6.x (5y behind, blocks PHP 8.4)  | M | HIGH     | @asyraf | #1842 |
| 2 | code     | OrderService god class (820 LoC)        | L | HIGH     | @team-orders | #1844 |
| 3 | test     | Checkout flow has no integration test   | M | CRITICAL | @asyraf | #1845 |

Last reviewed: 2026-05-01.  Next review: 2026-08-01.
```

```yaml
# ✅ Option B: GitHub label + saved search
# Label every debt-related issue with 'tech-debt'
# Saved search: https://github.com/org/repo/issues?q=is%3Aopen+label%3Atech-debt+sort%3Areactions-%2B1-desc
```

**Allocate budget:** dedicate a consistent fraction of every sprint to debt paydown (commonly 15–25%). Without an allocation, debt always loses to features.

**Benefits:**
- Debt is visible to product and leadership
- Prioritization is principled (effort × impact), not loudest-engineer
- Paydown velocity is measurable

## Remediation Strategy

- **Effort:** S to start (one register + a label), ongoing M to maintain
- **When to pay down:**
  - **Now:** start the register with the top 10 items from your last audit
  - **Quarterly:** review and reprioritize; close completed entries
  - **Per PR:** if a PR introduces accepted debt (shortcut, missing test), add a register entry as part of merge

**Anti-patterns:**
- Register that nobody owns → goes stale, becomes worse than nothing
- Register with 200 entries → useless; cap at top 20–30 active items
- Debt sprints disconnected from a register → effort goes to whatever's annoying that week, not what matters

Reference: [Martin Fowler — Technical Debt Quadrant](https://martinfowler.com/bliki/TechnicalDebtQuadrant.html) · [ThoughtWorks Tech Radar — Debt Register](https://www.thoughtworks.com/radar)

---


## Code Without Owners

**Impact: MEDIUM (Orphaned code = nobody reviews, nobody maintains, nobody knows)**

When code has no clear owner, two things happen: PRs touching it stall (nobody knows who should approve), and when it breaks at 2am, the on-call rotation finds out the hard way. Orphaned code accumulates as the "bus-factor of one" engineer who wrote it changes teams or leaves.

## How to Detect

```bash
# Files not covered by CODEOWNERS
gh api repos/:owner/:repo/contents | jq -r '.[].path' > all-files.txt
# Compare against CODEOWNERS patterns (manual or via `git check-attr` for path-attribute alternative)

# "Author concentration" — files where one person wrote >70% and they're gone
git ls-files | while read f; do
  TOP=$(git log --format='%ae' -- "$f" | sort | uniq -c | sort -rn | head -1)
  echo "$TOP $f"
done | sort -rn | head -30

# High-churn, low-author-count files (bus-factor risk)
git log --since='12 months ago' --name-only --format='COMMIT %ae' | \
  awk '/^COMMIT/{a=$2; next} {print a, $0}' | \
  sort -k2 | uniq -c | sort -rn | head

# Repos lacking a CODEOWNERS file at all
ls .github/CODEOWNERS docs/CODEOWNERS CODEOWNERS 2>/dev/null
```

## Incorrect

```
❌ A typical legacy repo:

- No .github/CODEOWNERS file
- 40% of files last touched by engineers who left 2+ years ago
- Critical billing module written by one engineer, no shared knowledge
- Incident response for `/api/legacy/*` routes pages a randomly-selected on-call
  who has never seen the code
- PRs touching low-traffic areas wait 2 weeks for a review because nobody owns them
```

**Problems:**
- Knowledge debt is invisible until incident — then it's catastrophic
- Code reviews degrade to rubber-stamps because nobody has context
- Refactoring is risky — "is anyone using this?" has no quick answer

## Correct

```
# ✅ .github/CODEOWNERS — every path has an owner team

# Default owners for everything not matched below
*                               @org/platform

# Domain owners
/app/Billing/                   @org/billing-team
/app/Auth/                      @org/identity-team
/app/Notifications/             @org/messaging-team
/resources/js/Pages/Admin/      @org/admin-frontend

# Infrastructure / DevEx
/.github/workflows/             @org/devex
/deploy/                        @org/devex

# Database
/database/migrations/           @org/data-engineering @org/platform
```

Add a CI step to validate the file and check coverage:

```yaml
- name: CODEOWNERS lint + coverage
  uses: mszostok/codeowners-validator@v0.7.4
  with:
    checks: "files,owners,duppatterns,syntax"
    experimental_checks: "notowned"
    github_access_token: ${{ secrets.GITHUB_TOKEN }}
```

The `notowned` experimental check flags files in the repo not matched by any CODEOWNERS pattern — the right tool for "is everything covered?".

**Benefits:**
- GitHub auto-requests reviews from owners — no more "who should review this?"
- Incident response routes to the right team
- New engineers can find a domain expert by path
- Refactor decisions get the input they need

## Remediation Strategy

- **Effort:**
  - **S** — bootstrap a CODEOWNERS with broad team-level patterns
  - **M** — refine ownership as teams form; add path-specific owners
  - **L** — rehome orphaned code (find a new owner team, document context, transfer ownership)
- **When to pay down:**
  - **NOW:** if you've ever asked "who owns this code?" and the answer was unclear
  - **As a project:** quarterly ownership review — re-attest that owners listed are still active
- **Anti-patterns:**
  - **Individual owners** for production code (bus-factor of 1) — prefer team owners
  - **Stale CODEOWNERS** referencing teams or people that no longer exist (CI lint catches this)
  - **Owning everything** by one team — fragment by domain so PRs route fast

**Tip:** when no team wants to own a piece of code, that's a strong signal to delete it (if unused), extract it (if shared), or fold it into a new team's charter (if business-critical).

Reference: [GitHub — CODEOWNERS](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners) · [GitLab — Code Owners](https://docs.gitlab.com/ee/user/project/codeowners/)

---


## Lingering Feature Flags

**Impact: MEDIUM (Old flags become permanent special cases — dead branches that never die)**

Feature flags are a release-management tool, not a config primitive. Once a feature has been at 100% rollout for weeks, the flag is no longer protecting anyone — it's just dead branches polluting the codebase. Lingering flags accumulate as conditional logic that nobody understands, can't safely delete, and silently doubles the test surface.

## How to Detect

```bash
# Search for feature-flag SDK references
grep -rEn '(feature|flag|toggle)::(isOn|enabled|isEnabled|active)' \
  --include='*.php' --include='*.ts' --include='*.tsx' --include='*.js' .

# Specific SDKs
grep -rEn '(LaunchDarkly|posthog|growthbook|unleash|split\.io|flipper|optimizely)' .

# Laravel pennant
grep -rEn 'Feature::active\(|Feature::for\(' app/ resources/

# Count distinct flag identifiers and look up their ages in the flag platform
# (most platforms expose a "stale flags" view — LaunchDarkly, Statsig, GrowthBook all do)
```

For each flag found, check:
- **Rollout state:** 0%? 100%? Or in-flight rollout? (Anything > 6 weeks at 0 or 100% is debt.)
- **Last evaluation:** flag platform usually tracks last-checked time per flag
- **Owner:** is there a name attached? Is that person still on the team?

## Incorrect

```typescript
// ❌ Flag from 2 years ago, shipped to 100% for 18 months, still in code
function CheckoutPage() {
  const showNewCheckout = useFlag('new-checkout-redesign-v3');   // long since 100%
  return showNewCheckout ? <NewCheckout /> : <LegacyCheckout />;
}

// ❌ Nested flag conditions become an N×M maze
if (flags.useNewPricing && flags.useNewTaxEngine && !flags.legacyShippingFallback) {
  // ... one path
} else if (flags.useNewPricing && !flags.useNewTaxEngine) {
  // ... another path
} else {
  // ... legacy path nobody has touched in a year
}
```

**Problems:**
- `LegacyCheckout` is dead code masquerading as a fallback
- The flag service is queried for every page load even though the result is constant
- New engineers see 3 versions of checkout and don't know which is current
- Removing the flag is "scary" because nobody has touched the legacy branch in months

## Correct

```typescript
// ✅ Flag retired: branch chosen, dead branch removed, flag deleted from platform
function CheckoutPage() {
  return <Checkout />;     // formerly NewCheckout
}
// LegacyCheckout: deleted. New-checkout-redesign-v3 flag: archived in LaunchDarkly.
```

Lifecycle policy as a CI gate:

```yaml
# .github/workflows/stale-flags.yml — runs weekly
# LaunchDarkly: use the official action (the tool is a Go binary called
# `ld-find-code-refs`, not an npm package — `npx` will not work).
- name: Stale flag check (LaunchDarkly)
  uses: launchdarkly/find-code-references@v2
  with:
    accessToken: ${{ secrets.LD_ACCESS_TOKEN }}
    projKey:     ${{ vars.LD_PROJECT_KEY }}
    repoName:    ${{ github.event.repository.name }}
```

Maintain a register:

```markdown
| Flag                          | State    | Created    | Owner    | Action |
|-------------------------------|----------|------------|----------|--------|
| new-checkout-redesign-v3      | 100% 18mo| 2024-09-01 | @asyraf  | DELETE |
| experimental-promo-engine     | 50% A/B  | 2026-04-01 | @growth  | KEEP   |
| disable-legacy-search         | 0% 9mo   | 2025-09-15 | @search  | DELETE |
```

**Benefits:**
- Dead branches removed → smaller test matrix, simpler code, smaller bundle
- Flag platform stops being queried for permanent constants (latency win)
- Onboarding engineers see one path, not three

## Remediation Strategy

- **Effort:**
  - **S** — retire a single flag (delete code branch + remove flag)
  - **M** — clean up nested flag combinations
  - **L** — instate a flag-lifecycle program (creation requires expiry date, weekly stale-flag review)
- **When to pay down:**
  - **NOW:** any flag at 100% for > 6 weeks with no rollback plan
  - **Per sprint:** drop one stale flag — small effort, compounding cleanup
  - **Then:** lifecycle policy — every new flag has an expiry date in the platform

**Lifecycle policy (recommended):**
1. **Create** — flag has a name, owner, intended rollout duration, and target removal date
2. **Roll out** — gradually 1% → 5% → 25% → 100%
3. **Stabilize** — at 100% for one release cycle; verify metrics
4. **Retire** — delete the losing branch, remove flag from code, archive in platform
5. **Audit** — weekly stale-flag report; any flag past expiry pings the owner

**Anti-patterns:**
- **Permanent flags used as "config"** — that's not a flag, that's an env var; treat differently
- **Flags as authz** — use proper authorization layers, not flag SDKs
- **No expiry on creation** — every flag should be born with a death date

Reference: [Martin Fowler — Feature Toggles](https://martinfowler.com/articles/feature-toggles.html) · [ld-find-code-refs (GitHub)](https://github.com/launchdarkly/ld-find-code-refs) · [Laravel Pennant](https://laravel.com/docs/pennant)

---

