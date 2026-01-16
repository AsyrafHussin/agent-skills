# Sections

This file defines all sections, their ordering, impact levels, and descriptions.
The section ID (in parentheses) is the filename prefix used to group rules.

---

## 1. Resource Design (rest)

**Impact:** CRITICAL
**Description:** REST resource design is the foundation of a well-architected API. Proper HTTP method usage, status codes, and resource naming enable caching, retry logic, and semantic operations. Idempotency prevents duplicate operations.

## 2. Error Handling (error)

**Impact:** CRITICAL
**Description:** Consistent, detailed error responses reduce support burden and enable programmatic error handling. Machine-readable error codes, validation details, and request IDs are essential for debugging and monitoring.

## 3. Security (sec)

**Impact:** CRITICAL
**Description:** Security controls protect user data and prevent abuse. Authentication, authorization, HTTPS, input validation, rate limiting, and sensitive data protection are non-negotiable for production APIs.

## 4. Pagination & Filtering (page)

**Impact:** HIGH
**Description:** Efficient data retrieval for large datasets. Cursor and offset-based pagination, consistent parameter naming, and flexible filtering reduce API response times and improve user experience.

## 5. Versioning (ver)

**Impact:** HIGH
**Description:** API versioning enables evolution without breaking existing clients. URL path or header-based versioning, backward compatibility, and deprecation strategies manage API changes gracefully.

## 6. Response Format (resp)

**Impact:** MEDIUM
**Description:** Consistent response structure, JSON conventions, and field selection improve API predictability and developer experience. Response compression reduces bandwidth usage.

## 7. Documentation (doc)

**Impact:** MEDIUM
**Description:** OpenAPI/Swagger specs, request/response examples, and changelogs make APIs self-documenting and reduce integration time for developers.
