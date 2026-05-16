---
title: Secrets in Source Code
impact: CRITICAL
impactDescription: "Once committed, a secret must be rotated AND scrubbed — git history is forever"
tags: security, secrets, credentials
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
