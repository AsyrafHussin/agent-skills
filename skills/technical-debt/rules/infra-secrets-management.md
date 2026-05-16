---
title: Insecure Secrets Management
impact: HIGH
impactDescription: ".env files leak, plain env vars surface in logs, no rotation = compounding risk"
tags: security, secrets, configuration
---

## Insecure Secrets Management

**Impact: HIGH (.env files leak, plain env vars surface in logs, no rotation = compounding risk)**

Even when secrets stay out of source control, ad-hoc handling — `.env` files copied between machines, plain env vars echoed into CI logs, indefinite credential lifetimes — leaves a long tail of leakage paths. This rule covers everything *outside* the repo: secret-manager adoption, rotation, scope, and audit.

## How to Detect

For each environment (dev, staging, prod), check:

1. Where are secrets stored? (encrypted secret manager vs. plain files vs. shell history)
2. Are they versioned and audit-logged?
3. Is rotation automated or manual?
4. Are CI/CD secrets scoped per repo/job, or org-wide?
5. Are workload identities (OIDC / IAM roles) used instead of long-lived keys?

```bash
# Audit any .env files committed historically
gitleaks git                                  # full history scan
git log --all --diff-filter=A --name-only | grep -E '\\.env$'

# CI: scan for echoes of secrets in logs (common when devs add `set -x`)
grep -E 'API_KEY=|TOKEN=|PASSWORD=' .github/workflows/*.yml

# Cloud: list long-lived access keys (AWS example)
aws iam list-access-keys --user-name <iam-user>   # any keys > 90 days old?
```

## Incorrect

```bash
# ❌ Anti-patterns

# 1. Plain .env file committed in history (even if removed later)
$ git log -p --all -- '.env'
... (compromised)

# 2. CI workflow echoing secrets via debugging
- run: echo "AWS_KEY=$AWS_ACCESS_KEY_ID"      # ends up in build logs

# 3. Single shared IAM access key used by all CI jobs, 4 years old, never rotated
$ aws iam list-access-keys --user-name ci-deploy
CreateDate: 2022-03-15   # 4 years; full admin permissions

# 4. Slack/Notion link sharing for "DB credentials" — anyone with the link sees the password
```

**Problems:**
- A leaked CI log exposes the entire production environment
- Long-lived credentials with broad scope = compromise = full account access
- No audit trail of who accessed what credential when

## Correct

```yaml
# ✅ GitHub Actions: OIDC to AWS — no long-lived keys needed
permissions:
  id-token: write
  contents: read
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: aws-actions/configure-aws-credentials@v6
        with:
          role-to-assume: arn:aws:iam::123456789012:role/deploy-role
          aws-region: ap-southeast-1
      # AWS calls now use short-lived session tokens scoped to this workflow
```

```bash
# ✅ Production secrets in a dedicated manager (one of these):
#   - HashiCorp Vault       (self-hosted, comprehensive)
#   - AWS Secrets Manager   (with rotation Lambdas)
#   - GCP Secret Manager
#   - Doppler / 1Password Secrets Automation (SaaS)
#
# Apps read at startup via SDK; rotation is automated.

# ✅ Local dev: .env files exist but are gitignored and contain dev-only credentials
echo '.env' >> .gitignore
```

**Benefits:**
- No long-lived credentials to leak; access is scoped to job + duration
- Audit logs record every secret access
- Rotation is automatic and predictable

## Remediation Strategy

- **Effort:**
  - **S** — gitignore .env, install pre-commit gitleaks hook
  - **M** — migrate one application from env-file → secret manager
  - **L** — full org migration to OIDC + secret manager + rotation
- **When to pay down:**
  - **NOW:** any committed .env, any 90-day-old long-lived key, any secrets in plain CI logs
  - **This quarter:** migrate CI/CD to OIDC where the cloud provider supports it (AWS, GCP, Azure all do)
  - **Then:** automated rotation for all production secrets, with audit alerts on access spikes

**Hierarchy of secret handling, from worst to best:**
1. Hardcoded in repo (CRITICAL — must rotate immediately)
2. Untracked `.env` file emailed/Slacked around (high leak risk)
3. Long-lived credentials in CI/CD secrets store
4. Short-lived credentials issued via OIDC workload identity (best)

Reference: [GitHub Actions OIDC](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments) · [AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/) · [HashiCorp Vault](https://developer.hashicorp.com/vault) · [Doppler](https://docs.doppler.com/)
