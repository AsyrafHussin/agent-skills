# Code Review Security

Security review lens for vulnerabilities, input validation, data protection, auth/authz, and secure defaults.

## Overview

This skill provides:
- A repo-structured adaptation of the Pi Ensemble review lens
- Focused review guidance for security
- A standardized Must Fix / Observations / Summary output contract
- Adversarial-input discipline to avoid shallow approvals

## Categories

### 1. Injection Vulnerabilities (Critical)
SQL, command, XSS, template, and path traversal injection risks.
### 2. Authentication & Authorization (Critical)
Identity checks, session handling, access control, and token safety.
### 3. Data Protection (High)
Secrets, PII, encryption, logs, and exposure boundaries.
### 4. Input Validation (High)
Whitelisting, bounds checks, sanitization, and schema validation.
### 5. Dependency Security (Medium)
Known vulnerable packages and supply-chain exposure.
### 6. Configuration Security (High)
Hardcoded secrets, insecure defaults, debug exposure, and CORS/security headers.

## Usage

- "security review"
- "check for vulnerabilities"
- "auth review"
- "Review this diff for security issues"

## References

- [Pi Ensemble security lens](https://raw.githubusercontent.com/randomm/pi-ensemble/main/skill/code-review-security/SKILL.md)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
