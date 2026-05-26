---
title: SECURITY.md — Vulnerability Reporting Policy
impact: HIGH
impactDescription: "Without a clear policy, researchers go public; with one, they email you privately"
tags: essential, security, vulnerability-disclosure
---

## SECURITY.md — Vulnerability Reporting Policy

**Impact: HIGH (Without a clear policy, researchers go public; with one, they email you privately)**

A `SECURITY.md` tells security researchers how to report a vulnerability to you privately. Without it, well-intentioned researchers either open a public GitHub issue (instant CVE) or give up and never report. GitHub's "Security" tab links directly to this file.

## When required

- **Any internet-facing service** — required
- **Open-source libraries** — required from first public release
- **Internal-only projects** — optional, but useful for clarity even internally
- **Closed-source applications used by customers** — recommended

## Recommended structure

```markdown
# Security Policy

## Supported versions

We provide security updates for the following versions:

| Version | Supported |
|---------|-----------|
| 3.x     | ✅        |
| 2.x     | ✅ (until 2026-12) |
| 1.x     | ❌ (EOL)  |

## Reporting a vulnerability

**Please do not file a public GitHub issue for security vulnerabilities.**

Email us at **security@example.com** with:
- A description of the issue
- Steps to reproduce
- Affected versions (if known)
- Any proof-of-concept code (optional)

We acknowledge reports within **2 business days** and aim to issue a fix within **30 days** for high/critical severity.

If you prefer encrypted communication, our PGP key is at <https://example.com/security.asc>.

## Disclosure timeline

- Day 0: We receive your report and acknowledge
- Day 1–14: Triage, reproduce, develop fix
- Day 15–30: Release patched version, notify users
- Day 30+: Public disclosure (we'll credit you unless you prefer anonymity)

## Safe harbor

We will not pursue legal action against researchers who:
- Make a good-faith effort to follow this policy
- Avoid privacy violations, service disruption, or destruction of data
- Give us reasonable time to remediate before public disclosure
```

## Incorrect

```markdown
❌ No SECURITY.md at all on a public web app

(No mechanism for reporting; researchers either guess an email or go public.)
```

```markdown
❌ "Just open an issue"

# Security

If you find a security issue, please open a GitHub issue.
```

(This guarantees public disclosure of vulnerabilities — the worst possible default.)

## Use GitHub's built-in private reporting

GitHub supports **Private Security Advisories** — researchers can report via the Security tab without exposing the issue publicly. Enable in Settings → Security → "Private vulnerability reporting". Reference it in `SECURITY.md`:

```markdown
You may also use **GitHub's private vulnerability reporting**:
<https://github.com/org/repo/security/advisories/new>
```

## Detection

```bash
# Internet-facing? Public repo? — should have SECURITY.md
test -f SECURITY.md || test -f .github/SECURITY.md \
  || echo "MISSING SECURITY.md (required for public/internet-facing projects)"
```

Reference: [GitHub — Adding a security policy](https://docs.github.com/en/code-security/getting-started/adding-a-security-policy-to-your-repository) · [GitHub Private Vulnerability Reporting](https://docs.github.com/en/code-security/security-advisories/guidance-on-reporting-and-writing-information-about-vulnerabilities/privately-reporting-a-security-vulnerability) · [Disclose.io safe-harbor template](https://github.com/disclose/disclose)
