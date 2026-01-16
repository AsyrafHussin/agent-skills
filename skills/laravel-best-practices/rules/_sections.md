# Rule Sections

## Priority Levels

| Level | Description | When to Apply |
|-------|-------------|---------------|
| CRITICAL | Essential for any Laravel app | Always |
| HIGH | Important for maintainability | Most projects |
| MEDIUM | Performance and scalability | Growing applications |
| LOW | Specialized patterns | Large-scale apps |

## Section Overview

### Architecture & Structure (CRITICAL)
Foundational patterns for organizing Laravel applications. Service classes, actions, and proper separation of concerns.

### Eloquent & Database (CRITICAL)
Patterns for efficient database operations. Preventing N+1 queries, proper indexing, and query optimization.

### Controllers & Routing (HIGH)
RESTful conventions, resource controllers, and proper request handling.

### Validation & Requests (HIGH)
Form request classes, custom validation rules, and authorization patterns.

### Security (HIGH)
Protection against common vulnerabilities and authentication/authorization best practices.

### Performance (MEDIUM)
Caching strategies, queue usage, and optimization techniques.

### API Design (MEDIUM)
RESTful API patterns, resource transformers, and consistent responses.

### Testing (LOW-MEDIUM)
Testing strategies, factories, and mocking patterns.
