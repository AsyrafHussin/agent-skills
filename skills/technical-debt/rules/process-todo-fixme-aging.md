---
title: Aging TODO, FIXME, and HACK Comments
impact: MEDIUM
impactDescription: "Untracked promises that compound silently"
tags: todo, fixme, comments, process
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
