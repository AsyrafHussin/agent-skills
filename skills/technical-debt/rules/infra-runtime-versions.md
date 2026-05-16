---
title: End-of-Life Runtime Versions
impact: CRITICAL
impactDescription: "EOL runtimes stop receiving security patches"
tags: runtime, eol, security
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
