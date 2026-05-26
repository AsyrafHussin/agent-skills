---
title: as any / @ts-ignore Escape Hatches
impact: HIGH
impactDescription: "AI sprinkles these wherever inference is hard — defeats the type system in exactly the places it earns its value"
tags: style, typescript, type-safety, ai-fingerprint
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
