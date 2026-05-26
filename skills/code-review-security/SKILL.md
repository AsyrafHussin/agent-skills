---
name: code-review-security
description: Security-focused review lens for identifying injection risks, auth flaws, secret exposure, unsafe configuration, and missing validation during code review. Triggers on "security review", "check for vulnerabilities", "auth review", "injection risks", and related review requests.
license: MIT
metadata:
  author: Agent Skills Team
  version: "1.0.0"
---

# Code Review Security

Security review lens for vulnerabilities, input validation, data protection, auth/authz, and secure defaults. Contains 6 rules across 6 focused categories for targeted multi-lens review work.

## When to Apply

Reference these guidelines when:
- Running a targeted review for this concern
- Merging findings from a broader multi-lens review
- Stress-testing a risky diff before approval
- Writing or refining repo-owned review instructions for this lens

## Rule Categories by Priority

| Priority | Category | Impact | Prefix |
|----------|----------|--------|--------|
| 1 | Injection Vulnerabilities | CRITICAL | `sec-injection-vulnerabilities` |
| 2 | Authentication & Authorization | CRITICAL | `sec-auth-authorization` |
| 3 | Data Protection | HIGH | `sec-data-protection` |
| 4 | Input Validation | HIGH | `sec-input-validation` |
| 5 | Dependency Security | MEDIUM | `sec-dependency-security` |
| 6 | Configuration Security | HIGH | `sec-configuration-security` |

## Quick Reference

- `sec-injection-vulnerabilities` - SQL, command, XSS, template, and path traversal injection risks
- `sec-auth-authorization` - Identity checks, session handling, access control, and token safety
- `sec-data-protection` - Secrets, PII, encryption, logs, and exposure boundaries
- `sec-input-validation` - Whitelisting, bounds checks, sanitization, and schema validation
- `sec-dependency-security` - Known vulnerable packages and supply-chain exposure
- `sec-configuration-security` - Hardcoded secrets, insecure defaults, debug exposure, and CORS/security headers

## Scope Discipline

Apply this lens when explicitly requested or when a review workflow dispatches it.

### In Scope
- Review vulnerabilities, injection risks, auth/authz flaws, and unsafe data handling
- Check input validation, sanitization, and bounds enforcement
- Verify secrets, PII, and encrypted data stay protected
- Assess dependency and configuration security relevant to the diff

### Out of Scope
- ❌ Primary type coverage analysis
- ❌ Retry and timeout hygiene unless it affects security controls
- ❌ Performance or architecture trade-offs as the main lens
- ❌ Readability-only issues

## Output Format

```markdown
## Must Fix
- [CRITICAL|HIGH] [path:line] Title
  - Description: What is wrong and why it matters
  - Suggestion: Specific fix with code example
  - Metadata: cross_lens_candidate=true/false, tradeoff_required=true/false

## Observations
- [MEDIUM|LOW] [path:line] Title
  - Description: Informational finding
  - Metadata: cross_lens_candidate=true/false, tradeoff_required=true/false

## Summary
[One paragraph overall assessment]
```

## Severity Scale

- **CRITICAL**: Exploitable vulnerabilities, privilege escalation paths, or material data exposure.
- **HIGH**: Missing controls or weaknesses that could plausibly become vulnerabilities.
- **MEDIUM**: Defense-in-depth gaps and hardening opportunities.
- **LOW**: Minor hardening or hygiene improvements.

## Metadata Guidance Tags

- **cross_lens_candidate** — true when the security issue should also trigger another review lens, otherwise false.
- **tradeoff_required** — true when the fix affects usability, performance, or architecture trade-offs, otherwise false.

## Adversarial Input Discipline

- Construct one concrete adversarial input that exercises the primary code path changed in the diff.
- Trace what the system does with that input, including validation, sanitization, authorization, persistence, and output behavior.
- If no adversarial input can be constructed, return BLOCKED.

## Integration Notes

- Part of the six-pass review protocol. Precedence: SECURITY > ERROR_HANDLING > TYPE_SAFETY > PERFORMANCE > ARCHITECTURE > SIMPLICITY. Any CRITICAL finding blocks approval.
- Dedupe merged findings by `(path, line, title)`.
- Lead with the highest-severity must-fix items first.

## References

- [Pi Ensemble security lens](https://raw.githubusercontent.com/randomm/pi-ensemble/main/skill/code-review-security/SKILL.md)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
