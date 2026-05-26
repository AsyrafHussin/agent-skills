# Code Slop Detection - Complete Reference

**Version:** 1.0.0
**Organization:** Agent Skills Contributors
**Date:** May 2026
**License:** MIT

## Abstract

Taste-level review of code for AI-generated patterns in **PHP/Laravel and TypeScript/React** projects. Contains 24 rules across 6 categories covering comments, naming, over-engineering, defensive overdose, test slop, and style fingerprints. Where `technical-debt` measures quantitative code debt, this skill measures the qualitative failure mode: code that passes every metric but reads like a tutorial blog post — not human-written. Supports audit-mode output (CLEAN / SUSPICIOUS / INFLATED / CRITICAL).

## How to Use

When asked to "review for AI slop", "audit code-quality taste", or "find AI patterns", run through this skill's rules as a checklist against changed files (PR diff) or the full repo. For each file, output a verdict (CLEAN / SUSPICIOUS / INFLATED / CRITICAL) with top findings and a suggested action. End with a slop ledger summary.

## References

- [arXiv 2510.03029 — Investigating the Smells of LLM-Generated Code](https://arxiv.org/abs/2510.03029)
- [arXiv 2602.00409 — Are Coding Agents Generating Over-Mocked Tests?](https://arxiv.org/abs/2602.00409)
- [hardikpandya/stop-slop](https://github.com/hardikpandya/stop-slop)
- [flamehaven01/AI-SLOP-Detector](https://github.com/flamehaven01/AI-SLOP-Detector)

## Step 1: Detect Project Stack

Most rules are stack-agnostic in concept, but examples and detection commands differ between PHP and TypeScript.

| Signal | Stack | Tooling |
|--------|-------|---------|
| `composer.json` | PHP / Laravel | `phpstan`, `phpcs`, `phpmd`, manual grep |
| `package.json` | Node / TS / React | `eslint`, `tsc --noEmit`, `knip`, manual grep |

---

# Sections

This file defines all sections, their ordering, impact levels, and descriptions.
The section ID (in parentheses) is the filename prefix used to group rules.

---

## 1. Comments (comments)

**Impact:** CRITICAL
**Description:** The single loudest AI tell. Models love to narrate the next line in the comment above it, write empty docblocks that restate the function signature, and leave placeholder comments like `// TODO: implement` long after the code shipped. Cleaning these out is the fastest path to "this looks human-written".

## 2. Naming (naming)

**Impact:** CRITICAL
**Description:** AI-generated code swings between two opposite failure modes — generic placeholders (`data`, `result`, `info`) and over-descriptive run-ons (`theUserWhoIsCurrentlyLoggedIn`) — often in the same file. A human teammate converges on a register. Suffix abuse (`*Helper`, `*Manager`, `*Util`) and type-in-name patterns (`userObject`, `resultArray`) round out the family.

## 3. Over-engineering (over-eng)

**Impact:** HIGH
**Description:** Adding layers nobody asked for. Premature interfaces with one implementation, single-method classes that should be functions, wrappers called from one place, and pulling in a new dependency when an existing one does the job. AI defaults to "enterprise-grade" because its training distribution is enterprise-grade.

## 4. Defensive overdose (defensive)

**Impact:** HIGH
**Description:** Try/catch wrapped around code that can't throw. Null checks after a non-null assertion. `if (array && array.length > 0)` three times in the same function. Meanwhile real defenses (timeouts on external calls, rate limits, circuit breakers) are missing. AI is defensive in the wrong places.

## 5. Test slop (test)

**Impact:** HIGH
**Description:** Tests that mock everything and assert nothing. Tests that mirror the implementation's logic — they pass because they re-encode the same code, not because they verify behaviour. "Doesn't throw" assertions that don't check anything meaningful. Snapshot abuse instead of behavioural assertions.

## 6. Style fingerprints (style)

**Impact:** MEDIUM
**Description:** The small tells: hyper-consistent formatting (no human drift), `as any` / `@ts-ignore` sprinkled where types are hard, *absence* of `// HACK:` / `// XXX:` scars (real codebases have geology), stray `console.log` / `dd()` debug artifacts, and trivial boilerplate like `if (x) return true; else return false`.

---


## Narration Comments

**Impact: CRITICAL (The single loudest AI tell; cuts reading speed and signals the author didn't read what they shipped)**

Models love narrating the next line in a comment above it. The comment adds no information — it just restates the code in English. A reader has to read both, decide they say the same thing, and feel a tiny tax of distrust. Across a 500-line file, the tax compounds into "I have no idea what this codebase is doing."

A useful comment answers **why**, not **what**. If the code itself says *what*, the comment is noise.

## Incorrect

```php
// ❌ Comments narrate the next line — pure AI fingerprint

public function createUser(array $data): User
{
    // Validate the input data
    $validated = Validator::make($data, [...])->validate();

    // Create a new user with the validated data
    $user = User::create($validated);

    // Send a welcome email to the user
    Mail::to($user)->send(new WelcomeEmail($user));

    // Return the newly created user
    return $user;
}
```

```typescript
// ❌ Same pattern in TS — every line narrated above
function calculateTotal(items: Item[]): number {
  // Initialize the total to zero
  let total = 0;

  // Loop through each item in the items array
  for (const item of items) {
    // Add the item's price multiplied by quantity to the total
    total += item.price * item.quantity;
  }

  // Return the calculated total
  return total;
}
```

**Why it's slop:**
- Every comment is a literal English translation of the code on the next line
- A reader's eyes have to traverse twice (comment → code) to learn nothing extra
- Suggests the author was thinking out loud rather than thinking ahead
- A human author who knew this code wouldn't bother writing these comments

## Correct

```php
// ✅ Comments removed (the code says what it does); explain WHY only if non-obvious

public function createUser(array $data): User
{
    $validated = Validator::make($data, [...])->validate();

    $user = User::create($validated);
    Mail::to($user)->send(new WelcomeEmail($user));

    return $user;
}
```

```typescript
// ✅ Total comments removed; if any line needed comment, it'd be a WHY
function calculateTotal(items: Item[]): number {
  return items.reduce((sum, i) => sum + i.price * i.quantity, 0);
}
```

If a comment IS needed, it explains *why*:

```typescript
// ✅ Good comment — explains a hidden constraint
// Note: `total` is in cents (Stripe API uses integer minor units).
function calculateTotal(items: Item[]): number {
  // Stripe rejects single charges > 999,999_99 cents (= $999,999.99). Cap to avoid charge failure.
  const total = items.reduce((sum, i) => sum + i.price * i.quantity, 0);
  return Math.min(total, 99_999_999);   // 999,999.99 expressed in cents
}
```

**Why it reads human:**
- Code speaks for itself; reader processes once
- The one remaining comment carries a *load-bearing* piece of context (Stripe constraint) you couldn't infer from the code
- Signals the author understood the code well enough to know what didn't need explaining

## Detection

```bash
# Heuristic: comments immediately followed by code that uses the comment's keyword
# (rough but effective — flags lines where the comment is one word-of-code below)
grep -rEn '^[[:space:]]*//.*' --include='*.ts' --include='*.tsx' --include='*.js' --include='*.jsx' src/ | head -50
grep -rEn '^[[:space:]]*//.*' --include='*.php' app/ | head -50

# Stronger heuristic: comment + next non-blank line share a keyword
# (run in code review tools or PHPStorm "Comment Density" inspections)

# Easiest enforcement: ESLint capital-comments / no-inline-comments rules combined
# with a team-level "comments answer WHY only" code-review checklist.
```

There is no fully automatic linter for this — it requires a reviewer's judgement. The detection pattern is "does the comment translate, or add context?"

Reference: [Code Complete — Self-Documenting Code](https://www.microsoftpressstore.com/store/code-complete-9780735619678) · Internal: [`comments-empty-docblocks`](comments-empty-docblocks.md)

---


## Empty Docblocks That Restate the Signature

**Impact: HIGH (Adds bytes, zero information; suggests the author auto-generated without reading)**

A docblock that adds nothing the signature already says is a tell. `/** Get the user */` above `function getUser(): User` tells the reader nothing the function name and return type don't. Modern PHP and TypeScript both encode types in the signature; redundant docblocks are pure noise.

Useful docblocks document **edge cases, thrown exceptions, side effects, or non-obvious semantics** — not "this method does what its name says".

## Incorrect

```php
// ❌ Restates the signature; adds nothing
/**
 * Get the user by ID.
 *
 * @param int $id The user ID
 * @return User The user
 */
public function getUser(int $id): User
{
    return User::findOrFail($id);
}

/**
 * Adds two numbers together.
 *
 * @param int $a First number
 * @param int $b Second number
 * @return int The sum
 */
public function add(int $a, int $b): int
{
    return $a + $b;
}
```

```typescript
// ❌ Same pattern in TS
/**
 * Get the user.
 * @param id - The user ID
 * @returns The user
 */
function getUser(id: string): User { /* ... */ }

/**
 * Adds two numbers.
 * @param a - First number
 * @param b - Second number
 * @returns The sum
 */
function add(a: number, b: number): number { return a + b; }
```

**Why it's slop:**
- The signature already says all of this — return type, parameter types, what's returned
- Bytes per useful info: ~zero
- A human writes a docblock when there's something the signature *can't* express

## Correct — option 1: no docblock at all

```php
// ✅ Signature speaks for itself
public function getUser(int $id): User
{
    return User::findOrFail($id);
}

public function add(int $a, int $b): int
{
    return $a + $b;
}
```

## Correct — option 2: docblock that earns its place

```php
// ✅ Docblock documents what the signature CAN'T say
/**
 * Get a user, or throw if not found.
 *
 * @throws ModelNotFoundException if no user matches $id
 * @throws AuthorizationException if the current user can't view $id
 */
public function getUser(int $id): User
{
    Gate::authorize('view', User::class);
    return User::findOrFail($id);
}
```

```typescript
// ✅ TSDoc that documents real edge cases
/**
 * @throws OutOfRangeError if `amount` exceeds the Stripe 999,999.99 cap.
 * @remarks Rounds to 2 decimal places using banker's rounding.
 */
function formatAmountForStripe(amount: number): string { /* ... */ }
```

**Why it reads human:**
- The remaining docblocks all carry information the signature can't
- A reader scanning the file sees docblocks and trusts they're load-bearing

## Detection

```bash
# Files with @param/@return docblocks
grep -rln '@param\|@return\|@returns' --include='*.php' app/
grep -rln '@param\|@returns' --include='*.ts' --include='*.tsx' src/

# PHPStan can flag redundant @param when the typed signature is more informative:
vendor/bin/phpstan analyse --level=9
# (set `reportAlwaysTrueInLastCondition: true` and use phpstan-deprecation-rules)

# ESLint rule for TS — flags JSDoc with no additional info:
# https://github.com/gajus/eslint-plugin-jsdoc → jsdoc/require-jsdoc { contexts: 'never' }
# Or simpler: turn OFF require-jsdoc and let humans write docblocks only when warranted.
```

The most reliable enforcement is a **PR review rule**: "every `@param` / `@return` must add information the signature doesn't already provide. Delete the rest."

Reference: [PHPDoc Tags Reference](https://docs.phpdoc.org/3.0/guide/references/phpdoc/tags/) · [TSDoc](https://tsdoc.org/) · Internal: [`comments-narration`](comments-narration.md)

---


## Placeholder Comments Left In

**Impact: CRITICAL (Reveals AI-generated code that was never completed; ships unfinished work as 'done')**

Common AI-generated placeholders that should never reach `main`:

```
// implementation
// implementation here
// your code here
// helper function
// TODO: implement
// TODO: implement this
// TODO: add error handling
// TODO: implement validation
// FIXME: figure out edge case
// HACK: temporary
// placeholder
// add your logic here
```

These mean the AI produced scaffolding the developer was supposed to fill in. If they reach the PR, they signal the author shipped without reading.

Distinct from properly-tagged debt markers — a `// TODO(#1234, asyraf): rewrite after Stripe v2 migration` with an owner, ticket, and concrete unblocking condition is *not* slop. That's tracked debt.

## Incorrect

```php
// ❌ Placeholder comments shipped as 'done'

public function calculateRefund(Order $order): Money
{
    // TODO: implement
    return Money::zero();
}

public function process(Order $order): void
{
    // Validate the order
    // TODO: add validation

    $this->charge($order);

    // helper function
    // TODO: implement error handling
}
```

```typescript
// ❌ Same in TS
function exportUsers(): User[] {
  // your code here
  return [];
}

function handlePayment(token: string) {
  // implementation
  // TODO: implement
}
```

**Why it's slop:**
- The function name promises a behaviour the body doesn't deliver
- Reviewers usually skip past `// TODO` without realising the function is empty
- These slip through tests because the placeholder body type-checks
- If genuinely intended as future work, it belongs in the issue tracker, not the source file

## Correct

```php
// ✅ Either implement, or fail loud, or remove the method entirely

public function calculateRefund(Order $order): Money
{
    throw new BadMethodCallException(
        'Refund calculation not yet implemented — see #1842 for the policy spec.'
    );
}

// ✅ Or genuine, tracked work-in-progress with owner + ticket:
// TODO(#1842, asyraf): cap refunds at original-charge-minus-shipping
public function calculateRefund(Order $order): Money
{
    return $order->total->subtract($order->shippingCost);
}
```

```typescript
// ✅ Fail loud so the gap is impossible to ship past
function exportUsers(): User[] {
  throw new Error('exportUsers: not implemented (see #420)');
}
```

**Why it reads human:**
- An empty function body that throws is impossible to merge by accident; tests will catch it
- A tracked `TODO` with `(#issue, @owner)` is real debt, not abandoned scaffolding
- A human author either implements the function or explicitly fails — they don't ship "Returns nothing for now"

## Detection

```bash
# Common AI placeholders — match only when prefixed by a comment marker
# (without the required prefix, the regex would flood on class names like "ServiceImplementation").
grep -rEn --include='*.php' --include='*.ts' --include='*.tsx' --include='*.js' --include='*.jsx' \
  '(//|/\*|#)\s*(implementation|implementation here|your code here|helper function|placeholder|add your logic|add error handling|add validation)\b' \
  app/ src/ 2>/dev/null

# Bare TODO/FIXME (without ticket reference)
grep -rEn --include='*.php' --include='*.ts' --include='*.tsx' --include='*.js' --include='*.jsx' \
  '(TODO|FIXME)([^(#]|$)' app/ src/ 2>/dev/null | grep -v '#[0-9]'

# Combined: TODOs lacking issue references — CI gate
NEW=$(git diff --name-only --diff-filter=ACM origin/main...HEAD)
BARE=$(echo "$NEW" | xargs grep -EnH '\b(TODO|FIXME)\b' 2>/dev/null | grep -v '#[0-9]')
test -z "$BARE" || { echo "Bare TODO/FIXME found — add (#issue, @owner):"; echo "$BARE"; exit 1; }
```

See also `technical-debt`'s `process-todo-fixme-aging` rule — the broader policy for tracked debt markers.

Reference: Internal: [`comments-narration`](comments-narration.md)

---


## Closing-Brace Labels

**Impact: MEDIUM (Relic of older training corpora; clear AI fingerprint that modern editors make pointless)**

`} // end function` / `} // end if block` / `} // close foreach` are a strong AI tell. The pattern comes from older C/COBOL/Verilog training data where blocks could span hundreds of lines and editors didn't fold scope. Modern PHP, TypeScript, and any editor with bracket-matching makes the label pure noise.

If your function is so long you need a label on its closing brace, the real fix is to split the function — not annotate the brace.

## Incorrect

```php
// ❌ Labels on closing braces

class OrderService
{
    public function process(Order $order): void
    {
        if ($order->isPaid()) {
            foreach ($order->items as $item) {
                if ($item->requiresShipping()) {
                    $this->ship($item);
                } // end if requiresShipping
            } // end foreach items
        } // end if isPaid
    } // end function process
} // end class OrderService
```

```typescript
// ❌ TS variant
function processOrder(order: Order): void {
  if (order.isPaid()) {
    order.items.forEach(item => {
      if (item.requiresShipping()) {
        ship(item);
      } // end if requiresShipping
    }); // end forEach
  } // end if isPaid
} // end function processOrder
```

**Why it's slop:**
- Every modern IDE highlights the matching opener when the cursor is on a brace
- The labels add bytes for zero information
- A function that needed labels for clarity is a function that needs to be split
- Pattern is recognisably training-data noise (PHP-from-2008, Verilog, COBOL)

## Correct

```php
// ✅ Either small enough not to need labels, OR split for clarity

class OrderService
{
    public function process(Order $order): void
    {
        if (!$order->isPaid()) {
            return;
        }
        $this->shipEligibleItems($order);
    }

    private function shipEligibleItems(Order $order): void
    {
        foreach ($order->items->filter->requiresShipping() as $item) {
            $this->ship($item);
        }
    }
}
```

```typescript
// ✅ Smaller scopes, no labels needed
function processOrder(order: Order): void {
  if (!order.isPaid()) return;
  shipEligibleItems(order);
}

function shipEligibleItems(order: Order): void {
  order.items
    .filter(i => i.requiresShipping())
    .forEach(ship);
}
```

**Why it reads human:**
- Short functions; the closing brace is visually close to the opener
- No annotation noise; editor handles brace matching
- If a developer felt the need to label, they'd split the function instead

## Detection

```bash
# Closing brace followed by an "end" / function/class/if/foreach annotation comment
grep -rEn '^\s*\}\s*(\)\s*)?\s*(//|#).*\b(end|close|finish)\b' \
  --include='*.php' --include='*.ts' --include='*.tsx' --include='*.js' --include='*.jsx' \
  app/ src/

# Specific PHP closing-tag labels
grep -rEn '^\s*\}\s*//.*(end (function|class|if|foreach|for|while|switch))' --include='*.php' app/
```

A single hit is borderline; **three or more in one file is an unmistakable AI fingerprint.**

Reference: [Code Complete — Chapter 32 on Self-Documenting Code](https://www.microsoftpressstore.com/store/code-complete-9780735619678) · Internal: [`comments-narration`](comments-narration.md)

---


## Generic Placeholder Names

**Impact: CRITICAL (Reduces every variable to 'a thing' — kills the most powerful form of self-documentation)**

`data`, `result`, `info`, `temp`, `value`, `item`, `helper`, `manager`, `utils` — these are the names a model picks when it doesn't know what the value represents in the domain. They pass linters, type checks, every metric. They cost the reader real attention to track ("which `data` is this?").

Names are the cheapest, highest-leverage form of self-documentation in any codebase. A model that names every intermediate variable `data` or `result` is signalling it didn't think about the domain.

## Incorrect

```php
// ❌ Every variable is generic

public function exportUsers(): Collection
{
    $data = User::all();

    $result = [];
    foreach ($data as $item) {
        $info = [
            'name' => $item->name,
            'email' => $item->email,
        ];
        $result[] = $info;
    }

    return collect($result);
}
```

```typescript
// ❌ Same pattern
async function fetchOrders(userId: string) {
  const data = await api.getOrders(userId);
  const result = data.map(item => {
    const info = {
      id: item.id,
      total: item.total,
    };
    return info;
  });
  return result;
}
```

**Why it's slop:**
- Every variable is "a thing"; reader has no anchor to the domain
- `$item` inside a loop over `$data` is doubly opaque — what kind of item? what kind of data?
- A reader hitting line 50 of this file can't tell what `$result` holds without tracing the whole function
- A human author who lived with this code would use domain words

## Correct

```php
// ✅ Names tell you what the values are in the domain

public function exportUsers(): Collection
{
    return User::all()
        ->map(fn (User $user) => [
            'name'  => $user->name,
            'email' => $user->email,
        ]);
}
```

```typescript
// ✅ Domain words everywhere
async function fetchOrders(userId: string): Promise<OrderSummary[]> {
  const orders = await api.getOrders(userId);
  return orders.map(order => ({
    id:    order.id,
    total: order.total,
  }));
}
```

**Why it reads human:**
- A reader on any line knows the domain object in scope
- Domain names compose — `orders.map(order => …)` reads as a sentence
- The original `$data → $item → $info → $result` chain collapses into one expression because the names made the intermediate variables unnecessary

## When generic names ARE OK

A handful of names are conventional, short-scope, and fine:

- **`i`, `j`, `k`** — loop indices in a 3-line `for`
- **`x`, `y`, `z`** — math/geometry (coordinates)
- **`_`** — explicit "ignored value"
- **`acc`** — accumulator in a `reduce` callback
- **`prev`, `next`** — in middleware / chained handlers where they're language conventions
- **`req`, `res`** — Express handlers (don't fight a framework convention)

If the value flows through 5+ lines or escapes the immediate function, use a domain name.

## Detection

```bash
# Bare local-variable declarations using generic names (PHP)
grep -rEn '\$(data|result|info|temp|item|helper|value)\b' --include='*.php' app/ | wc -l

# TS/JS — generic const/let
grep -rEn '\b(const|let)\s+(data|result|info|temp|helper|value|item)\b' --include='*.ts' --include='*.tsx' --include='*.js' src/ | wc -l

# Density signal: files with > 10 generic-name hits
# (in a 200-line file, 10+ generic names is suspicious)
```

There's no fully automatic detector — judgment is required. The signal is **density**: a few are fine, a thicket is a slop fingerprint.

Reference: [Clean Code (Robert C. Martin) — Chapter 2: Meaningful Names](https://www.oreilly.com/library/view/clean-code/9780136083238/) · Internal: [`naming-over-descriptive`](naming-over-descriptive.md)

---


## Over-Descriptive Run-On Names

**Impact: HIGH (Inflated identifiers that read like English instead of code)**

The opposite failure mode of generic names: instead of `user`, the AI produces `theUserWhoIsCurrentlyLoggedIn`. Instead of `total`, `calculateTotalAmountFromItemsList`. The identifier becomes a sentence. Real engineers know that *context* (function scope, parameter types, surrounding code) carries half the meaning — you don't need to encode all of it in every variable name.

A name should be **as long as needed and no longer**. Once context establishes "we're inside `processOrder`", calling the order variable `order` is fine. Calling it `theOrderThatIsCurrentlyBeingProcessed` is slop.

## Incorrect

```php
// ❌ Identifiers read as English

public function processOrderForCheckout(
    OrderThatIsCurrentlyBeingProcessed $theOrderToProcess
): ProcessingResultObjectContainingStatus {
    $theCurrentlyLoggedInUserWhoIsCheckingOut = $theOrderToProcess->getUserWhoPlacedTheOrder();
    $theTotalAmountThatNeedsToBeCharged = $this->calculateTotalAmountFromItemsListInTheOrder($theOrderToProcess);

    return new ProcessingResultObjectContainingStatus(
        status: 'success',
        amountThatWasCharged: $theTotalAmountThatNeedsToBeCharged,
    );
}
```

```typescript
// ❌ Same in TS
async function fetchTheUserDataObjectForTheUserWithTheGivenId(
  theIdOfTheUserToFetch: string,
): Promise<TheUserDataResponseObject> {
  const theResponseFromTheAPI = await api.getUser(theIdOfTheUserToFetch);
  return theResponseFromTheAPI;
}
```

**Why it's slop:**
- Function signatures span multiple lines because identifiers eat the line budget
- Reader has to scan past redundant words ("theOrderToProcess" — yes, it's the parameter, of course we're processing it)
- "The" prefix is meaningless — what else would it be?
- Inflates token count without conveying anything new

## Correct

```php
// ✅ Tight; context carries the obvious

public function checkout(Order $order): ProcessingResult
{
    $user = $order->user;
    $total = $this->totalFor($order);

    return new ProcessingResult(status: 'success', charged: $total);
}
```

```typescript
// ✅ Tight TS
async function fetchUser(id: string): Promise<User> {
  return api.getUser(id);
}
```

**Why it reads human:**
- The function is named `checkout`; obviously the parameter is the order being checked out — no need to spell that out
- Function fits on one line; reader takes it in at a glance
- Domain words (`Order`, `User`, `ProcessingResult`) replace English sentences

## Guidelines

| Scope | Reasonable length |
|---|---|
| Loop index | 1 char (`i`) |
| Block-local (< 10 lines) | 1 word (`user`, `total`) |
| Function-local | 1–2 words (`pricedItems`, `taxedTotal`) |
| Public API parameter | 2–3 words when domain requires (`stripeChargeId`, `webhookPayload`) |
| Constants / config keys | Domain-precise (`MAX_STRIPE_AMOUNT_CENTS`) — those CAN be long |
| Class names | 1–3 words (`OrderService`, `RefundCalculator`) |

The rule of thumb: **if you can drop a word and meaning survives, drop it.**

## Detection

```bash
# Identifiers with 4+ camelCase words (heuristic; tune to taste).
# Note: grep -E (ERE) does NOT support (?:...) non-capturing groups; use a capturing group.
grep -rEn '[a-z]+([A-Z][a-z]+){3,}' --include='*.ts' --include='*.tsx' --include='*.js' src/ \
  | head -30

# PHP equivalent (variables / method names)
grep -rEn '\$[a-z][a-zA-Z]{20,}' --include='*.php' app/ | head -30
grep -rEn 'function [a-z][a-zA-Z]{30,}\(' --include='*.php' app/ | head -30

# Names starting with "the" — a strong AI tell
grep -rEnH '\b(theUser|theOrder|theData|theResult|theItem|theCurrent)' \
  --include='*.php' --include='*.ts' --include='*.tsx' app/ src/
```

Reference: [Clean Code — Chapter 2](https://www.oreilly.com/library/view/clean-code/9780136083238/) · Internal: [`naming-generic-placeholders`](naming-generic-placeholders.md) (the opposite failure mode)

---


## Suffix Abuse — *Helper, *Manager, *Util, *Wrapper, *Processor

**Impact: HIGH (Catch-all suffixes signal undecided design and pad code with empty abstractions)**

`UserHelper`, `OrderManager`, `DataUtil`, `RequestWrapper`, `PaymentProcessor` — these suffixes are the AI's go-to when it doesn't know what to call something. The suffix is non-information: every class is in some sense a "manager" or "helper" or "util". The name describes the noun-form of "code I had to put somewhere".

When a model generates these, it's because:
- The "Helper" class is doing things that belong on the model
- The "Manager" is a procedural blob masquerading as a class
- The "Util" is a kitchen sink that should be split or absorbed into domain types

Real domain code uses domain names: `Pricing`, `RefundPolicy`, `WebhookVerifier` — not `PricingHelper`, `RefundManager`, `WebhookProcessor`.

## Incorrect

```php
// ❌ Suffix abuse — every concern wrapped in *Helper or *Manager

class UserHelper
{
    public function formatName(User $user): string {
        return $user->firstName . ' ' . $user->lastName;
    }
}

class OrderManager
{
    public function processOrder(Order $order): void {
        // ...
    }
}

class PaymentProcessor
{
    public function processPayment(Order $order, string $token): Charge {
        // ...
    }
}

class DataUtil
{
    public static function arrayToCsv(array $rows): string { /* ... */ }
    public static function csvToArray(string $csv): array { /* ... */ }
    public static function snakeToCamel(string $s): string { /* ... */ }
    public static function camelToSnake(string $s): string { /* ... */ }
    // …grows forever
}
```

```typescript
// ❌ Same TS pattern
class StringHelper {
  static capitalize(s: string): string { /* ... */ }
  static slugify(s: string): string { /* ... */ }
}

class ResponseWrapper {
  constructor(private res: ApiResponse) {}
  getData() { return this.res.data; }
}
```

**Why it's slop:**
- `UserHelper.formatName(user)` could just be `user.fullName` on the User model
- `OrderManager.processOrder(order)` is two ways of saying the same thing
- `DataUtil` is a kitchen sink — every utility ends up here, none have a clear home
- The suffix doesn't add information; it admits the author didn't decide what the class is

## Correct

```php
// ✅ Behaviour lives on the domain type; helpers split by concern

class User extends Model
{
    public function getFullNameAttribute(): string
    {
        return "{$this->firstName} {$this->lastName}";
    }
}

// Action class — replaces "OrderManager.processOrder"
final class PlaceOrder
{
    public function __invoke(Order $order, PaymentIntent $payment): void { /* ... */ }
}

// Domain class with a clear single responsibility — replaces "PaymentProcessor"
final class StripePaymentGateway implements PaymentGateway
{
    public function charge(Money $amount, string $token): Charge { /* ... */ }
}

// CSV becomes its own type — replaces the kitchen-sink DataUtil
final class CsvExporter
{
    public function export(iterable $rows, array $headers): string { /* ... */ }
}
```

```typescript
// ✅ Methods on the domain type, or named domain functions
class User {
  get fullName(): string { return `${this.firstName} ${this.lastName}`; }
}

// Free function — capitalize doesn't need a class wrapper
export function capitalize(s: string): string { /* ... */ }
export function slugify(s: string): string { /* ... */ }
```

**Why it reads human:**
- Each class has a specific domain responsibility (CsvExporter exports CSVs; PlaceOrder places orders)
- Behaviour-on-data is on the type that holds the data (`user.fullName`)
- Free functions stand on their own when there's no state to wrap

## When suffixes ARE okay

A handful of suffixes carry real meaning in their conventions:

- **`*Repository`** — DDD-style data access (specific contract)
- **`*Service`** — used sparingly for orchestration that doesn't fit on a domain type
- **`*Controller`** — HTTP entry point (Laravel/Express convention)
- **`*Middleware`** — request pipeline (Express/Laravel convention)
- **`*Gateway`** — external integration boundary (Stripe, AWS)
- **`*Listener`** / `*Observer` — event-handler conventions (Laravel)

`*Helper`, `*Manager`, `*Util`, `*Wrapper`, `*Processor` are the suspect tier.

## Detection

```bash
# Class declarations with suspect suffixes
grep -rEn 'class\s+[A-Z][a-zA-Z]*(Helper|Manager|Util|Utils|Wrapper|Processor|Handler)\b' \
  --include='*.php' --include='*.ts' --include='*.tsx' app/ src/

# Count by suffix to size the problem
grep -rE 'class\s+[A-Z][a-zA-Z]*' --include='*.php' --include='*.ts' app/ src/ \
  | grep -oE '(Helper|Manager|Util|Utils|Wrapper|Processor)\b' \
  | sort | uniq -c | sort -rn
```

A repo with **5+ `*Helper`** or **3+ `*Manager`** classes is almost certainly leaking domain logic into catch-all classes. Refactor by asking "what would I call this if I couldn't use the suffix?"

Reference: [Clean Code — Chapter 2: Meaningful Names](https://www.oreilly.com/library/view/clean-code/9780136083238/) · Internal: [`over-eng-single-method-class`](over-eng-single-method-class.md)

---


## Type Encoded in Variable Names

**Impact: MEDIUM (Embeds type in the identifier when the type system already does it; ages badly with refactors)**

`userObject`, `resultArray`, `stringData`, `listOfItems`, `userIdString`, `dataMap` — these names encode the *type* of the value, redundantly with the actual type declaration. Both PHP 8+ and TypeScript have rich type systems that show the type in the signature. The name should encode what the value **means**, not what its shape is.

The pattern is a tell because it suggests the author optimised for "obvious from any one line" rather than "obvious in context". A human author trusts the reader to look at the signature.

## Incorrect

```php
// ❌ Type-in-name everywhere

public function processItemsList(array $itemsArray, int $userIdInteger): array
{
    $resultArray = [];
    foreach ($itemsArray as $itemObject) {
        $priceFloat = $itemObject->priceFloat;
        $resultArray[] = $priceFloat;
    }
    return $resultArray;
}
```

```typescript
// ❌ Same in TS
function getUserDataObject(userIdString: string): UserDataResponseObject {
  const responseObject = await api.fetch(userIdString);
  const dataArray = responseObject.dataArray;
  const userObjectList = dataArray.map((itemObject: UserObject) => itemObject);
  return userObjectList;
}
```

**Why it's slop:**
- The type is declared on the parameter and visible in the signature
- Renaming `$itemsArray` to a `Collection` requires updating every variable reference
- The name `$itemObject` is redundant — the static type says it's a `User`
- Pattern is a clear "tutorial-blog" signature; teammates don't write like this

## Correct

```php
// ✅ Names encode meaning, not shape

public function totalsFor(array $items, int $userId): array
{
    return array_map(fn (Item $item) => $item->price, $items);
}
```

```typescript
// ✅ Domain words, types in the signature
async function fetchUser(userId: string): Promise<User> {
  const { data } = await api.fetch(userId);
  return data;
}
```

**Why it reads human:**
- Each name describes the role (`items`, `userId`, `total`) not the shape
- Type info is in the type system, where refactors update it automatically
- If `items` becomes a `Collection<Item>`, the variable name still reads correctly

## When type-in-name IS okay

Conventions where the type is part of the meaning:

- **`*Id`** suffix — `userId` (distinguishes from `user` which is the whole object) ✓
- **`*Count`** suffix — `userCount` ✓
- **`is*`** / **`has*`** / **`can*`** booleans — `isPaid`, `hasShipping`, `canEdit` ✓
- **Maps with two-type names** — `usersByEmail` (carries both types in one word) ✓
- **DTO classes** — `UserResponse`, `OrderPayload` (clarifies "this is the wire form, not the domain model") ✓

The suspect patterns are `*Object`, `*Array`, `*List`, `*Map`, `*String`, `*Integer`, `*Float` on local variables.

## Detection

```bash
# Suspect type-suffix names on variables (PHP)
grep -rEn '\$[a-z][a-zA-Z]*?(Object|Array|List|Map|String|Integer|Float|Boolean|Bool)\b' \
  --include='*.php' app/

# TS / JS
grep -rEn '\b(const|let)\s+[a-z][a-zA-Z]*?(Object|Array|List|Map|String|Boolean)\b' \
  --include='*.ts' --include='*.tsx' --include='*.js' --include='*.jsx' src/
```

A few hits are fine (especially in test data setups). **Density across multiple files** is the signal — a project with 30+ `*Array` / `*Object` variables almost certainly came from a model.

Reference: [Clean Code — Chapter 2](https://www.oreilly.com/library/view/clean-code/9780136083238/) · [Joel Spolsky — Making Wrong Code Look Wrong](https://www.joelonsoftware.com/2005/05/11/making-wrong-code-look-wrong/) (the "useful Hungarian" original — different from this rule's anti-pattern) · Internal: [`naming-over-descriptive`](naming-over-descriptive.md)

---


## Premature Interface (One Implementation, No Plan for a Second)

**Impact: HIGH (Adds indirection nobody asked for; doubles the surface for every change)**

AI defaults to "enterprise-grade" because its training distribution is enterprise-grade. The result: every service gets an interface and one implementation, with no second implementation in the foreseeable future. The interface is pure ceremony — every change updates two files, every test mocks the interface, every reader chases two clicks to find the actual code.

**Interfaces earn their place when:**
- There IS a second implementation (e.g., `StripeGateway` and `AdyenGateway`)
- There IS a planned second implementation (in this quarter's roadmap)
- The implementation is being swapped in tests with a fake (and even then, dependency injection of the concrete type is often fine)

**Otherwise: write the concrete type. Add the interface when a second implementation actually shows up.** YAGNI is real.

## Incorrect

```php
// ❌ Interface with exactly one implementation, no plan for a second

interface UserRepositoryInterface
{
    public function find(int $id): ?User;
    public function save(User $user): void;
    public function delete(int $id): void;
}

final class UserRepository implements UserRepositoryInterface
{
    public function find(int $id): ?User { /* ... */ }
    public function save(User $user): void { /* ... */ }
    public function delete(int $id): void { /* ... */ }
}

// Binding in a service provider, mostly to satisfy the interface
$this->app->bind(UserRepositoryInterface::class, UserRepository::class);
```

```typescript
// ❌ Same TS pattern
interface IOrderService {
  placeOrder(payload: OrderPayload): Promise<Order>;
  cancelOrder(orderId: string): Promise<void>;
}

class OrderService implements IOrderService {
  async placeOrder(payload: OrderPayload): Promise<Order> { /* ... */ }
  async cancelOrder(orderId: string): Promise<void> { /* ... */ }
}
```

**Why it's slop:**
- Every method declared in two files
- `find` / `save` / `delete` change — both interface AND implementation must be updated; nothing prevents drift
- Reader hits `UserRepositoryInterface` in a controller, has to navigate through to find the actual code
- The `I*` prefix or `*Interface` suffix is itself a tell — clean naming uses domain words without the marker

## Correct

```php
// ✅ Concrete class until you need a second implementation

final class UserRepository
{
    public function find(int $id): ?User { /* ... */ }
    public function save(User $user): void { /* ... */ }
    public function delete(int $id): void { /* ... */ }
}

// Controller depends on the concrete class — Laravel auto-resolves it
public function show(int $id, UserRepository $users): UserResource
{
    $user = $users->find($id) ?? throw new NotFoundException;
    return new UserResource($user);
}
```

```typescript
// ✅ Concrete class
class OrderService {
  async placeOrder(payload: OrderPayload): Promise<Order> { /* ... */ }
  async cancelOrder(orderId: string): Promise<void> { /* ... */ }
}
```

**When a second implementation arrives** — e.g., you need a `CachedUserRepository` — *then* extract the interface. By that point you'll know what shape it should have because you have two concrete examples.

**Why it reads human:**
- Concrete class is reachable in one click
- No double-maintenance
- Adding a second implementation is a deliberate refactor, not a default scaffold

## Tests: don't create an interface just to mock

If you need to swap the dependency in tests, prefer:

1. **Inject the concrete class; instantiate with fakes** (e.g., `UserRepository` with an in-memory `Database` connection)
2. **Use Laravel's container-bound fake** (`$this->mock(UserRepository::class, ...)`) — no interface needed
3. **Method-level mocks** via PHPUnit / Vitest

An interface created *only* to mock the dependency in tests is also slop. Real interfaces serve production code paths.

## Detection

```bash
# Interfaces in the codebase
grep -rn '^interface\s\|^export interface' --include='*.php' --include='*.ts' app/ src/ \
  | wc -l

# For each interface, count implementations — flag interfaces with exactly 1
# (manual review with `phpstorm` or VSCode Go to Implementation)

# PHP heuristic: interfaces with 'Interface' suffix or 'I*' prefix
grep -rEn '^interface\s+(I[A-Z]|[A-Z][a-zA-Z]+Interface)\b' --include='*.php' app/

# TS heuristic
grep -rEn '^export interface I[A-Z]|^interface I[A-Z]' --include='*.ts' --include='*.tsx' src/
```

Reference: [Martin Fowler — YAGNI](https://martinfowler.com/bliki/Yagni.html) · [Sandi Metz — The Wrong Abstraction](https://sandimetz.com/blog/2016/1/20/the-wrong-abstraction) · Internal: [`over-eng-useless-wrapper`](over-eng-useless-wrapper.md), [`naming-suffix-abuse`](naming-suffix-abuse.md)

---


## Single-Method Class That Should Be a Function

**Impact: HIGH (Wraps a free function in class ceremony for no reason)**

A class with exactly one public method, no state, and no dependencies should usually be a function. AI generates these because its training data is enterprise Java/C# where classes are mandatory for everything. PHP and TypeScript both support free functions / static methods / single-purpose action classes — the wrapping ceremony is pure slop.

The exception: **invokable action classes** (Laravel convention) using `__invoke()` are legitimate when they have constructor-injected dependencies and represent a named domain action. The slop variant is the class with no constructor, no state, one static method, and no clear domain identity.

## Incorrect

```php
// ❌ Class wrapping a single pure function

class StringFormatter
{
    public static function slugify(string $input): string
    {
        return Str::slug($input);
    }
}

class EmailValidator
{
    public function isValid(string $email): bool
    {
        return filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
    }
}

class TimestampHelper
{
    public static function toIso(DateTime $dt): string
    {
        return $dt->format(DateTimeInterface::ATOM);
    }
}
```

```typescript
// ❌ Same in TS
class StringUtil {
  static capitalize(s: string): string {
    return s.charAt(0).toUpperCase() + s.slice(1);
  }
}

class ResponseFormatter {
  static format(data: unknown): ApiResponse {
    return { data, timestamp: Date.now() };
  }
}
```

**Why it's slop:**
- Callers write `StringFormatter::slugify(...)` instead of `slugify(...)`
- `new EmailValidator()->isValid($email)` is six tokens for what should be one
- The class adds no encapsulation (no state to encapsulate)
- Importing/autoloading the class wastes bytes for zero benefit
- Pattern is recognisably "Java port" — PHP and TS have first-class functions

## Correct

```php
// ✅ Free function or method on the existing domain type

// helpers.php (or composer autoload "files")
function slugify(string $input): string
{
    return Str::slug($input);
}

function isValidEmail(string $email): bool
{
    return filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
}

// Or extension methods on Carbon
class CustomCarbon extends Carbon
{
    public function toIso(): string { return $this->format(DateTimeInterface::ATOM); }
}
```

```typescript
// ✅ Free exported functions
export function capitalize(s: string): string {
  return s.charAt(0).toUpperCase() + s.slice(1);
}

export function formatResponse<T>(data: T): ApiResponse<T> {
  return { data, timestamp: Date.now() };
}
```

**Why it reads human:**
- Caller writes `slugify(input)` — one token, no ceremony
- No autoloader hit, no class instantiation, no dependency to mock in tests
- Functions are testable directly with no setup

## Single-method class is OK when…

These are legitimate, NOT slop:

```php
// ✅ Invokable action with injected dependencies — Laravel idiom
final class PlaceOrder
{
    public function __construct(
        private PaymentGateway $payments,
        private InventoryService $inventory,
    ) {}

    public function __invoke(OrderRequest $request): Order { /* ... */ }
}

// ✅ Job / command — meant to be queued
final class SendWeeklyDigest implements ShouldQueue
{
    public function handle(MailerService $mailer): void { /* ... */ }
}

// ✅ Form Request — Laravel pattern
final class StoreUserRequest extends FormRequest { /* ... */ }
```

The test: **does it have state, dependencies, or a domain identity beyond "I wrap a function"?** If yes, it's a class. If no, it should be a function.

## Detection

```bash
# PHP — classes with exactly one public method and no constructor injection
# (rough heuristic: file has 'class X' + exactly one 'public function')
for f in $(find app/ -name '*.php'); do
  PUBLIC=$(grep -cE '^\s+public function ' "$f")
  CTOR=$(grep -cE '^\s+public function __construct' "$f")
  PROPS=$(grep -cE '^\s+(private|protected) (readonly )?[a-zA-Z]+ \$' "$f")
  if [ "$PUBLIC" = "1" ] && [ "$CTOR" = "0" ] && [ "$PROPS" = "0" ]; then
    echo "SUSPECT (single-method, no state): $f"
  fi
done

# TS — classes with one method and no constructor or fields
grep -rln 'class\s\+[A-Z]' --include='*.ts' src/ | while read f; do
  METHODS=$(grep -cE '^\s+(public\s+|private\s+|protected\s+)?[a-zA-Z]+\s*\(' "$f")
  CTOR=$(grep -c 'constructor' "$f")
  if [ "$METHODS" = "1" ] && [ "$CTOR" = "0" ]; then
    echo "SUSPECT: $f"
  fi
done
```

Reference: Internal: [`naming-suffix-abuse`](naming-suffix-abuse.md), [`over-eng-useless-wrapper`](over-eng-useless-wrapper.md)

---


## Useless Wrapper Functions

**Impact: HIGH (Adds indirection layer for no behavioural gain; callers chase one extra hop to find real code)**

A function that exists only to delegate to another function, with no added validation, transformation, or context. AI generates these because it pattern-matches on "wrap external dependencies for testability" but applies it indiscriminately.

A wrapper earns its place when it adds **something** the underlying call doesn't: validation, default arguments, error normalisation, logging, retry logic, type narrowing. A wrapper that just forwards arguments is pure indirection.

## Incorrect

```php
// ❌ Wrappers that just delegate

class UserService
{
    public function __construct(private UserRepository $repo) {}

    public function findUser(int $id): ?User
    {
        return $this->repo->find($id);
    }

    public function deleteUser(int $id): void
    {
        $this->repo->delete($id);
    }

    public function createUser(array $data): User
    {
        return $this->repo->create($data);
    }
}
```

```typescript
// ❌ Same TS pattern
class StripeWrapper {
  private stripe: Stripe;
  constructor(stripe: Stripe) { this.stripe = stripe; }

  charge(amount: number, token: string) {
    return this.stripe.charges.create({ amount, source: token });
  }

  refund(chargeId: string) {
    return this.stripe.refunds.create({ charge: chargeId });
  }
}
```

**Why it's slop:**
- `UserService::findUser($id)` does exactly what `$repo->find($id)` does
- Every caller now goes through TWO layers (`Service → Repo`) to find the actual logic
- Adding a parameter requires touching both files
- Tests of `UserService` mostly verify "did we call the repo correctly?" — they don't verify business behaviour

## Correct

```php
// ✅ Drop the wrapper; use the underlying type directly

// Controller depends on the Repository or Model directly
public function show(int $id, UserRepository $users): UserResource
{
    return new UserResource($users->findOrFail($id));
}
```

## When a wrapper IS worth keeping

A wrapper earns its place when it adds something real:

```php
// ✅ Wrapper that normalises errors + adds retry — real value
final class ResilientStripeGateway
{
    public function __construct(private Stripe $stripe) {}

    public function charge(Money $amount, string $token): Charge
    {
        return retry(times: 3, sleepMs: 200, callback: function () use ($amount, $token) {
            try {
                return $this->stripe->charges->create([
                    'amount' => $amount->cents(),
                    'currency' => $amount->currency()->code(),
                    'source' => $token,
                ]);
            } catch (CardException $e) {
                throw new PaymentDeclined($e->getMessage(), $e);   // domain exception
            }
        });
    }
}
```

This wrapper adds:
- Retry on transient failures
- Domain-specific exception (`PaymentDeclined`, not `CardException`)
- Type-safe `Money` input rather than raw cents + currency strings

That's worth the file.

**Why it reads human:**
- Wrappers exist when they add value; non-wrappers don't pad the layer count
- Caller can see "this charge call is resilient" because the type is `ResilientStripeGateway`
- No mystery hops to chase

## Detection

```bash
# Heuristic: methods that are one-liners and just call $this->dependency->method($args)
# PHP — find public methods whose body is exactly one delegate call
grep -rEn -A1 '^\s+public function [a-zA-Z]+\([^)]*\)' --include='*.php' app/ | \
  awk '/public function/{name=$0; next} /^\s+return \$this->[a-zA-Z]+->[a-zA-Z]+\(/{print name; print $0; print "---"}'

# TS — same heuristic
grep -rEn -A1 '^\s+(public |async )?[a-zA-Z]+\([^)]*\):' --include='*.ts' src/ | \
  awk '/[a-zA-Z]+\(/{name=$0; next} /return this\.[a-zA-Z]+\./{print name; print $0; print "---"}'
```

The cleanest signal: **call-sites**. If a wrapper is called from exactly one place, it's almost certainly slop. Inline it.

```bash
# For each public method in Service.php, count call sites
# (rough — use phpstorm 'Find Usages' for accurate results)
```

Reference: [Sandi Metz — The Wrong Abstraction](https://sandimetz.com/blog/2016/1/20/the-wrong-abstraction) · [Martin Fowler — Inline Function](https://refactoring.com/catalog/inlineFunction.html) · Internal: [`over-eng-single-method-class`](over-eng-single-method-class.md), [`over-eng-premature-interface`](over-eng-premature-interface.md)

---


## Dependency Creep — New Library When Existing One Suffices

**Impact: HIGH (Inflates bundle/install size, adds CVE surface, signals model picked training-data favourites)**

AI tends to introduce a new dependency whenever a problem matches a library it has seen in training, even when the project already includes a dependency that solves the same problem. Example signals:

- Adding `date-fns` to a project that already uses `dayjs`
- Adding `axios` to a project that already imports `fetch` everywhere
- Adding `lodash` to a project that already has `lodash-es` (or uses native ES methods)
- Adding `uuid` when the project already uses `nanoid`
- Adding `winston` when the app uses `pino`
- Adding `joi` / `yup` to a TS project that already uses `zod`
- Adding `bcrypt` (the npm package) in a Laravel project where `Hash::make` handles it
- Adding `moment` in a project that just removed `moment` last quarter

Each new dependency:
- Adds to install / bundle size
- Adds a new CVE surface (every `npm audit` lights up)
- Forces the team to maintain TWO libraries for the same concern, indefinitely
- Drifts as the LLM picks whichever was popular in the training-data slice it sampled from

## Incorrect

```json
// ❌ package.json — two libs doing the same job
{
  "dependencies": {
    "dayjs": "^1.11.10",            // already in use
    "date-fns": "^3.0.0",           // ADDED — same purpose
    "axios": "^1.6.0",              // ADDED — but app uses fetch everywhere
    "lodash": "^4.17.21",           // ADDED — lodash-es already present
    "lodash-es": "^4.17.21"
  }
}
```

```php
// ❌ composer.json — two HTTP clients
{
  "require": {
    "guzzlehttp/guzzle": "^7.8",
    "symfony/http-client": "^7.0"          // ADDED — same purpose
  }
}
```

**Why it's slop:**
- Both libraries get pulled into every install
- New `dayjs` code still gets written alongside the new `date-fns` code — drift forever
- Two CVE feeds to track
- Reviewer didn't notice because each PR seems reasonable in isolation

## Correct

```bash
# ✅ Use the existing dep; reject the PR adding the new one with a note

# Before merging a PR that adds a new dependency, ask:
#  1. Does an existing dependency in package.json/composer.json already do this?
#  2. Does a native browser/Node/PHP API already do this?
#  3. Is the new dep > 50KB gzipped (or > 1MB unpacked) for a single function call?
#  4. Does it have CVEs / abandoned upstream?
#
# If any answer is "yes" or "maybe", reject and use the existing tool.
```

Common collisions and the canonical choice:

| Concern | Pick one |
|---|---|
| Date / time | `dayjs` OR `date-fns` (not both) |
| HTTP client | `fetch` OR `axios` (not both) |
| UUID | `uuid` OR `nanoid` (not both) |
| Schema validation (TS) | `zod` (preferred) — reject `joi`/`yup`/`ajv` additions |
| HTTP client (PHP) | `guzzlehttp/guzzle` (Laravel default) — reject `symfony/http-client` unless project-wide |
| Logging | `monolog` (Laravel default) for PHP; pick one of `pino`/`winston` for Node |
| Form validation (Laravel) | Built-in `FormRequest` — reject standalone validation libs |

## Detection

```bash
# Node: dependency-list audit
node -e "
const pkg = require('./package.json');
const all = { ...pkg.dependencies, ...pkg.devDependencies };
const dupes = [
  ['dayjs', 'date-fns', 'moment', 'luxon'],
  ['axios', 'node-fetch', 'got', 'superagent'],
  ['uuid', 'nanoid', 'cuid'],
  ['lodash', 'lodash-es', 'ramda'],
  ['joi', 'yup', 'zod', 'ajv'],
];
for (const group of dupes) {
  const found = group.filter(p => all[p]);
  if (found.length > 1) console.log('OVERLAP:', found.join(', '));
}
"

# PHP: check for HTTP-client overlap
grep -E '\"(guzzlehttp/guzzle|symfony/http-client|kriswallsmith/buzz)\"' composer.json
```

When a new dep lands in a PR, the PR author should justify why the existing options don't fit. "AI suggested this" is not a justification.

Reference: [`technical-debt`'s `deps-unused-deps`](../technical-debt/rules/deps-unused-deps.md) (the broader audit) · [BundlePhobia](https://bundlephobia.com/) (size impact for npm packages)

---


## Generic catch Blocks That Don't Distinguish Errors

**Impact: CRITICAL (82% of AI PRs per OX Security study; masks bugs as 'handled', breaks observability, kills debuggability)**

The single most-cited AI anti-pattern in 2025 research. Try/catch wrapped around code, with a generic catch that logs and swallows. Looks responsible. Is the opposite — it hides bugs and converts loud failures into silent corruption.

Real error handling distinguishes:
- **What can throw** (specific exception types)
- **What's recoverable** (catch, retry, fall back)
- **What's a bug** (re-throw, let it propagate)

A bare `catch (e) { console.error(e); }` does none of this. It says "if anything goes wrong, I'll keep going" — and "anything" includes programmer errors that should crash so they're noticed.

## Incorrect

```php
// ❌ Generic catch that swallows everything

public function processPayment(Order $order): bool
{
    try {
        $charge = $this->stripe->charges->create([
            'amount'   => $order->total->cents(),
            'currency' => 'usd',
            'source'   => $order->paymentToken,
        ]);
        $order->update(['stripe_charge_id' => $charge->id, 'status' => 'paid']);
        return true;
    } catch (Exception $e) {
        Log::error('Payment failed: ' . $e->getMessage());
        return false;
    }
}
```

```typescript
// ❌ Same TS pattern
async function exportUsers(): Promise<User[]> {
  try {
    const users = await api.fetchAllUsers();
    return users;
  } catch (e) {
    console.error('Export failed:', e);
    return [];
  }
}
```

**Why it's slop:**
- `Exception $e` catches *everything*, including `TypeError` (programmer bug) and `OutOfMemoryError` (system crash) — those should not be "handled" by logging and continuing
- Returning `false` / `[]` lets the caller think the operation succeeded with no data, masking a real failure
- `Log::error` lacks context — what was the order? what user? what amount? Future debugger has no anchor
- A real Stripe error (card declined) gets the same treatment as a typo in the code

## Correct

```php
// ✅ Catch specifically; re-throw what you don't understand

public function processPayment(Order $order): void
{
    try {
        $charge = $this->stripe->charges->create([
            'amount'   => $order->total->cents(),
            'currency' => 'usd',
            'source'   => $order->paymentToken,
        ]);
        $order->update(['stripe_charge_id' => $charge->id, 'status' => 'paid']);
    } catch (CardException $e) {
        // Customer-facing failure — known, expected
        $order->update(['status' => 'declined', 'decline_reason' => $e->getStripeCode()]);
        throw new PaymentDeclined($e->getStripeCode(), previous: $e);
    } catch (RateLimitException $e) {
        // Transient — caller retries
        throw new TransientPaymentFailure(retryAfterSeconds: 30, previous: $e);
    }
    // Any other exception propagates: TypeError, OutOfMemoryError, etc.
    // — those are bugs we WANT to know about.
}
```

```typescript
// ✅ Specific errors, propagate the unknown
async function exportUsers(): Promise<User[]> {
  try {
    return await api.fetchAllUsers();
  } catch (e) {
    if (e instanceof AbortError) {
      // Caller aborted; that's a feature, not a failure
      throw e;
    }
    if (e instanceof RateLimitError) {
      await sleep(e.retryAfterMs);
      return api.fetchAllUsers();  // one retry
    }
    // Network / parse / bug — propagate; don't return [] (the caller would think it succeeded with no users)
    throw e;
  }
}
```

**Why it reads human:**
- Each `catch` branch handles a specific, named condition with a specific remediation
- Unknown errors propagate — the system fails loud, observability catches them, on-call wakes up before customers do
- No silent "return false" / "return empty array" — caller can't accidentally treat failure as empty success

## The "what would I want at 2am?" test

When debugging a production incident, you want:
- **Errors at the right loudness** — bugs crash with stack traces; expected failures have domain-specific exceptions you can grep
- **Context, not just the message** — order ID, user ID, request ID, all in the log line
- **No silent corruption** — a "successful" call that returned no data is the worst kind of failure

The generic-catch pattern fails all three.

## Detection

```bash
# Bare catches in PHP
grep -rEn 'catch\s*\(\s*\\?Exception\b' --include='*.php' app/

# Bare catches in TS / JS
grep -rEn '\}\s*catch\s*\(\s*[a-zA-Z_]+\s*\)\s*\{' --include='*.ts' --include='*.tsx' --include='*.js' src/

# Catch blocks that only log and continue
grep -rEnB1 'console\.error\(.*\)\s*;\s*$' --include='*.ts' --include='*.tsx' src/

# PHP equivalent
grep -rEnB1 'Log::(error|warning)\(.*\)' --include='*.php' app/Http/ app/Services/

# Linter rules
# ESLint: no-empty (already standard); add @typescript-eslint/no-explicit-any
# PHPStan level >= 8 flags catch(\Throwable) in some configurations
```

A small handful of generic catches is fine (e.g., at HTTP boundaries with structured logging). **Density** is the signal — > 5 generic catches in a single service is the AI fingerprint.

Reference: Internal: [`defensive-impossible-null`](defensive-impossible-null.md), [`defensive-missing-real`](defensive-missing-real.md)

---


## Null Checks for Impossible Nulls

**Impact: HIGH (Defensive code in places that can't fail; clutters the path and signals model didn't trust the type system)**

When the type system or surrounding code already guarantees a value is non-null, a defensive null check adds noise and tells the reader "the author wasn't sure". Common cases:

- A non-nullable parameter (typed `User $user`, not `?User $user`) — can't be null
- After `Model::findOrFail()` — never returns null (throws on miss)
- After a non-null assertion in the previous line
- Inside a `forEach` / `map` loop where the iterable can't contain null

The pattern signals the model defaulted to "check everything for safety" without reading the types around it.

## Incorrect

```php
// ❌ Null check after findOrFail — findOrFail throws if not found

public function show(int $id): View
{
    $user = User::findOrFail($id);

    if ($user === null) {
        abort(404);
    }

    return view('users.show', compact('user'));
}

// ❌ Null check on a non-nullable parameter

public function notify(User $user, string $message): void  // $user is required
{
    if ($user === null) {
        return;
    }
    Mail::to($user)->send(new GenericNotification($message));
}

// ❌ Null check repeated in same function
public function process(Order $order): void
{
    if ($order === null) return;
    $items = $order->items;
    if ($order === null) return;     // already checked above; flow can't make it null
    foreach ($items as $item) {
        if ($order === null) return; // still impossible
        // ...
    }
}
```

```typescript
// ❌ TS: defensive checks the compiler already enforced

function greet(user: User): string {       // `user: User` — non-null in type system
  if (!user) return '';                    // dead branch; TS would error
  return `Hi, ${user.name}`;
}

// ❌ Defensive after a guard
function process(user: User | null): void {
  if (!user) return;                       // narrows to User
  if (user) {                              // always true after the guard
    doStuff(user);
  }
}

// ❌ Optional chaining on a definitely-defined value
const userId = currentUser.id;             // currentUser is non-nullable here
console.log(currentUser?.email);           // ?. is unnecessary
```

**Why it's slop:**
- The dead branch is dead — never executes; reviewers must mentally rule it out
- Signals the author didn't trust the type system
- `findOrFail` is well-known Laravel API; checking its result for null says "I read the docs but don't believe them"
- Stacking redundant guards in one function (the third example) is unmistakable

## Correct

```php
// ✅ Trust findOrFail — it throws, doesn't return null

public function show(int $id): View
{
    return view('users.show', [
        'user' => User::findOrFail($id),
    ]);
}

// ✅ Non-nullable parameter — no check needed
public function notify(User $user, string $message): void
{
    Mail::to($user)->send(new GenericNotification($message));
}
```

```typescript
// ✅ Trust the types
function greet(user: User): string {
  return `Hi, ${user.name}`;
}

// ✅ Single guard; TS narrows the type below
function process(user: User | null): void {
  if (!user) return;
  doStuff(user);   // TS knows user is User here
}

// ✅ No optional chaining when not optional
console.log(currentUser.email);
```

**Why it reads human:**
- Each line carries weight; no dead branches
- Type system does the work, not runtime checks
- Reader can scan past the function without ruling out impossible paths

## When null checks ARE warranted

- **At trust boundaries** — input from HTTP/queue/file, before the type narrows
- **On nullable returns** from third-party libraries (e.g., `Eloquent::find()` returns `?Model`)
- **In `try {} catch` flows** where the value might be partially constructed
- **When the type really IS nullable** — `?User $user` parameters that the caller can leave empty

The test: **does the type say it could be null?** If yes, check. If no, trust.

## Detection

```bash
# Null checks immediately after findOrFail in PHP — a recognisable AI signature
# (Two-pass: find files with findOrFail, then look for null checks inside them.)
grep -rl 'findOrFail' --include='*.php' app/ | xargs grep -EnB1 'if\s*\(\s*\$[a-zA-Z_]+\s*===?\s*null\s*\)' 2>/dev/null

# TS: optional chaining on values typed as non-null
# (best caught via TS strict mode: noUncheckedIndexedAccess + strictNullChecks)
npx tsc --noEmit --strict

# PHPStan level 8+ catches many "always-false condition" cases
vendor/bin/phpstan analyse --level=9
```

**PHPStan level 9+ (level 10 is max in PHPStan 2.0) and TypeScript strict mode are your best mechanical defenses** — both will flag "condition is always false" or "value is never null". Adopt them and most of these impossible-null checks disappear from new code automatically.

Reference: [PHPStan rule levels](https://phpstan.org/user-guide/rule-levels) · [TS strict mode](https://www.typescriptlang.org/tsconfig#strict) · Internal: [`defensive-generic-catch`](defensive-generic-catch.md), [`defensive-missing-real`](defensive-missing-real.md)

---


## Defensive in the Wrong Places — Missing the Real Defenses

**Impact: CRITICAL (Code looks safe but the actual failure modes — network, queues, races — are unprotected)**

OX Security 2025: **76% of AI-assisted PRs miss timeouts on external calls**. The same PRs are full of impossible null checks and generic try/catch. The pattern is: defensive in places that don't need it; not defensive in places that do.

What AI often misses (the failure modes that actually take down production):

- **No timeouts on outbound HTTP** — a slow third-party API hangs your request indefinitely
- **No retry policy + jitter** — a transient blip cascades into a customer-visible failure
- **No idempotency keys on payment/order creation** — a network retry creates duplicate charges
- **No rate-limit checks before hitting an external API** — banned by Stripe / Shopify / etc.
- **No circuit breaker** — keeps hammering a known-down dependency, queue backs up
- **No locking / unique constraint** — race condition double-spends inventory
- **No backpressure on workers** — queue grows unbounded; OOM

These are the defenses that matter. A robust system has these; the dramatic-looking try/catch and null checks don't replace them.

## Incorrect

```php
// ❌ Defensive theatre: try/catch wraps the wrong thing; real risks unprotected

public function syncOrder(string $orderId): void
{
    try {
        // No timeout — could hang for minutes
        $response = Http::get("https://api.shopify.com/orders/{$orderId}");
        $data = $response->json();

        if ($data === null) {            // impossible — Http returns array or throws
            Log::error('Sync failed');
            return;
        }

        // No idempotency — if this retries, we create duplicates
        Order::create($data);
    } catch (Exception $e) {
        Log::error('Sync failed: ' . $e->getMessage());
    }
}
```

```typescript
// ❌ Same pattern in TS
async function syncOrder(orderId: string): Promise<void> {
  try {
    // No timeout; no retry; no backoff
    const res = await fetch(`https://api.shopify.com/orders/${orderId}`);
    const data = await res.json();

    if (!data) {       // impossible if res.json() resolved
      console.error('Sync failed');
      return;
    }

    await db.orders.create({ data });
  } catch (e) {
    console.error(e);    // swallow
  }
}
```

**Why it's slop:**
- The `if ($data === null)` is dead defence; the actual failure mode is "Shopify takes 90s to respond and our request timeout hits us first"
- The try/catch swallows real failures but doesn't add the things that prevent them
- Retried calls create duplicate orders (idempotency missing)
- Looks responsible; isn't

## Correct

```php
// ✅ The real defences — timeouts, idempotency, retry with backoff, circuit-break

public function syncOrder(string $orderId): void
{
    $response = Http::timeout(10)                                  // hard timeout
        ->retry(times: 3, sleepMilliseconds: 200, when: fn ($e) =>
            $e instanceof ConnectionException || $e->getCode() === 429)
        ->get("https://api.shopify.com/orders/{$orderId}");

    $response->throw();   // throw on 4xx/5xx — let it propagate

    Order::updateOrCreate(
        ['shopify_id' => $orderId],                                // idempotency via unique key
        $response->json()
    );
}
```

```typescript
// ✅ TS with AbortController for timeout + retry policy
async function syncOrder(orderId: string): Promise<void> {
  const data = await fetchWithRetry(
    `https://api.shopify.com/orders/${orderId}`,
    { timeoutMs: 10_000, retries: 3, retryOnStatus: [429, 502, 503, 504] },
  );

  // Idempotent upsert by external id (unique index on shopify_id)
  await db.orders.upsert({
    where: { shopify_id: orderId },
    create: data,
    update: data,
  });
}
```

**Why it reads human:**
- The actual failure modes (slow third party, transient errors, duplicate retries) are each addressed
- Errors NOT covered by retry propagate — observability/alerts catch them
- The idempotency key (`shopify_id`) prevents duplicate orders even under retry storms
- The "try/catch" is gone — it added no value here

## The real-defences checklist

For any code that talks to the outside world, ask:

- [ ] **Timeout** on every outbound call (HTTP, DB, queue, cache)?
- [ ] **Retry** policy: how many, with what backoff, on which error types?
- [ ] **Idempotency**: if the call is retried, will it create duplicate side effects?
- [ ] **Rate limit** awareness: am I tracking my own request rate or relying on the upstream's error?
- [ ] **Circuit breaker** for known-down dependencies (or at least a backoff cap)?
- [ ] **Locking / unique constraints** on writes that could race?
- [ ] **Backpressure** on workers reading from a queue?
- [ ] **Real errors propagated** to observability instead of swallowed?

A try/catch that doesn't add any of these isn't a defense. It's theatre.

## Detection

```bash
# HTTP calls without explicit timeout (Laravel Http facade)
grep -rEn 'Http::get\(|Http::post\(|Http::put\(|Http::delete\(' --include='*.php' app/ \
  | grep -v 'timeout('

# fetch() calls without AbortController / signal
grep -rEn 'await fetch\(' --include='*.ts' --include='*.tsx' --include='*.js' src/ \
  | grep -vE 'signal:|AbortController'

# create() / insert() without idempotency check (rough — manual review)
grep -rEn '(Order|Charge|Payment)::create\(' --include='*.php' app/
```

A repo can pass every other rule in this skill and still ship the wrong defenses. **This rule is the one to take seriously on payment, checkout, and integration code paths.**

Reference: [Stripe — Idempotent Requests](https://docs.stripe.com/api/idempotent_requests) · [Hystrix / Resilience4j circuit breaker patterns](https://github.com/resilience4j/resilience4j) · OX Security study · Internal: [`defensive-generic-catch`](defensive-generic-catch.md)

---


## Mock-Everything Tests That Assert Nothing

**Impact: CRITICAL (Tests that pass forever; they re-encode the implementation rather than verify behaviour)**

When AI generates tests, the most common failure mode is "mock every dependency, then assert that the mocks were called". The test passes the moment it's written and passes forever — including when the underlying behaviour is silently broken. arXiv 2602.00409 (2026): coding agents produce significantly more over-mocked tests than human authors.

A real test verifies **outcome**, not **interaction**. Mocking a database to assert "create() was called with X" verifies that you wrote the implementation that calls `create()`. It doesn't verify that the customer record actually lands in the table.

## Incorrect

```typescript
// ❌ Mock-everything test that asserts mock interactions

import { describe, it, expect, vi } from 'vitest';
import { UserService } from './UserService';

describe('UserService', () => {
  it('createUser saves the user', async () => {
    const mockRepo = {
      save: vi.fn().mockResolvedValue({ id: 1, email: 'a@b.com' }),
      findByEmail: vi.fn().mockResolvedValue(null),
    };
    const mockMailer = { send: vi.fn() };
    const mockHasher = { hash: vi.fn().mockReturnValue('hashed') };

    const service = new UserService(mockRepo, mockMailer, mockHasher);
    await service.createUser({ email: 'a@b.com', password: 'x' });

    expect(mockRepo.save).toHaveBeenCalled();         // weak — passes if .save() called with anything
    expect(mockMailer.send).toHaveBeenCalled();
    expect(mockHasher.hash).toHaveBeenCalledWith('x'); // verifies you wrote .hash() — not that it's secure
  });
});
```

```php
// ❌ Same pattern in PHPUnit / Pest
test('processOrder calls payment gateway', function () {
    $payments = $this->mock(PaymentGateway::class);
    $payments->shouldReceive('charge')->once();      // verifies an interaction, not an outcome

    $service = new OrderService($payments);
    $service->process(new Order(/* ... */));
});
```

**Why it's slop:**
- The test passes even if `save()` stores the wrong fields, or `hash()` returns the input unchanged, or `charge()` skips actual payment
- It locks in the implementation's structure (every refactor breaks the test even when behaviour is fine)
- "Was the mock called" is rarely a useful assertion — what matters is "did the outcome happen"
- The test gives false confidence in coverage reports

## Correct — verify the outcome, with real dependencies where possible

```typescript
// ✅ Integration test against a real (in-memory) DB; assert what changed
import { describe, it, expect, beforeEach } from 'vitest';
import { createTestDb } from './testUtils';
import { UserService } from './UserService';

describe('UserService', () => {
  let db;
  beforeEach(async () => { db = await createTestDb(); });

  it('createUser creates a user row with hashed password and sends welcome email', async () => {
    const mailer = makeFakeMailer();   // collect-and-inspect, not a mock-with-asserts
    const service = new UserService(db.users, mailer);

    await service.createUser({ email: 'a@b.com', password: 'plaintext' });

    const stored = await db.users.findByEmail('a@b.com');
    expect(stored).toBeDefined();
    expect(stored!.email).toBe('a@b.com');
    expect(stored!.passwordHash).not.toBe('plaintext');     // hash actually happened
    expect(bcrypt.compareSync('plaintext', stored!.passwordHash)).toBe(true);
    expect(mailer.sentTo('a@b.com')).toHaveLength(1);       // outcome, not interaction
  });
});
```

```php
// ✅ Laravel: use the real database (RefreshDatabase) + fake the boundary services
test('processOrder charges customer and marks order paid', function () {
    Mail::fake();
    Http::fake([
        'api.stripe.com/*' => Http::response(['id' => 'ch_test_123', 'status' => 'succeeded'], 200),
    ]);

    $order = Order::factory()->create(['status' => 'pending', 'total' => 100_00]);

    (new OrderService(new StripeGateway()))->process($order);

    expect($order->fresh()->status)->toBe('paid');
    expect($order->fresh()->stripe_charge_id)->toBe('ch_test_123');
    Mail::assertSent(OrderConfirmation::class, fn ($m) => $m->order->is($order));
});
```

**Why it reads human:**
- Each assertion checks **what should be true after the operation** — not "did you call X"
- Refactoring the internal calls is safe — the test still passes if the outcomes hold
- Real bugs (forgotten hash, wrong field assignment, missing email) trigger the test failures
- The test reads like a customer story: "after createUser, the user exists, password is hashed, welcome email is sent"

## When mocks ARE appropriate

A few legitimate uses:

- **Cost / side-effect boundaries** — real Stripe calls (use `Http::fake()` in Laravel; mock the HTTP boundary)
- **Slow externals** — third-party APIs with rate limits (mock the HTTP boundary)
- **Time-sensitive code** — freeze the clock (Carbon test helpers, vi.setSystemTime), don't mock all of time
- **Hard-to-reproduce error paths** — force a `ConnectionException` to test the retry handler

The pattern: **mock at the network/IO boundary, not at every internal class.**

## Detection

```bash
# Tests that import the mock library more than the assertion library (rough)
grep -rEn '(vi\.fn|jest\.fn|->mock\(|->shouldReceive)' \
  --include='*.test.ts' --include='*.spec.ts' --include='*Test.php' \
  | wc -l

# Tests asserting only on mock interactions, no DB / outcome checks
# (heuristic: file has 'toHaveBeenCalled' / 'shouldReceive' but no 'expect(<repo>.find' or 'assertDatabaseHas')
for f in $(find . -name '*Test.php' -o -name '*.test.ts' 2>/dev/null); do
  has_mock=$(grep -cE 'toHaveBeenCalled|shouldReceive' "$f")
  has_outcome=$(grep -cE 'assertDatabaseHas|->fresh\(|findByEmail|toBe\(|toEqual\(' "$f")
  if [ "$has_mock" -gt 3 ] && [ "$has_outcome" -lt 1 ]; then
    echo "MOCK-ONLY: $f"
  fi
done
```

Reference: [arXiv 2602.00409 — Are Coding Agents Generating Over-Mocked Tests?](https://arxiv.org/abs/2602.00409) · Internal: [`test-mirror-implementation`](test-mirror-implementation.md), [`test-doesnt-throw`](test-doesnt-throw.md)

---


## "Doesn't Throw" Tests

**Impact: HIGH (Tests that assert nothing meaningful; pass even when the function does the wrong thing)**

A test that calls the function and then asserts the function didn't throw is barely a test. It verifies one of the weakest possible properties — "the program didn't crash" — and gives false confidence in coverage. AI often defaults to this pattern because it's the easiest way to make a test pass.

A real test verifies a specific outcome. "Didn't throw" is an outcome only in the narrowest cases (you specifically want to verify a thrown error from a previous bug is now absent).

## Incorrect

```typescript
// ❌ Tests that just verify "the call completes"

describe('OrderService', () => {
  it('places an order', async () => {
    const service = new OrderService();
    await expect(service.place(makeValidOrder())).resolves.not.toThrow();
  });

  it('exports users', async () => {
    const service = new UserExportService();
    const result = await service.export();
    expect(result).toBeDefined();              // weak: undefined is the only failure mode caught
  });

  it('handles empty input', () => {
    const result = sum([]);
    expect(result).not.toBeNull();             // passes for 0, NaN, '', false, anything except null/undefined
  });
});
```

```php
// ❌ Same pattern in PHPUnit / Pest
test('it places an order', function () {
    $service = new OrderService();

    expect(fn () => $service->place(makeValidOrder()))->not->toThrow();
});

test('export returns something', function () {
    $result = (new UserExportService())->export();
    expect($result)->not->toBeNull();
});
```

**Why it's slop:**
- "Doesn't throw" tells you nothing about whether the right thing happened
- `place(order)` could return without throwing AND not actually place the order
- `export()` could return `undefined` (the test fails) OR `null` (test fails) OR an empty array, a misleading "true", etc. — the test catches only the narrowest failure
- `not.toBeNull()` passes for `0`, `''`, `NaN`, `false` — all of which are usually bugs

## Correct — assert what should happen

```typescript
// ✅ Specific outcomes

describe('OrderService', () => {
  it('place(order) persists the order and returns the saved row with status=pending', async () => {
    const service = new OrderService(db);
    const order = makeValidOrder({ total: 100_00 });

    const saved = await service.place(order);

    expect(saved.id).toBeDefined();
    expect(saved.status).toBe('pending');
    expect(saved.total).toBe(100_00);
    expect(await db.orders.findById(saved.id)).toMatchObject({ status: 'pending' });
  });

  it('exports all active users to CSV', async () => {
    await db.users.insertMany([
      { email: 'a@b.com', active: true },
      { email: 'c@d.com', active: false },
      { email: 'e@f.com', active: true },
    ]);

    const csv = await new UserExportService(db).export();

    expect(csv).toContain('a@b.com');
    expect(csv).toContain('e@f.com');
    expect(csv).not.toContain('c@d.com');     // inactive should be excluded
  });

  it('sum([]) returns 0', () => {
    expect(sum([])).toBe(0);
  });
});
```

**Why it reads human:**
- Each test verifies a specific, named outcome
- Bugs are detected: changing `status: 'pending'` to `'paid'` would fail the test; returning all users (not just active) would fail; `sum([])` returning `NaN` would fail
- Test names describe the *behaviour*, not the *function called*

## When "doesn't throw" IS legitimate

Rarely:
- **Smoke tests** for a never-throws contract on a public API (one or two tests; not the bulk of the suite)
- **Regression tests** for a specific previously-thrown error: "after fix #123, calling X with Y no longer throws"

In these cases, name the test clearly: `it('does not throw on empty input — regression for #123')`.

## Detection

```bash
# Tests whose only assertion is "not.toThrow" or "not.toBeNull"
grep -rEn '\.not\.toThrow\(|\.not\.toBeNull\(|\.toBeDefined\(' \
  --include='*.test.ts' --include='*.spec.ts' --include='*.test.tsx' src/ tests/ \
  | head -30

# PHPUnit / Pest equivalents
grep -rEn '\bexpect\(.*\)->not->toThrow\(\)|->assertNotNull\(\$result\)' \
  --include='*Test.php' --include='*.test.php' tests/

# Heuristic: test files where the ratio of weak assertions to strong assertions is high
# (manual review of files with > 5 weak assertions)
```

A few weak assertions are fine. A test suite dominated by them is a slop fingerprint.

Reference: [Martin Fowler — Test Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html) · Internal: [`test-mock-everything`](test-mock-everything.md), [`test-mirror-implementation`](test-mirror-implementation.md)

---


## Tests That Mirror the Implementation

**Impact: HIGH (Tests re-encode the production code; pass because they re-implement the same logic, not because they verify behaviour)**

AI generates tests by reading the function and translating its logic into the test. If `calculateTax` does `subtotal * 0.06`, the test sets `subtotal = 100`, then says "expect result to be `100 * 0.06`". The test passes because both sides compute the same thing. It would still pass if the function silently switched to `subtotal * 0.07` — IF the AI also "updated" the test to match.

Real tests state the **expected concrete answer**, not a formula computed from the input. They verify what *should* be true, not "the function does what the function does".

## Incorrect

```typescript
// ❌ The test re-implements the function

// Production:
export function calculateTax(subtotal: number): number {
  return subtotal * 0.06;
}

// Test:
import { calculateTax } from './calculateTax';

describe('calculateTax', () => {
  it('calculates tax', () => {
    const subtotal = 100;
    expect(calculateTax(subtotal)).toBe(subtotal * 0.06);    // re-encodes the formula
  });

  it('handles zero', () => {
    expect(calculateTax(0)).toBe(0 * 0.06);                  // tautology
  });
});
```

```php
// ❌ Same pattern in PHP
public function test_it_calculates_tax(): void
{
    $subtotal = 100;
    $expected = $subtotal * 0.06;                            // re-encoded
    $this->assertEquals($expected, calculateTax($subtotal));
}
```

**Why it's slop:**
- The test passes because RHS and LHS use the same formula — you've verified `x === x`
- If the formula in production silently changes to `0.07`, but the test "uses the formula" (or AI updates the test to match), the test still passes — and the bug ships
- Both sides of the equation come from the same place, so the test catches nothing the type system doesn't

## Correct — concrete, named expected values

```typescript
// ✅ Concrete expectations
describe('calculateTax', () => {
  it('charges 6% on the subtotal', () => {
    expect(calculateTax(100)).toBe(6);
    expect(calculateTax(50)).toBe(3);
    expect(calculateTax(1.50)).toBe(0.09);
  });

  it('returns 0 for a 0 subtotal', () => {
    expect(calculateTax(0)).toBe(0);
  });

  it('rejects negative subtotals', () => {
    expect(() => calculateTax(-10)).toThrow(InvalidSubtotal);
  });
});
```

```php
// ✅ Concrete expectations
public function test_charges_six_percent_on_subtotal(): void
{
    $this->assertEquals(6.0, calculateTax(100));
    $this->assertEquals(3.0, calculateTax(50));
    $this->assertEquals(0.09, calculateTax(1.50));
}
```

**Why it reads human:**
- The expected value is *named* — `6` is what 6% of 100 is, written as the answer
- If the production formula changes to `0.07`, the test fails at `expect(calculateTax(100)).toBe(6)` because `7 !== 6`
- The bug is caught at the boundary; the test holds the spec

## A particularly dangerous variant: bug-then-regenerate

```
User: "There's a bug — calculateTax is returning the wrong value for negatives."
AI: "I'll regenerate the test for you."
```

The regenerated test happens to pass for the new buggy behaviour. The test now ratifies the bug. This pattern is documented in Larridin (2025) and is one of the most insidious failure modes.

**Rule:** when fixing a bug, the test is written FIRST (red), with concrete expected values that reflect the correct behaviour. Then the production code is changed. Then the test goes green. Never accept "regenerate the test to match the new implementation".

## Detection

This is hard to detect mechanically — the pattern is "RHS includes the same constants/operations as the production code". Useful heuristics:

```bash
# Tests where expected values use the same magic number as the source
# (extract numeric constants from src/, then grep tests for them)
grep -rEoh '[0-9]+\.[0-9]+|[0-9]+_[0-9]+' --include='*.ts' src/ | sort -u > /tmp/src-constants.txt
grep -rEoh '[0-9]+\.[0-9]+|[0-9]+_[0-9]+' --include='*.test.ts' tests/ | sort -u > /tmp/test-constants.txt
comm -12 /tmp/src-constants.txt /tmp/test-constants.txt | head
# If many magic numbers from src appear in tests, they may be re-encoding the implementation

# Mutation testing is the gold-standard detection:
# - Stryker (JS/TS): https://stryker-mutator.io/
# - Infection (PHP): https://infection.github.io/
# If a mutation in the source doesn't fail a test, the test is mirror-ware.
```

**Mutation testing is the right answer here.** Add Stryker or Infection to your CI and require a minimum mutation score on PRs touching the unit-test directory.

Reference: [Stryker Mutator](https://stryker-mutator.io/) · [Infection](https://infection.github.io/) · Internal: [`test-mock-everything`](test-mock-everything.md), [`test-doesnt-throw`](test-doesnt-throw.md)

---


## Snapshot Tests Replacing Behavioural Assertions

**Impact: HIGH (Snapshots ratify whatever the function currently returns; engineers approve diffs without reading them)**

A snapshot test calls the function, captures the output, and asserts "matches the saved snapshot". For genuinely stable, large, structural outputs (rendered React component DOM, generated SQL, large JSON shapes), snapshots are useful. For everything else — and especially when AI generates them — they're a way to make a test pass without writing a real assertion.

The failure mode is well-documented: a test fails on a real bug, the engineer runs `vitest -u` to "update snapshots", ships. The snapshot test now ratifies the bug. Repeat across a team and snapshots become noise the team has trained itself to ignore.

## Incorrect

```typescript
// ❌ Snapshot tests instead of behavioural assertions

describe('calculateRefund', () => {
  it('matches snapshot', () => {
    expect(calculateRefund(order)).toMatchSnapshot();         // what should the answer be?
  });
});

describe('formatPrice', () => {
  it('matches snapshot', () => {
    expect(formatPrice(1234.5)).toMatchSnapshot();
  });
});
```

```typescript
// ❌ Component snapshots that lock in entire DOM
describe('OrderCard', () => {
  it('renders', () => {
    const { container } = render(<OrderCard order={makeOrder()} />);
    expect(container).toMatchSnapshot();                       // 800-line snapshot file
  });
});
```

**Why it's slop:**
- "Matches snapshot" doesn't say what the answer is — readers can't tell what the test verifies
- When the snapshot fails (because the team made an intentional change), the engineer reflexively runs `-u` to update
- Snapshot files grow huge; nobody reviews them in PRs
- AI generates snapshot tests by default because it's the easiest way to "test" without writing an actual assertion

## Correct — explicit behavioural assertions

```typescript
// ✅ Concrete expectations

describe('calculateRefund', () => {
  it('refunds the full order minus the restocking fee', () => {
    const order = makeOrder({ total: 100_00, restockingFee: 5_00 });
    expect(calculateRefund(order)).toEqual({
      amount: 95_00,
      stripeChargeId: order.stripeChargeId,
    });
  });

  it('refunds zero if the order is past the return window', () => {
    const order = makeOrder({ returnsCloseAt: subDays(new Date(), 1) });
    expect(() => calculateRefund(order)).toThrow(RefundWindowClosed);
  });
});

describe('formatPrice', () => {
  it('formats USD with two decimal places and a leading $', () => {
    expect(formatPrice(1234.5)).toBe('$1,234.50');
  });
});

describe('OrderCard', () => {
  it('shows the order number, total, and status', () => {
    const order = makeOrder({ id: 'ord_123', total: 50_00, status: 'paid' });
    render(<OrderCard order={order} />);
    expect(screen.getByText('Order #ord_123')).toBeInTheDocument();
    expect(screen.getByText('$50.00')).toBeInTheDocument();
    expect(screen.getByText('Paid')).toBeInTheDocument();
  });
});
```

**Why it reads human:**
- The test reads as the spec — "refunds the full order minus the restocking fee"
- Concrete expected values; bug-then-regenerate doesn't silently work
- Component tests assert *behaviour* (specific text appears) not *DOM shape* (every class name and attribute)

## When snapshots ARE worth using

Genuine cases for snapshot testing:

- **Large generated artifacts** — SQL output from an ORM, generated migration files, OpenAPI specs
- **Structural data** — public API JSON responses where any structural change should be reviewed
- **Visual regression** — actual screenshot comparison (e.g., Playwright `toHaveScreenshot`)

For these:
- Keep the snapshot small (don't snapshot the whole HTML tree if you can snapshot the relevant attribute)
- Review the snapshot diff IN the PR (treat snapshot files as code)
- Don't `-u` reflexively; understand the change

## Detection

```bash
# Snapshot usage in the repo
grep -rEn 'toMatchSnapshot\(|toMatchInlineSnapshot\(' --include='*.test.ts' --include='*.spec.ts' --include='*.test.tsx' src/ tests/ | wc -l

# Snapshot files
find . -name '__snapshots__' -type d 2>/dev/null

# Heuristic: snapshot files larger than 200 lines = probably too big to review
find . -path '*/__snapshots__/*.snap' -type f -exec wc -l {} \; 2>/dev/null \
  | awk '$1 > 200 { print "TOO LARGE: " $2 " (" $1 " lines)" }'

# Tests that use ONLY snapshot assertions, no toBe / toEqual
for f in $(find . -name '*.test.tsx' -o -name '*.test.ts' 2>/dev/null); do
  snap=$(grep -c 'toMatchSnapshot' "$f")
  real=$(grep -cE 'toBe\(|toEqual\(|toContain\(|toBeInTheDocument' "$f")
  if [ "$snap" -gt 0 ] && [ "$real" = "0" ]; then
    echo "SNAPSHOT-ONLY: $f"
  fi
done
```

Reference: [Kent C. Dodds — Effective Snapshot Testing](https://kentcdodds.com/blog/effective-snapshot-testing) · [Vitest snapshot docs](https://vitest.dev/guide/snapshot) · Internal: [`test-mock-everything`](test-mock-everything.md), [`test-mirror-implementation`](test-mirror-implementation.md)

---


## Hyper-Consistent Formatting

**Impact: MEDIUM (A codebase where every file looks linter-perfect is suspiciously not-human)**

Real codebases have geology — older files use older conventions, busy modules have rushed sections, 2am hotfix code has different spacing than carefully-reviewed weeks. AI-generated code is **uniformly polished**: every function the same length, every blank line in the same place, every import alphabetized, no styling drift anywhere. This isn't a hallmark of quality — it's a hallmark of generation.

The fingerprint is **uniformity at scale**. One file looking pristine is fine. A 2000-line PR where every file is identically pristine, across files that should have different velocity histories, is a strong AI signal.

## What it looks like

The PR diff shows:

- Every PHP file has exactly 4-space indent, exactly one blank line between methods, exactly two blank lines between class members, every parameter aligned identically
- Every TS file has the same import-grouping pattern, named imports always alphabetised, every arrow function body wrapped identically
- No `// HACK:` / `// XXX:` / late-night comments anywhere
- No commented-out code (clean — but also no signs of someone exploring)
- Identical commit-message structure across 50 commits
- All files use the same paradigm (e.g., every function is an arrow function; every class uses constructor property promotion) even in places where the existing codebase mixed styles

## Why this matters

Code geology is a tool for the next developer:
- **Different ages tell different stories** — "this section is from 2022, before we switched to..."
- **`// HACK:` markers signal known fragility** that hasn't been worth refactoring
- **Commented-out code reveals exploration** — "we tried X, didn't work, kept the option in case"
- **Velocity differences** show which paths were rushed and might harbour bugs

A repo with no geology forces every reader to start fresh — no inherited context, no "the team didn't bother fixing this, it works fine" signal. AI-generated codebases are flat: every line treated as equally important.

## The "all green field" tell

Hyper-consistency is most suspicious in **brownfield additions** — i.e., when AI adds code to a codebase that has its own conventions. Real engineers either:

- **Match the existing style** (consistent with the surrounding 50 files)
- **Refactor the surrounding files** alongside their change (drift announces itself in the diff)
- **Introduce a deliberate change** with an explanation ("new files use the new pattern; old files updated as touched")

AI typically does the first poorly — it picks "the most recent convention" or "the most popular convention from training data" rather than matching the actual codebase. Result: the new files are linter-perfect by some standard, but inconsistent with the surrounding codebase.

## What "human formatting" looks like

You won't catch this with a linter. It's a feel:

- Some files have slightly off blank-line spacing where someone hit enter twice
- Imports sometimes group "stuff I added recently" at the top
- One file has 6-space indent inside a heredoc because the dev didn't fight prettier on it
- A function has an extra blank line before a tricky if-branch where the author paused to think
- A 2-line comment is in mid-sentence-case because someone typed it angry

These tiny artifacts are how humans write code. Their absence — especially across many files at once — is the slop.

## Detection

There's no automatic detector. The signal is human-judgment:

1. **Run the diff through `git diff --stat`** — many files changed at once is normal; many files all with the same line-count profile is suspicious
2. **`git log --author` distribution** — if the diff is from one author but covers 30 files in one commit, raise an eyebrow
3. **Compare new files to surrounding files** — does the indentation, import order, brace style match the existing convention?
4. **Search for `// HACK:` / `// XXX:` markers** in the new files — their *absence* is the signal:
   ```bash
   git diff origin/main...HEAD -- '*.ts' '*.php' | grep -E '// HACK:|// XXX:|// FIXME' || echo "NO HACK/XXX MARKERS"
   ```
5. **Mixed-paradigm check** — in a TS PR, count `function` declarations vs arrow functions; in a PHP PR, count `final class` vs `class`. A sudden swing is a tell.

## What to do

If a PR looks hyper-consistent:

- Ask the author: "Was this AI-assisted? Walk me through the section in `OrderService` line 80." Their ability to explain the *intent*, not just the code, separates "AI-written, human-reviewed" from "AI-written, accepted-as-is".
- Sample a few functions and ask: "What's the trade-off you chose here?" If they shrug, the code wasn't really written by them.

The goal isn't to ban AI assistance — it's to make sure the author owns the code they ship.

Reference: Internal: [`style-no-hack-scars`](style-no-hack-scars.md)

---


## as any / @ts-ignore Escape Hatches

**Impact: HIGH (AI sprinkles these wherever inference is hard — defeats the type system in exactly the places it earns its value)**

AI's common move when TypeScript types don't line up cleanly: cast to `any`, add `@ts-ignore`, or annotate with `: any`. The escape hatch silences the compiler — and silences the protection it was about to provide. Worst case: a real bug (wrong shape, missing field, wrong arg order) survives review because the compiler stopped warning.

A reviewer should treat every `as any` / `@ts-ignore` / `@ts-expect-error` as **a request for justification**. Sometimes legitimate (third-party library with bad types, complex generics, runtime-validated unknowns). Often not.

## Incorrect

```typescript
// ❌ as any sprinkled to silence the compiler

async function processWebhook(payload: unknown): Promise<void> {
  const event = (payload as any).event;                    // unknown → any without validation
  const orderId = (payload as any).data.order.id;          // chain of unsafe access
  await db.orders.update({ where: { id: orderId }, data: { status: 'paid' } });
}

// ❌ @ts-ignore over a real type mismatch
function calculateTotal(items: Item[]): number {
  // @ts-ignore — items.map sometimes returns undefined? not sure
  return items.map(i => i.price * i.quantity).reduce((a, b) => a + b, 0);
}

// ❌ : any in function signature
function processData(data: any) {                          // gives up on the input shape
  return data.value;
}

// ❌ Mixing ts-ignore with no comment
function send(): void {
  // @ts-ignore
  emailService.send(undefined);                            // why is undefined OK here?
}
```

**Why it's slop:**
- `as any` propagates — every subsequent access is also unchecked
- A real bug (e.g., the payload's actual shape is `payload.event.data.order_id`, not `payload.data.order.id`) doesn't fail until production
- `@ts-ignore` with no explanation is one of the most-cited AI tells in 2025 reviews
- `: any` parameters defeat every downstream type check

## Correct

```typescript
// ✅ Validate at the boundary; types flow through downstream

import { z } from 'zod';

const WebhookPayload = z.object({
  event: z.string(),
  data: z.object({
    order: z.object({ id: z.string() }),
  }),
});

async function processWebhook(payload: unknown): Promise<void> {
  const { event, data } = WebhookPayload.parse(payload);   // typed from here on
  await db.orders.update({ where: { id: data.order.id }, data: { status: 'paid' } });
}

// ✅ Fix the real type issue rather than ignore it
function calculateTotal(items: Item[]): number {
  return items.reduce((sum, i) => sum + i.price * i.quantity, 0);
}

// ✅ Type the input
function processData<T extends { value: string }>(data: T): string {
  return data.value;
}
```

## When the escape hatch IS warranted

Legitimate use is narrow:

- **Bad third-party types** — the lib's `.d.ts` is wrong; you've filed an upstream issue. Use `@ts-expect-error` (not `@ts-ignore`) with a comment naming the issue:
  ```typescript
  // @ts-expect-error: upstream types incorrectly require `id` — see https://github.com/lib/issues/420
  ```
- **Migration phase** — moving a JS file to TS gradually; mark debt and track removal
- **Complex generic constraints** where the compiler can't prove correctness but humans can — annotate with a comment explaining why it's safe

Every escape hatch in production code should have an explanatory comment. No bare `// @ts-ignore`.

## Detection

```bash
# Count escape hatches in the diff / repo
grep -rEn '\bas any\b' --include='*.ts' --include='*.tsx' src/ | wc -l
grep -rEn '@ts-ignore|@ts-nocheck' --include='*.ts' --include='*.tsx' src/ | wc -l
grep -rEn ': any\b' --include='*.ts' --include='*.tsx' src/ | wc -l

# Bare @ts-ignore / @ts-expect-error without an explanatory comment
grep -rEnA1 '@ts-(ignore|expect-error)\s*$' --include='*.ts' --include='*.tsx' src/

# ESLint rules to enforce in tsconfig / .eslintrc:
# @typescript-eslint/no-explicit-any: error
# @typescript-eslint/ban-ts-comment: { ts-ignore: "allow-with-description" }
# @typescript-eslint/no-unsafe-*: error
```

Add to ESLint config:

```json
{
  "rules": {
    "@typescript-eslint/no-explicit-any": "error",
    "@typescript-eslint/ban-ts-comment": [
      "error",
      { "ts-ignore": "allow-with-description", "ts-expect-error": "allow-with-description" }
    ]
  }
}
```

**Bar:** zero `as any` / bare `@ts-ignore` in new code. Existing ones grandfathered but tracked.

Reference: [TypeScript handbook — strict mode](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html) · [typescript-eslint rules](https://typescript-eslint.io/rules/) · [Zod](https://zod.dev/) · Internal: [`defensive-impossible-null`](defensive-impossible-null.md)

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

---


## Debug Artifacts Left in Production Code

**Impact: HIGH (console.log, dd(), dump(), var_dump — AI's exploratory leftovers ship to production)**

`console.log("here")`, `console.log("got user", user)`, `dd($order)`, `dump($result)`, `var_dump($payload)`, `print_r($data)`, `echo $error` — these are the breadcrumbs left from when the developer (or AI) was debugging. They ship to production and:

- Leak sensitive data into stdout / log aggregators (PII, tokens)
- Bloat production logs to the point you can't grep for real signal
- `dd()` literally halts execution — if it reaches prod, your endpoint returns "1" + var_dump output instead of JSON

AI is particularly bad about this because the model tends to add `console.log("got result", x)` "to help with debugging" and rarely removes it before "finalising" the function.

## Incorrect

```typescript
// ❌ Debug artifacts left in

async function processPayment(order: Order, token: string): Promise<Charge> {
  console.log('processPayment start', order.id);                  // shipped
  console.log('token', token);                                    // SHIPS THE TOKEN
  const charge = await stripe.charges.create({ /* ... */ });
  console.log('got charge', charge);                              // shipped
  return charge;
}
```

```php
// ❌ Same in PHP
public function processWebhook(Request $request): JsonResponse
{
    $payload = $request->json()->all();
    dd($payload);                                                 // halts execution; returns a debug page
    // …rest never runs
}

public function calculateTax(Order $order): Money
{
    dump($order);                                                 // prints to stdout in production
    print_r($order->items);
    $taxRate = 0.06;
    var_dump($taxRate);
    return $order->subtotal->multiplied($taxRate);
}
```

**Why it's slop:**
- `dd()` in a controller is an outage — the request never completes
- `console.log('token', token)` is a credentials leak; on serverless logs, every Stripe call ships the token to CloudWatch
- `dump()` / `var_dump()` show up in HTTP responses if not in a CLI context (especially during `artisan tinker` or test failures)
- A repo with 50+ stray `console.log` in production paths signals nobody is reading their own code before merging

## Correct

```typescript
// ✅ No debug; if logging matters, use the proper logger with context
import { logger } from '@/lib/logger';

async function processPayment(order: Order, token: string): Promise<Charge> {
  // Real logger — structured, redacts secrets, levels enforced
  const log = logger.child({ orderId: order.id });
  log.info('payment.start');

  const charge = await stripe.charges.create({ /* ... */ });

  log.info('payment.success', { chargeId: charge.id, amountCents: charge.amount });
  return charge;
}
```

```php
// ✅ Structured logging at the boundary; no debug() calls

public function processWebhook(Request $request): JsonResponse
{
    Log::withContext([
        'webhook_id' => $request->header('Stripe-Webhook-Id'),
        'event_type' => $request->json('type'),
    ])->info('webhook.received');

    // … actual handling …

    return response()->json(['ok' => true]);
}
```

**Why it reads human:**
- A logger with structured fields and levels (info/warn/error) — not stdout spam
- Tokens / secrets get redacted by the logger (or not logged at all)
- The log lines are intentional, useful for production debugging, and won't break the response

## When debug calls in production code ARE warranted

Rare. Usually zero. Specific cases:

- **Logs in well-defined CLI scripts** that are *meant* to be verbose: a one-off data migration script can use `echo` / `console.log` freely
- **Test-only files** (`*.test.ts`, `*Test.php`) — fine to keep debug there during development
- **Explicit `if (DEBUG_MODE) console.log(...)`** wrapped behind a feature flag — fine, but rare in practice

Production controllers, services, jobs, listeners, middleware: **zero raw debug calls**.

## Detection

```bash
# JavaScript / TypeScript — console.log in production code
grep -rEn '\bconsole\.(log|debug|info|warn)\(' --include='*.ts' --include='*.tsx' --include='*.js' --include='*.jsx' \
  src/ resources/js/ 2>/dev/null \
  | grep -v -E '\.test\.|\.spec\.|/__tests__/|/scripts/'

# PHP — debug helpers
grep -rEn '\b(dd|dump|var_dump|print_r|var_export)\s*\(' --include='*.php' \
  app/ 2>/dev/null

# CI gate — block PRs that introduce debug artifacts
NEW_DEBUG=$(git diff --diff-filter=ACM origin/main...HEAD -- 'app/**/*.php' 'src/**/*.ts' \
  | grep -E '^\+.*\b(console\.log|dd\(|dump\(|var_dump\(|print_r\()')
test -z "$NEW_DEBUG" || { echo "Debug artifacts in PR:"; echo "$NEW_DEBUG"; exit 1; }
```

ESLint rules:

```json
{
  "rules": {
    "no-console": ["error", { "allow": ["warn", "error"] }]
  }
}
```

PHPStan + a custom rule can flag `dd`/`dump` similarly. PHP-CS-Fixer has a `no_debug_print` rule.

Reference: [Laravel Logging docs](https://laravel.com/docs/logging) · [ESLint no-console](https://eslint.org/docs/latest/rules/no-console) · [Pino structured logging](https://github.com/pinojs/pino)

---


## Trivial Boilerplate

**Impact: MEDIUM (Pattern-matched boilerplate that hides intent and inflates line count)**

The class of "code that says less than the underlying expression". Common AI variants:

- `if (x) return true; else return false;` — when `return x` does it
- `return x === true ? true : false;` — ditto
- `const isPaid: boolean = order.status === 'paid';` — TS infers `boolean` already; the annotation is noise
- `const name: string = 'asyraf';` — TS infers `string`
- `await Promise.resolve(value)` — when `value` is already non-Promise
- `try { await fn(); } catch (e) { throw e; }` — pass-through catch
- `[...array]` to "make a copy" when the next operation doesn't mutate

Each is a small AI fingerprint. A few are fine. A repo with many of them across files reads as model-generated.

## Incorrect

```typescript
// ❌ if-true-false
function isPaid(order: Order): boolean {
  if (order.status === 'paid') {
    return true;
  } else {
    return false;
  }
}

// ❌ Ternary returning the booleans it was given
function isCompleted(order: Order): boolean {
  return order.status === 'completed' ? true : false;
}

// ❌ Redundant type annotations on obvious literals
const name: string = 'asyraf';
const age: number = 30;
const isActive: boolean = true;
const orders: Order[] = await db.orders.find();   // db.orders.find() return type IS Order[]

// ❌ Pass-through catch
async function processPayment(order: Order): Promise<Charge> {
  try {
    return await stripe.charges.create({ /* ... */ });
  } catch (e) {
    throw e;                                       // catch literally does nothing
  }
}

// ❌ Unnecessary await + Promise.resolve
async function formatName(user: User): Promise<string> {
  return await Promise.resolve(`${user.first} ${user.last}`);
}

// ❌ Spread to "copy" — but nothing mutates
function totalsFor(items: Item[]): number {
  const copied = [...items];                       // unused: the next op doesn't mutate
  return copied.reduce((s, i) => s + i.price, 0);
}
```

```php
// ❌ PHP equivalents
public function isPaid(Order $order): bool
{
    if ($order->status === 'paid') {
        return true;
    } else {
        return false;
    }
}

public function processWebhook(Request $request): JsonResponse
{
    try {
        return $this->handle($request);
    } catch (\Exception $e) {
        throw $e;                                  // pass-through
    }
}
```

**Why it's slop:**
- Each line carries no information beyond the simpler form
- The `try { ... } catch (e) { throw e; }` is the most embarrassing — six tokens to do nothing
- Type annotations on inferred literals fight the type system instead of using it
- A reader's eyes have to traverse "what does this expand to?" rather than read the expression directly

## Correct

```typescript
// ✅ Direct expressions
function isPaid(order: Order): boolean {
  return order.status === 'paid';
}

function isCompleted(order: Order): boolean {
  return order.status === 'completed';
}

// ✅ Let TS infer
const name = 'asyraf';
const age = 30;
const isActive = true;
const orders = await db.orders.find();

// ✅ No catch when not handling
async function processPayment(order: Order): Promise<Charge> {
  return stripe.charges.create({ /* ... */ });
}

// ✅ No need to await a non-Promise
function formatName(user: User): string {
  return `${user.first} ${user.last}`;
}

// ✅ No spread when nothing mutates
function totalsFor(items: Item[]): number {
  return items.reduce((s, i) => s + i.price, 0);
}
```

```php
// ✅ PHP equivalents
public function isPaid(Order $order): bool
{
    return $order->status === 'paid';
}

public function processWebhook(Request $request): JsonResponse
{
    return $this->handle($request);
}
```

**Why it reads human:**
- Each expression does exactly the work; nothing extra
- Type inference does its job; the type annotations are added where they earn their place (function signatures, public APIs)
- No pass-through catch; if an error needs handling, it's handled

## When annotations / spreads ARE worth it

A few cases:

- **Public-API parameters / returns** — explicit types document the contract: `function add(a: number, b: number): number`
- **Complex inferred types** — annotate when the inferred type is hard to read
- **Boundary code** — `as const`, `satisfies`, or explicit annotations on configuration objects help downstream type narrowing
- **Spread when you actually want a copy** — before sorting, before mutating, before passing to a function that mutates

The trivial-boilerplate test: **does removing the construct change the program's behaviour or the reader's understanding?** If no, remove.

## Detection

```bash
# if-true-false
grep -rEnB1 -A2 'if\s*\([^)]+\)\s*\{?\s*$' --include='*.ts' --include='*.tsx' --include='*.js' --include='*.php' \
  src/ app/ 2>/dev/null | grep -B2 -A1 'return true\b' | grep -A2 'else' | head

# Pass-through catch
grep -rEnB0 -A1 'catch\s*\([^)]*\)\s*\{' --include='*.ts' --include='*.tsx' --include='*.php' \
  src/ app/ 2>/dev/null | grep -B1 '^\s*throw\b'

# Ternary returning booleans
grep -rEn '\?\s*true\s*:\s*false\b' --include='*.ts' --include='*.tsx' --include='*.js' --include='*.php' \
  src/ app/

# Redundant type annotations on string/number/boolean literals (TS)
grep -rEn ':\s*(string|number|boolean)\s*=\s*("[^"]*"|[0-9]+|true|false)' --include='*.ts' --include='*.tsx' src/
```

ESLint rules:

```json
{
  "rules": {
    "no-useless-return": "error",
    "no-useless-catch": "error",
    "no-unneeded-ternary": "error"
  }
}
```

Reference: [ESLint — no-useless-catch / ternary / return](https://eslint.org/docs/latest/rules/) · [PHP-CS-Fixer rules](https://cs.symfony.com/) · Internal: [`defensive-generic-catch`](defensive-generic-catch.md)

---

