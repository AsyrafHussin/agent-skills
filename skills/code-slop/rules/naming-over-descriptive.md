---
title: Over-Descriptive Run-On Names
impact: HIGH
impactDescription: "Inflated identifiers that read like English instead of code"
tags: naming, verbose, ai-fingerprint
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
