---
title: Lingering Feature Flags
impact: MEDIUM
impactDescription: "Old flags become permanent special cases — dead branches that never die"
tags: feature-flags, process, cleanup
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
