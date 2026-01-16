# API Design Patterns

RESTful API design principles and best practices for building consistent, developer-friendly APIs.

## Overview

This skill provides guidance for:
- RESTful resource design
- HTTP methods and status codes
- Error handling patterns
- Pagination and filtering
- API versioning
- Security best practices

## Categories

### 1. Resource Design (Critical)
Nouns over verbs, plural resources, proper nesting, HTTP methods, status codes.

### 2. Error Handling (Critical)
Consistent error format, meaningful messages, validation details, error codes.

### 3. Security (Critical)
Authentication, authorization, rate limiting, input validation, CORS.

### 4. Pagination & Filtering (High)
Cursor-based, offset-based, filtering, sorting, field selection.

### 5. Versioning (High)
URL path versioning, header versioning, backward compatibility.

### 6. Response Format (Medium)
Consistent structure, JSON conventions, compression.

## Usage

Ask Claude to:
- "Review my API design"
- "Check REST best practices"
- "Design error responses"
- "Review API endpoints"

## Key Principles

### REST Resource Design
- Use nouns, not verbs: `/users` not `/getUsers`
- Use plural names: `/users` not `/user`
- Nest logically: `/users/123/orders`
- Max 2 levels of nesting

### HTTP Methods
- GET: Retrieve
- POST: Create
- PUT: Full update
- PATCH: Partial update
- DELETE: Remove

### Status Codes
- 200: Success
- 201: Created
- 204: No Content
- 400: Bad Request
- 401: Unauthorized
- 403: Forbidden
- 404: Not Found
- 422: Validation Error
- 429: Rate Limited
- 500: Server Error

## References

- [HTTP Status Codes](https://httpstatuses.com/)
- [JSON:API Specification](https://jsonapi.org/)
- [OpenAPI Specification](https://swagger.io/specification/)
