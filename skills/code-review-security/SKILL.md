---
name: code-review-security
description: Security-focused review lens for identifying injection risks, auth flaws, secret exposure, unsafe configuration, and missing validation during code review.
license: MIT
metadata:
  author: Agent Skills Team
  version: "1.1.0"
---

# Code Review Security

Targeted security lens for vulnerabilities, validation, data protection, auth/authz, and secure defaults.

## Review Focus

| Priority | Category | Prefix |
|----------|----------|--------|
| CRITICAL | Injection vulnerabilities | `sec-injection-vulnerabilities` |
| CRITICAL | Authentication & authorization | `sec-auth-authorization` |
| HIGH | Data protection | `sec-data-protection` |
| HIGH | Input validation | `sec-input-validation` |
| MEDIUM | Dependency security | `sec-dependency-security` |
| HIGH | Configuration security | `sec-configuration-security` |

Look for SQL/command/template/path injection, authorization gaps, unsafe session/token handling, secret or PII exposure, missing validation, vulnerable dependencies, and insecure defaults.

## Scope

Run this lens when security is the dominant concern or a workflow explicitly dispatches it. Do not broaden into a generic review bundle.

Out of scope as primary concerns: type coverage, retry hygiene without security impact, performance/architecture trade-offs, readability-only feedback.

## Handoff

When another concern becomes dominant, emit **at most one** focused handoff with the observed `path:line` and why that concern is dominant:

- `code-review-error-handling` for cleanup/retry/failure behavior weakening controls;
- `code-review-type-safety` for untrusted shape or validation proof;
- `code-review-architecture` for broken authz/trust boundaries;
- `code-review-performance` for user-triggerable cost amplification.

## Evidence Discipline

- Identify the trust boundary and concrete sink/control before claiming a vulnerability.
- When the diff changes untrusted input, authorization, persistence, or output encoding, construct one relevant adversarial input and trace validation, authorization, persistence, and output handling.
- Keep sanitization, persistence formatting, and output escaping context-specific; do not treat one as a substitute for another.
- If required trust-boundary evidence cannot be inspected, return `blocked` instead of guessing.
- Prefer secure defaults and smaller attack surface over configurable complexity.

## Terminal Contract

```text
STATUS: findings
<path>:<line>: <CRITICAL|HIGH|MEDIUM|LOW> <problem>. <concrete fix>.
HANDOFF: <code-review-* lens> <path>:<line> <why this concern is dominant>   # optional, at most one
```

```text
STATUS: clean
```

```text
STATUS: blocked
UNKNOWN: <exact missing evidence>.
```

`STATUS` is this lens' judgment only. The caller owns merge, dedupe, precedence, approval, persistence, and workflow progression.

## Severity

- **CRITICAL**: exploitable vulnerability, privilege escalation, or material data exposure.
- **HIGH**: missing control with plausible security impact.
- **MEDIUM**: defense-in-depth gap with concrete value.
- **LOW**: minor hardening or hygiene.

## References

- [Pi Ensemble security lens](https://raw.githubusercontent.com/randomm/pi-ensemble/main/skill/code-review-security/SKILL.md)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
