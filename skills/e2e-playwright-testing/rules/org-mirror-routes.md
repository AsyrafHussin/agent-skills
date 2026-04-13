---
title: Mirror Route Structure in Test Organization
impact: MEDIUM
impactDescription: "scalable test organization as app grows"
tags: organization, structure, routes, feature-based
---

## Mirror Route Structure in Test Organization

**Impact: MEDIUM (scalable test organization as app grows)**

Organize E2E test files to mirror your application's route groups. This makes tests discoverable and scalable as features are added.

## Incorrect

```
tests/e2e/specs/
  test1.spec.js          # What does this test?
  test2.spec.js          # No grouping
  all-tests.spec.js      # One giant file with 200 tests
  login-and-buy.spec.js  # Mixed concerns
```

**Problems:**
- Can't find the test for a specific feature
- Adding a new feature means figuring out which file to edit
- One giant file causes merge conflicts
- Mixed concerns make failures hard to diagnose

## Correct

```
tests/e2e/specs/
  auth/
    login.spec.js
    register.spec.js
  customer/                    # matches role:customer routes
    egold/
      buy.spec.js
      sell.spec.js
      redeem.spec.js
      kyc-gate.spec.js
    gold-saving/
      new-agreement.spec.js
    special-trade-in/
      new-agreement.spec.js
    semantan-point/
      convert.spec.js
    team/
      member-profile.spec.js
  admin/                       # matches role:admin routes
    customers/
      kyc-approve.spec.js
    egold/
      approve-reject.spec.js
    pos-demo/
      transaction.spec.js
  management/                  # matches role:superadmin routes
    branches/
      create-edit.spec.js
    gold-price/
      add-price.spec.js
  ui/                          # cross-cutting concerns
    dark-mode.spec.js
    responsive.spec.js
    navigation.spec.js
    global-search.spec.js
```

**Naming convention:**
- Folders match route prefixes (`/admin/*` → `admin/`)
- Files match the feature or page name
- `ui/` folder for cross-cutting concerns (theme, responsive, navigation)
- One `describe` block per file, focused on one page/feature

Reference: [Playwright Docs — Test Organization](https://playwright.dev/docs/test-configuration)
