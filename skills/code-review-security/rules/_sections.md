# Sections

This file defines the section ordering, severity, and summary used by this review lens.

---

## 1. Injection Vulnerabilities (sec-injection-vulnerabilities)

**Impact:** CRITICAL
**Description:** SQL, command, XSS, template, and path traversal injection risks.

## 2. Authentication & Authorization (sec-auth-authorization)

**Impact:** CRITICAL
**Description:** Identity checks, session handling, access control, and token safety.

## 3. Data Protection (sec-data-protection)

**Impact:** HIGH
**Description:** Secrets, PII, encryption, logs, and exposure boundaries.

## 4. Input Validation (sec-input-validation)

**Impact:** HIGH
**Description:** Whitelisting, bounds checks, sanitization, and schema validation.

## 5. Dependency Security (sec-dependency-security)

**Impact:** MEDIUM
**Description:** Known vulnerable packages and supply-chain exposure.

## 6. Configuration Security (sec-configuration-security)

**Impact:** HIGH
**Description:** Hardcoded secrets, insecure defaults, debug exposure, and CORS/security headers.
