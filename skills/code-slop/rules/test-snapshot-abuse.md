---
title: Snapshot Tests Replacing Behavioural Assertions
impact: HIGH
impactDescription: "Snapshots ratify whatever the function currently returns; engineers approve diffs without reading them"
tags: testing, snapshots, ai-fingerprint
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
