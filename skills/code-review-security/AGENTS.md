# Code Review Security — Full Compiled Reference

Security review lens for vulnerabilities, input validation, data protection, auth/authz, and secure defaults.

**Version:** 1.0.0 | **Rules:** 6 | **License:** MIT

---

## Operational Contract

When applying this skill, agents must:
- Treat this skill as repo-owned guidance and defer to repository or task-specific instructions when they conflict.
- Limit work to the smallest relevant file and rule set for the current request.
- Stop and ask when the scope, validation command, or required context is missing or contradictory.
- Prefer machine-readable evidence first, then summarize files reviewed, commands run, failures, and unresolved risks.

## Validation & Evidence

- Run the repository's existing validation commands in documented order when code changes are requested.
- If the repository does not define validation for the task, say so instead of inventing one.
- When the lens requires cross-file inspection, name the extra files reviewed.

## Trigger Phrases

The skill activates on:
- "security review"
- "check for vulnerabilities"
- "auth review"
- "injection risks"
- "secure code review"

## Scope Discipline

### In Scope
- Review vulnerabilities, injection risks, auth/authz flaws, and unsafe data handling
- Check input validation, sanitization, and bounds enforcement
- Verify secrets, PII, and encrypted data stay protected
- Assess dependency and configuration security relevant to the diff

### Do Not Broaden Into
- ❌ Primary type coverage analysis
- ❌ Retry and timeout hygiene unless it affects security controls
- ❌ Performance or architecture trade-offs as the main lens
- ❌ Readability-only issues

## Output Format

Use this exact structure:

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

## Metadata Guidance

- **cross_lens_candidate** — true when the security issue should also trigger another review lens, otherwise false.
- **tradeoff_required** — true when the fix affects usability, performance, or architecture trade-offs, otherwise false.

## Adversarial Input Discipline

- Construct one concrete adversarial input that exercises the primary code path changed in the diff.
- Trace what the system does with that input, including validation, sanitization, authorization, persistence, and output behavior.
- If no adversarial input can be constructed, return BLOCKED.

---


## Section 1: Injection Vulnerabilities — CRITICAL

SQL, command, XSS, template, and path traversal injection risks.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to injection vulnerabilities.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.

---


## Section 2: Authentication & Authorization — CRITICAL

Identity checks, session handling, access control, and token safety.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to authentication & authorization.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.

---


## Section 3: Data Protection — HIGH

Secrets, PII, encryption, logs, and exposure boundaries.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to data protection.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.

---


## Section 4: Input Validation — HIGH

Whitelisting, bounds checks, sanitization, and schema validation.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to input validation.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.

---


## Section 5: Dependency Security — MEDIUM

Known vulnerable packages and supply-chain exposure.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to dependency security.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.

---


## Section 6: Configuration Security — HIGH

Hardcoded secrets, insecure defaults, debug exposure, and CORS/security headers.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to configuration security.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.


---

## Integration Notes

- Part of the six-pass review protocol. Precedence: SECURITY > ERROR_HANDLING > TYPE_SAFETY > PERFORMANCE > ARCHITECTURE > SIMPLICITY. Any CRITICAL finding blocks approval.
- Findings merge deterministically across lenses by `(path, line, title)`.
- CRITICAL and HIGH findings belong in `## Must Fix`; MEDIUM and LOW belong in `## Observations` unless repo-specific instructions say otherwise.
