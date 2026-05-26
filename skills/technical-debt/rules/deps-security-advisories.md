---
title: Known Security Advisories
impact: CRITICAL
impactDescription: "Public CVEs are pre-published attack instructions"
tags: dependencies, security, cve
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
