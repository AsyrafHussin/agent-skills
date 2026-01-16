# API Design Patterns - Complete Reference

This document contains the complete compiled rules and guidelines for RESTful API design patterns.

**Version:** 1.0.0  
**License:** MIT  
**References:**
- [RESTful API](https://restfulapi.net)
- [Zalando REST API Guidelines](https://zalando.github.io/restful-api-guidelines)
- [RFC 7231 - HTTP/1.1 Semantics](https://www.ietf.org/rfc/rfc7231.txt)
- [RFC 6749 - OAuth 2.0](https://www.ietf.org/rfc/rfc6749.txt)
- [JWT.io](https://jwt.io)

---


# API Design Patterns

RESTful API design principles, error handling, pagination, versioning, and security best practices. Guidelines for building consistent, developer-friendly APIs.

## When to Apply

Reference these guidelines when:
- Designing new API endpoints
- Reviewing existing API structure
- Implementing error handling
- Setting up pagination/filtering
- Planning API versioning

## Rule Categories by Priority

| Priority | Category | Impact | Prefix |
|----------|----------|--------|--------|
| 1 | Resource Design | CRITICAL | `rest-` |
| 2 | Error Handling | CRITICAL | `error-` |
| 3 | Security | CRITICAL | `sec-` |
| 4 | Pagination & Filtering | HIGH | `page-` |
| 5 | Versioning | HIGH | `ver-` |
| 6 | Response Format | MEDIUM | `resp-` |
| 7 | Documentation | MEDIUM | `doc-` |

## Quick Reference

### 1. Resource Design (CRITICAL)

- `rest-nouns-not-verbs` - Use nouns for endpoints
- `rest-plural-resources` - Use plural resource names
- `rest-http-methods` - Correct HTTP method usage
- `rest-nested-resources` - Proper resource nesting
- `rest-status-codes` - Appropriate status codes
- `rest-idempotency` - Idempotent operations
- `rest-hateoas` - Hypermedia links

### 2. Error Handling (CRITICAL)

- `error-consistent-format` - Consistent error structure
- `error-meaningful-messages` - Helpful error messages
- `error-validation-details` - Field-level validation errors
- `error-codes` - Machine-readable error codes
- `error-stack-traces` - Never expose in production

### 3. Security (CRITICAL)

- `sec-authentication` - Proper auth implementation
- `sec-authorization` - Resource-level permissions
- `sec-rate-limiting` - Prevent abuse
- `sec-input-validation` - Validate all input
- `sec-cors` - CORS configuration
- `sec-sensitive-data` - Protect sensitive data

### 4. Pagination & Filtering (HIGH)

- `page-cursor-based` - Cursor pagination for large datasets
- `page-offset-based` - Offset pagination for simple cases
- `page-consistent-params` - Consistent parameter names
- `page-metadata` - Include pagination metadata
- `filter-query-params` - Filter via query parameters
- `sort-flexible` - Flexible sorting options

### 5. Versioning (HIGH)

- `ver-url-path` - Version in URL path
- `ver-header-based` - Version in headers
- `ver-backward-compatible` - Maintain compatibility
- `ver-deprecation` - Deprecation strategy

### 6. Response Format (MEDIUM)

- `resp-consistent-structure` - Consistent response envelope
- `resp-json-conventions` - JSON naming conventions
- `resp-partial-responses` - Field selection
- `resp-compression` - Response compression

### 7. Documentation (MEDIUM)

- `doc-openapi` - OpenAPI/Swagger spec
- `doc-examples` - Request/response examples
- `doc-changelog` - API changelog

## Essential Guidelines

### Resource Naming

```
# ❌ Verbs in URLs - RPC style
GET    /getUsers
POST   /createUser
PUT    /updateUser/123
DELETE /deleteUser/123

# ✅ Nouns with HTTP methods - REST style
GET    /users          # List users
POST   /users          # Create user
GET    /users/123      # Get user
PUT    /users/123      # Update user (full)
PATCH  /users/123      # Update user (partial)
DELETE /users/123      # Delete user
```

### Nested Resources

```
# ✅ Logical nesting (max 2 levels)
GET    /users/123/orders              # User's orders
GET    /users/123/orders/456          # Specific order
POST   /users/123/orders              # Create order for user

# ❌ Too deeply nested
GET    /users/123/orders/456/items/789/reviews

# ✅ Flatten when appropriate
GET    /order-items/789/reviews       # Direct access
```

### HTTP Methods & Status Codes

```
# GET - Retrieve resource(s)
GET /users              → 200 OK + array
GET /users/123          → 200 OK + object
GET /users/999          → 404 Not Found

# POST - Create resource
POST /users             → 201 Created + object + Location header
POST /users (invalid)   → 400 Bad Request + errors

# PUT - Full update (replace)
PUT /users/123          → 200 OK + updated object
PUT /users/999          → 404 Not Found

# PATCH - Partial update
PATCH /users/123        → 200 OK + updated object

# DELETE - Remove resource
DELETE /users/123       → 204 No Content
DELETE /users/999       → 404 Not Found

# Other common status codes
401 Unauthorized        # Not authenticated
403 Forbidden           # Authenticated but not authorized
409 Conflict            # Resource state conflict
422 Unprocessable       # Validation failed
429 Too Many Requests   # Rate limited
500 Internal Error      # Server error
503 Service Unavailable # Maintenance/overload
```

### Error Response Format

```json
// ❌ Inconsistent error formats
{ "error": "Not found" }
{ "message": "Invalid email" }
{ "errors": ["Error 1", "Error 2"] }

// ✅ Consistent error envelope
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The request contains invalid data",
    "details": [
      {
        "field": "email",
        "code": "INVALID_FORMAT",
        "message": "Please provide a valid email address"
      },
      {
        "field": "password",
        "code": "TOO_SHORT",
        "message": "Password must be at least 8 characters"
      }
    ],
    "request_id": "req_abc123"
  }
}

// ✅ Not found error
{
  "error": {
    "code": "NOT_FOUND",
    "message": "User with ID 123 not found",
    "request_id": "req_def456"
  }
}

// ✅ Server error (no sensitive details)
{
  "error": {
    "code": "INTERNAL_ERROR",
    "message": "An unexpected error occurred. Please try again.",
    "request_id": "req_ghi789"
  }
}
```

### Pagination

```json
// ✅ Offset-based pagination
GET /users?page=2&per_page=20

{
  "data": [...],
  "meta": {
    "current_page": 2,
    "per_page": 20,
    "total_pages": 10,
    "total_count": 195
  },
  "links": {
    "first": "/users?page=1&per_page=20",
    "prev": "/users?page=1&per_page=20",
    "next": "/users?page=3&per_page=20",
    "last": "/users?page=10&per_page=20"
  }
}

// ✅ Cursor-based pagination (for large datasets)
GET /users?cursor=eyJpZCI6MTIzfQ&limit=20

{
  "data": [...],
  "meta": {
    "has_more": true,
    "next_cursor": "eyJpZCI6MTQzfQ"
  },
  "links": {
    "next": "/users?cursor=eyJpZCI6MTQzfQ&limit=20"
  }
}
```

### Filtering & Sorting

```
# ✅ Query parameter filtering
GET /users?status=active
GET /users?role=admin&status=active
GET /users?created_after=2024-01-01

# ✅ Sorting
GET /users?sort=created_at        # Ascending (default)
GET /users?sort=-created_at       # Descending (prefix with -)
GET /users?sort=last_name,-created_at  # Multiple fields

# ✅ Field selection (sparse fieldsets)
GET /users?fields=id,name,email
GET /users/123?fields=id,name,orders

# ✅ Search
GET /users?q=john
GET /users?search=john@example
```

### Consistent Response Structure

```json
// ✅ Single resource
GET /users/123
{
  "data": {
    "id": "123",
    "type": "user",
    "attributes": {
      "name": "John Doe",
      "email": "john@example.com",
      "created_at": "2024-01-15T10:30:00Z"
    },
    "relationships": {
      "orders": {
        "links": {
          "related": "/users/123/orders"
        }
      }
    }
  }
}

// ✅ Collection
GET /users
{
  "data": [
    { "id": "123", "type": "user", ... },
    { "id": "124", "type": "user", ... }
  ],
  "meta": {
    "total_count": 100
  },
  "links": {
    "self": "/users?page=1",
    "next": "/users?page=2"
  }
}

// ✅ Simpler envelope (also acceptable)
{
  "user": {
    "id": "123",
    "name": "John Doe"
  }
}

{
  "users": [...],
  "pagination": {...}
}
```

### Versioning

```
# ✅ URL path versioning (recommended - explicit)
GET /api/v1/users
GET /api/v2/users

# ✅ Header versioning
GET /api/users
Accept: application/vnd.myapi.v2+json

# ✅ Query parameter (simple, but less clean)
GET /api/users?version=2
```

### Rate Limiting

```
# ✅ Include rate limit headers
HTTP/1.1 200 OK
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 998
X-RateLimit-Reset: 1640995200

# ✅ Rate limited response
HTTP/1.1 429 Too Many Requests
Retry-After: 60

{
  "error": {
    "code": "RATE_LIMITED",
    "message": "Too many requests. Please retry after 60 seconds.",
    "retry_after": 60
  }
}
```

### Actions on Resources

```
# For non-CRUD actions, use sub-resources or actions
# ✅ Sub-resource style
POST /users/123/activate
POST /orders/456/cancel
POST /posts/789/publish

# ✅ Or controller-style for complex operations
POST /auth/login
POST /auth/logout
POST /auth/refresh
POST /password/reset
POST /password/reset/confirm
```

### Request Validation

```json
// ✅ Validate and return all errors at once
POST /users
{
  "email": "invalid",
  "password": "short"
}

// Response: 422 Unprocessable Entity
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed",
    "details": [
      {
        "field": "email",
        "code": "INVALID_FORMAT",
        "message": "Must be a valid email address"
      },
      {
        "field": "password",
        "code": "TOO_SHORT",
        "message": "Must be at least 8 characters",
        "meta": {
          "min_length": 8,
          "actual_length": 5
        }
      }
    ]
  }
}
```

### HATEOAS (Hypermedia)

```json
// ✅ Include links for discoverability
{
  "data": {
    "id": "123",
    "status": "pending",
    "total": 99.99
  },
  "links": {
    "self": "/orders/123",
    "cancel": "/orders/123/cancel",
    "pay": "/orders/123/pay",
    "items": "/orders/123/items"
  },
  "actions": {
    "cancel": {
      "method": "POST",
      "href": "/orders/123/cancel"
    },
    "pay": {
      "method": "POST",
      "href": "/orders/123/pay",
      "fields": [
        { "name": "payment_method", "type": "string", "required": true }
      ]
    }
  }
}
```

### Bulk Operations

```json
// ✅ Bulk create
POST /users/bulk
{
  "users": [
    { "name": "User 1", "email": "user1@example.com" },
    { "name": "User 2", "email": "user2@example.com" }
  ]
}

// Response with partial success
{
  "data": {
    "succeeded": [
      { "id": "123", "name": "User 1" }
    ],
    "failed": [
      {
        "index": 1,
        "error": {
          "code": "DUPLICATE_EMAIL",
          "message": "Email already exists"
        }
      }
    ]
  },
  "meta": {
    "total": 2,
    "succeeded": 1,
    "failed": 1
  }
}
```

## Output Format

When auditing APIs, output findings in this format:

```
endpoint - [category] Description of issue
```

Example:
```
GET /getUsers - [rest] Use noun '/users' instead of verb '/getUsers'
POST /users - [error] Missing validation error details in 400 response
GET /users - [page] Missing pagination metadata in list response
```

## How to Use

Read individual rule files for detailed explanations:

```
rules/rest-http-methods.md
rules/error-consistent-format.md
rules/page-cursor-based.md
```

---

# Detailed Rules

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

---


## Include HATEOAS Links for Discoverability

HATEOAS (Hypermedia as the Engine of Application State) provides links in responses that guide clients to related resources and available actions.

## Bad Example

```json
// Anti-pattern: No links, client must construct URLs
{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com",
  "orderId": 456
}
// Client must know to call GET /orders/456 to get order details
// No indication of available actions
```

```javascript
// Response without navigation
app.get('/users/:id', async (req, res) => {
  const user = await db.findUser(req.params.id);
  res.json(user); // Raw data only
});
```

## Good Example

```javascript
// Response with HATEOAS links
app.get('/users/:id', async (req, res) => {
  const user = await db.findUser(req.params.id);
  const baseUrl = `${req.protocol}://${req.get('host')}`;

  res.json({
    id: user.id,
    name: user.name,
    email: user.email,
    _links: {
      self: {
        href: `${baseUrl}/users/${user.id}`,
        method: 'GET'
      },
      update: {
        href: `${baseUrl}/users/${user.id}`,
        method: 'PUT'
      },
      delete: {
        href: `${baseUrl}/users/${user.id}`,
        method: 'DELETE'
      },
      orders: {
        href: `${baseUrl}/users/${user.id}/orders`,
        method: 'GET'
      },
      createOrder: {
        href: `${baseUrl}/users/${user.id}/orders`,
        method: 'POST'
      }
    }
  });
});

// Collection with pagination links
app.get('/users', async (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const limit = parseInt(req.query.limit) || 20;
  const { users, total } = await db.findUsers({ page, limit });
  const baseUrl = `${req.protocol}://${req.get('host')}`;
  const totalPages = Math.ceil(total / limit);

  res.json({
    data: users.map(user => ({
      ...user,
      _links: {
        self: { href: `${baseUrl}/users/${user.id}` }
      }
    })),
    _links: {
      self: { href: `${baseUrl}/users?page=${page}&limit=${limit}` },
      first: { href: `${baseUrl}/users?page=1&limit=${limit}` },
      last: { href: `${baseUrl}/users?page=${totalPages}&limit=${limit}` },
      ...(page > 1 && {
        prev: { href: `${baseUrl}/users?page=${page - 1}&limit=${limit}` }
      }),
      ...(page < totalPages && {
        next: { href: `${baseUrl}/users?page=${page + 1}&limit=${limit}` }
      })
    },
    _meta: {
      currentPage: page,
      totalPages,
      totalItems: total,
      itemsPerPage: limit
    }
  });
});
```

```json
// Example response with HATEOAS
{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com",
  "status": "active",
  "_links": {
    "self": {
      "href": "https://api.example.com/users/123",
      "method": "GET"
    },
    "update": {
      "href": "https://api.example.com/users/123",
      "method": "PUT"
    },
    "deactivate": {
      "href": "https://api.example.com/users/123/deactivate",
      "method": "POST"
    },
    "orders": {
      "href": "https://api.example.com/users/123/orders",
      "method": "GET"
    },
    "avatar": {
      "href": "https://api.example.com/users/123/avatar",
      "method": "GET",
      "type": "image/png"
    }
  },
  "_embedded": {
    "latestOrder": {
      "id": 456,
      "total": 99.99,
      "_links": {
        "self": { "href": "https://api.example.com/orders/456" }
      }
    }
  }
}
```

```python
# FastAPI with HATEOAS helper
from fastapi import FastAPI, Request
from pydantic import BaseModel
from typing import Dict, List, Optional, Any

app = FastAPI()

class Link(BaseModel):
    href: str
    method: str = "GET"
    type: Optional[str] = None

class HATEOASResponse(BaseModel):
    data: Any
    _links: Dict[str, Link]
    _embedded: Optional[Dict[str, Any]] = None

def build_user_links(request: Request, user_id: int) -> Dict[str, Link]:
    base_url = str(request.base_url).rstrip('/')
    return {
        "self": Link(href=f"{base_url}/users/{user_id}"),
        "update": Link(href=f"{base_url}/users/{user_id}", method="PUT"),
        "delete": Link(href=f"{base_url}/users/{user_id}", method="DELETE"),
        "orders": Link(href=f"{base_url}/users/{user_id}/orders"),
    }

@app.get("/users/{user_id}")
async def get_user(user_id: int, request: Request):
    user = await db.get_user(user_id)
    return {
        **user.dict(),
        "_links": build_user_links(request, user_id)
    }

@app.get("/orders/{order_id}")
async def get_order(order_id: int, request: Request):
    order = await db.get_order(order_id)
    base_url = str(request.base_url).rstrip('/')

    return {
        **order.dict(),
        "_links": {
            "self": {"href": f"{base_url}/orders/{order_id}"},
            "customer": {"href": f"{base_url}/users/{order.customer_id}"},
            "items": {"href": f"{base_url}/orders/{order_id}/items"},
            "cancel": {"href": f"{base_url}/orders/{order_id}/cancel", "method": "POST"}
                if order.status == "pending" else None,
            "invoice": {"href": f"{base_url}/orders/{order_id}/invoice", "type": "application/pdf"}
        }
    }
```

## HAL Format (Common Standard)

```json
{
  "_links": {
    "self": { "href": "/orders/123" },
    "customer": { "href": "/customers/456", "title": "John Doe" },
    "items": { "href": "/orders/123/items" }
  },
  "id": 123,
  "total": 99.99,
  "status": "shipped",
  "_embedded": {
    "items": [
      {
        "_links": { "self": { "href": "/products/789" } },
        "name": "Widget",
        "quantity": 2
      }
    ]
  }
}
```

## Why

1. **Self-Documenting**: Responses tell clients exactly what actions are available and how to perform them.

2. **Loose Coupling**: Clients don't need hardcoded URL patterns; they follow links dynamically.

3. **Evolvability**: APIs can change URL structures without breaking clients that follow links.

4. **Discoverability**: New features are automatically discoverable through new links.

5. **Context-Aware**: Links can vary based on resource state (e.g., "cancel" only shown for pending orders).

6. **Reduced Documentation**: Clients can explore the API by following links.

7. **Workflow Guidance**: Links guide users through multi-step processes naturally.

---


## Use HTTP Methods Correctly

HTTP methods have specific semantics and should be used according to their intended purpose. Each method has distinct characteristics for safety and idempotency.

## Bad Example

```json
// Anti-pattern: Incorrect method usage
POST /users/123          // Should use GET to retrieve
GET /users/create        // Should use POST to create
POST /users/123/delete   // Should use DELETE method
GET /orders/123/update   // Should use PUT/PATCH
POST /search             // GET is better for safe operations
```

```javascript
// Incorrect method usage
app.post('/users/:id', (req, res) => {
  // Fetching user with POST - wrong!
  const user = db.findUser(req.params.id);
  res.json(user);
});

app.get('/users/delete/:id', (req, res) => {
  // Deleting with GET - dangerous!
  db.deleteUser(req.params.id);
  res.json({ deleted: true });
});

app.post('/users/:id/update', (req, res) => {
  // Custom action verb with POST
  db.updateUser(req.params.id, req.body);
  res.json({ updated: true });
});
```

## Good Example

```javascript
// Correct HTTP method usage
const express = require('express');
const router = express.Router();

// GET - Retrieve resource(s), safe and idempotent
router.get('/users', async (req, res) => {
  const users = await db.findUsers(req.query);
  res.json(users);
});

router.get('/users/:id', async (req, res) => {
  const user = await db.findUser(req.params.id);
  if (!user) return res.status(404).json({ error: 'User not found' });
  res.json(user);
});

// POST - Create new resource, not idempotent
router.post('/users', async (req, res) => {
  const user = await db.createUser(req.body);
  res.status(201).json(user);
});

// PUT - Replace entire resource, idempotent
router.put('/users/:id', async (req, res) => {
  const user = await db.replaceUser(req.params.id, req.body);
  res.json(user);
});

// PATCH - Partial update, idempotent
router.patch('/users/:id', async (req, res) => {
  const user = await db.updateUser(req.params.id, req.body);
  res.json(user);
});

// DELETE - Remove resource, idempotent
router.delete('/users/:id', async (req, res) => {
  await db.deleteUser(req.params.id);
  res.status(204).send();
});

// HEAD - Same as GET but no body, for checking existence
router.head('/users/:id', async (req, res) => {
  const exists = await db.userExists(req.params.id);
  res.status(exists ? 200 : 404).send();
});

// OPTIONS - Return allowed methods
router.options('/users', (req, res) => {
  res.set('Allow', 'GET, POST, OPTIONS');
  res.status(204).send();
});
```

```python
# FastAPI with correct HTTP methods
from fastapi import FastAPI, HTTPException, status

app = FastAPI()

# GET - Retrieve
@app.get("/users")
def list_users(skip: int = 0, limit: int = 10):
    return db.get_users(skip=skip, limit=limit)

@app.get("/users/{user_id}")
def get_user(user_id: int):
    user = db.get_user(user_id)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user

# POST - Create
@app.post("/users", status_code=status.HTTP_201_CREATED)
def create_user(user: UserCreate):
    return db.create_user(user)

# PUT - Full replacement
@app.put("/users/{user_id}")
def replace_user(user_id: int, user: UserUpdate):
    return db.replace_user(user_id, user)

# PATCH - Partial update
@app.patch("/users/{user_id}")
def update_user(user_id: int, user: UserPatch):
    return db.update_user(user_id, user.dict(exclude_unset=True))

# DELETE - Remove
@app.delete("/users/{user_id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_user(user_id: int):
    db.delete_user(user_id)
    return None
```

## HTTP Methods Reference

| Method  | Purpose | Safe | Idempotent | Request Body | Response Body |
|---------|---------|------|------------|--------------|---------------|
| GET     | Retrieve | Yes  | Yes        | No           | Yes           |
| POST    | Create   | No   | No         | Yes          | Yes           |
| PUT     | Replace  | No   | Yes        | Yes          | Yes           |
| PATCH   | Update   | No   | Yes        | Yes          | Yes           |
| DELETE  | Remove   | No   | Yes        | Optional     | Optional      |
| HEAD    | Headers  | Yes  | Yes        | No           | No            |
| OPTIONS | Methods  | Yes  | Yes        | No           | No            |

## Why

1. **Semantic Clarity**: Each method has a clear, well-defined purpose that all developers understand.

2. **Cacheability**: GET requests can be cached by browsers and CDNs because they're safe and idempotent.

3. **Browser Behavior**: Browsers handle different methods appropriately (e.g., warn before resubmitting POST forms).

4. **Middleware Support**: Security tools, load balancers, and proxies understand HTTP semantics.

5. **Retry Logic**: Idempotent methods (GET, PUT, DELETE) can be safely retried on network failures.

6. **Security**: GET requests shouldn't modify data, preventing accidental changes from link clicks or crawlers.

7. **API Documentation**: Tools like Swagger/OpenAPI rely on correct method usage for accurate documentation.

---


## Implement Idempotency for Safe Retries

Idempotent operations produce the same result regardless of how many times they're executed. Implement idempotency keys for non-idempotent operations to enable safe retries.

## Bad Example

```javascript
// Anti-pattern: Non-idempotent POST without protection
app.post('/payments', async (req, res) => {
  // Each retry creates a duplicate payment!
  const payment = await db.createPayment({
    amount: req.body.amount,
    customerId: req.body.customerId
  });
  await chargeCard(payment);
  res.status(201).json(payment);
});

// Anti-pattern: No idempotency key checking
app.post('/orders', async (req, res) => {
  // Network timeout after processing = client retries = duplicate order
  const order = await db.createOrder(req.body);
  await processOrder(order);
  res.status(201).json(order);
});
```

```json
// Client retries without idempotency key
POST /payments
{
  "amount": 100,
  "customerId": "cust_123"
}
// Timeout... retry... duplicate payment created!
```

## Good Example

```javascript
const express = require('express');
const router = express.Router();

// Idempotency key middleware
const idempotencyStore = new Map(); // Use Redis in production

async function idempotencyMiddleware(req, res, next) {
  const idempotencyKey = req.headers['idempotency-key'];

  if (!idempotencyKey) {
    return res.status(400).json({
      error: 'missing_idempotency_key',
      message: 'Idempotency-Key header is required for this operation'
    });
  }

  const cacheKey = `${req.path}:${idempotencyKey}`;
  const cached = idempotencyStore.get(cacheKey);

  if (cached) {
    // Return cached response
    return res.status(cached.status).json(cached.body);
  }

  // Store original json function
  const originalJson = res.json.bind(res);

  // Override to cache response
  res.json = function(body) {
    idempotencyStore.set(cacheKey, {
      status: res.statusCode,
      body: body
    });
    // Set TTL (24 hours typical)
    setTimeout(() => idempotencyStore.delete(cacheKey), 24 * 60 * 60 * 1000);
    return originalJson(body);
  };

  next();
}

// Apply to non-idempotent operations
router.post('/payments', idempotencyMiddleware, async (req, res) => {
  const payment = await db.createPayment({
    amount: req.body.amount,
    customerId: req.body.customerId,
    idempotencyKey: req.headers['idempotency-key']
  });

  await chargeCard(payment);
  res.status(201).json(payment);
});

// Idempotent by design using upsert
router.put('/users/:id/preferences', async (req, res) => {
  // PUT is idempotent - same request always produces same result
  const preferences = await db.upsertPreferences(
    req.params.id,
    req.body
  );
  res.json(preferences);
});

// Natural idempotency with unique constraints
router.post('/subscriptions', async (req, res) => {
  try {
    const subscription = await db.createSubscription({
      userId: req.body.userId,
      planId: req.body.planId
    });
    res.status(201).json(subscription);
  } catch (error) {
    if (error.code === 'UNIQUE_VIOLATION') {
      // Return existing subscription
      const existing = await db.findSubscription(
        req.body.userId,
        req.body.planId
      );
      return res.status(200).json(existing);
    }
    throw error;
  }
});
```

```python
# FastAPI with idempotency
from fastapi import FastAPI, Header, HTTPException
from functools import wraps
import redis

app = FastAPI()
redis_client = redis.Redis()

def idempotent(ttl_seconds: int = 86400):
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, idempotency_key: str = Header(...), **kwargs):
            cache_key = f"idempotency:{func.__name__}:{idempotency_key}"

            # Check cache
            cached = redis_client.get(cache_key)
            if cached:
                return json.loads(cached)

            # Execute operation
            result = await func(*args, **kwargs)

            # Cache result
            redis_client.setex(cache_key, ttl_seconds, json.dumps(result))

            return result
        return wrapper
    return decorator

@app.post("/payments")
@idempotent(ttl_seconds=86400)
async def create_payment(payment: PaymentCreate):
    result = await process_payment(payment)
    return {"id": result.id, "status": result.status}
```

```json
// Client request with idempotency key
POST /payments HTTP/1.1
Host: api.example.com
Content-Type: application/json
Idempotency-Key: unique-request-id-12345

{
  "amount": 100,
  "customerId": "cust_123"
}

// Response (same for retries)
HTTP/1.1 201 Created
Idempotency-Key: unique-request-id-12345

{
  "id": "pay_789",
  "amount": 100,
  "customerId": "cust_123",
  "status": "completed"
}
```

## Idempotency by HTTP Method

| Method | Naturally Idempotent | Notes |
|--------|---------------------|-------|
| GET | Yes | Always safe to retry |
| HEAD | Yes | Always safe to retry |
| OPTIONS | Yes | Always safe to retry |
| PUT | Yes | Full replacement is idempotent |
| DELETE | Yes | Deleting twice = same result |
| POST | No | Needs idempotency key |
| PATCH | Usually | Depends on implementation |

## Why

1. **Network Reliability**: Networks fail. Clients need to safely retry requests without causing duplicate operations.

2. **Financial Safety**: Duplicate payments or orders can cause significant problems. Idempotency prevents this.

3. **User Experience**: Users can safely click "submit" multiple times without fear of duplicate actions.

4. **Distributed Systems**: In microservices, requests may be processed multiple times due to retries and message queues.

5. **Client Simplicity**: Clients don't need complex logic to track what succeeded; they can simply retry.

6. **Audit Trail**: Idempotency keys provide a way to track and correlate requests across systems.

7. **At-Least-Once Delivery**: Many message systems guarantee at-least-once delivery, requiring idempotent consumers.

---


## Design Nested Resources for Hierarchical Relationships

Use nested URLs to represent parent-child relationships between resources, but avoid deep nesting beyond two levels.

## Bad Example

```json
// Anti-pattern: Deeply nested resources (3+ levels)
GET /companies/123/departments/456/employees/789/projects/101/tasks/202
POST /organizations/1/teams/2/members/3/assignments/4/subtasks

// Anti-pattern: Flat structure losing context
GET /tasks/202          // Which project? Which employee?
GET /comments/999       // Comment on what?

// Anti-pattern: Inconsistent nesting
GET /users/123/orders   // Nested
GET /order-items?orderId=456  // Query param
GET /products/789/reviews     // Nested again
```

```javascript
// Overly deep nesting
app.get('/companies/:companyId/departments/:deptId/employees/:empId/reviews/:reviewId',
  (req, res) => {
    // 4 levels deep - too complex!
    const { companyId, deptId, empId, reviewId } = req.params;
    // ...
  }
);
```

## Good Example

```json
// Correct: Maximum 2 levels of nesting
GET /users/123/orders           // User's orders
GET /orders/456/items           // Order's items
GET /posts/789/comments         // Post's comments

// Access deep resources directly when needed
GET /tasks/202                  // Direct access with task ID
GET /employees/789              // Direct access with employee ID

// Use query parameters for filtering
GET /tasks?projectId=101        // Filter tasks by project
GET /tasks?employeeId=789&status=active
```

```javascript
// Express router with appropriate nesting
const router = express.Router();

// Parent resource
router.get('/users', listUsers);
router.get('/users/:userId', getUser);
router.post('/users', createUser);

// Nested child resource (1 level)
router.get('/users/:userId/orders', getUserOrders);
router.post('/users/:userId/orders', createUserOrder);
router.get('/users/:userId/orders/:orderId', getUserOrder);

// Second-level nested resource (2 levels max)
router.get('/users/:userId/orders/:orderId/items', getOrderItems);
router.post('/users/:userId/orders/:orderId/items', addOrderItem);

// Direct access for deep resources
router.get('/orders/:orderId', getOrder);
router.get('/order-items/:itemId', getOrderItem);
router.patch('/order-items/:itemId', updateOrderItem);
```

```python
# FastAPI with nested resources
from fastapi import APIRouter

router = APIRouter()

# Users - parent resource
@router.get("/users/{user_id}")
def get_user(user_id: int):
    return db.get_user(user_id)

# Posts - nested under users
@router.get("/users/{user_id}/posts")
def get_user_posts(user_id: int, skip: int = 0, limit: int = 10):
    return db.get_posts_by_user(user_id, skip, limit)

@router.post("/users/{user_id}/posts")
def create_user_post(user_id: int, post: PostCreate):
    return db.create_post(user_id, post)

# Comments - nested under posts (2 levels)
@router.get("/posts/{post_id}/comments")
def get_post_comments(post_id: int):
    return db.get_comments_by_post(post_id)

# Direct access for comments when needed
@router.get("/comments/{comment_id}")
def get_comment(comment_id: int):
    return db.get_comment(comment_id)

@router.patch("/comments/{comment_id}")
def update_comment(comment_id: int, update: CommentUpdate):
    return db.update_comment(comment_id, update)
```

```yaml
# OpenAPI spec with nested resources
openapi: 3.0.0
paths:
  /users/{userId}/orders:
    get:
      summary: Get all orders for a user
      parameters:
        - name: userId
          in: path
          required: true
          schema:
            type: integer

  /users/{userId}/orders/{orderId}:
    get:
      summary: Get a specific order for a user

  /orders/{orderId}/items:
    get:
      summary: Get all items in an order
    post:
      summary: Add item to order

  # Direct access endpoint
  /orders/{orderId}:
    get:
      summary: Get order by ID directly
```

## Why

1. **Clarity of Relationships**: Nested URLs clearly show ownership and hierarchy (e.g., `/users/123/orders` shows orders belonging to user 123).

2. **Natural Authorization**: The URL structure makes it easy to enforce that users can only access their own resources.

3. **Avoid Deep Nesting**: Beyond 2 levels, URLs become unwieldy and difficult to work with. Use direct access or query parameters instead.

4. **Flexibility**: Provide both nested and direct access patterns to accommodate different use cases.

5. **Scoped Operations**: Creating a resource under a parent automatically associates them (POST `/users/123/orders` creates an order for user 123).

6. **Better Error Messages**: Nested structure enables specific errors like "Order 456 not found for user 123."

7. **Consistent Patterns**: Establish a predictable pattern that developers can rely on across your API.

---


## Use Nouns, Not Verbs for Resource Names

REST API endpoints should represent resources (nouns), not actions (verbs). HTTP methods already convey the action being performed.

## Bad Example

```json
// Anti-pattern: Verbs in endpoint names
GET /getUsers
POST /createUser
PUT /updateUser/123
DELETE /deleteUser/123
GET /fetchAllOrders
POST /addNewProduct
```

```javascript
// Express routes with verb-based endpoints
app.get('/getUsers', getUsers);
app.post('/createUser', createUser);
app.get('/fetchUserById/:id', getUserById);
app.put('/updateUserProfile/:id', updateUser);
app.delete('/removeUser/:id', deleteUser);
```

## Good Example

```json
// Correct: Nouns representing resources
GET /users
POST /users
GET /users/123
PUT /users/123
DELETE /users/123
GET /orders
POST /products
```

```javascript
// Express routes with noun-based endpoints
app.get('/users', listUsers);
app.post('/users', createUser);
app.get('/users/:id', getUser);
app.put('/users/:id', updateUser);
app.delete('/users/:id', deleteUser);
```

```python
# FastAPI with noun-based resources
from fastapi import FastAPI

app = FastAPI()

@app.get("/users")
def list_users():
    return users

@app.post("/users")
def create_user(user: UserCreate):
    return new_user

@app.get("/users/{user_id}")
def get_user(user_id: int):
    return user

@app.put("/users/{user_id}")
def update_user(user_id: int, user: UserUpdate):
    return updated_user

@app.delete("/users/{user_id}")
def delete_user(user_id: int):
    return {"deleted": True}
```

## Why

1. **RESTful Convention**: REST treats URLs as resource identifiers. The HTTP method (GET, POST, PUT, DELETE) already describes the action.

2. **Consistency**: Using nouns creates a predictable, consistent API structure that developers can easily understand and use.

3. **Simplicity**: Reduces the number of endpoints needed. One resource endpoint can handle multiple operations via different HTTP methods.

4. **Discoverability**: Resources become self-documenting when they represent entities in your domain model.

5. **Cacheability**: GET requests to noun-based endpoints can be cached more effectively since they represent stable resource identifiers.

6. **HTTP Semantics**: Leverages the built-in semantics of HTTP methods rather than inventing custom verbs.

---


## Use Plural Nouns for Resource Collections

Resource names should consistently use plural nouns to represent collections, maintaining uniformity across your API.

## Bad Example

```json
// Anti-pattern: Inconsistent singular/plural usage
GET /user          // Singular for collection
GET /user/123      // Singular for individual
GET /products      // Plural for collection
GET /product/123   // Singular for individual
GET /order         // Inconsistent
POST /person       // Mixed conventions
```

```yaml
# OpenAPI spec with inconsistent naming
paths:
  /user:
    get:
      summary: Get all users
  /user/{id}:
    get:
      summary: Get single user
  /products:
    get:
      summary: Get all products
  /product/{id}:
    get:
      summary: Get single product
```

## Good Example

```json
// Correct: Consistent plural nouns
GET /users         // Collection of users
GET /users/123     // Single user from collection
POST /users        // Create user in collection
PUT /users/123     // Update user in collection
DELETE /users/123  // Remove user from collection

GET /products      // Collection
GET /products/456  // Single item
GET /orders        // Collection
GET /orders/789    // Single item
```

```yaml
# OpenAPI spec with consistent plurals
openapi: 3.0.0
paths:
  /users:
    get:
      summary: List all users
      responses:
        '200':
          description: Array of users
    post:
      summary: Create a new user

  /users/{userId}:
    get:
      summary: Get a specific user
    put:
      summary: Update a specific user
    delete:
      summary: Delete a specific user

  /products:
    get:
      summary: List all products

  /products/{productId}:
    get:
      summary: Get a specific product
```

```javascript
// Express router with consistent plurals
const router = express.Router();

// Users resource
router.get('/users', listUsers);
router.post('/users', createUser);
router.get('/users/:id', getUser);
router.put('/users/:id', updateUser);
router.delete('/users/:id', deleteUser);

// Products resource
router.get('/products', listProducts);
router.post('/products', createProduct);
router.get('/products/:id', getProduct);
router.put('/products/:id', updateProduct);
router.delete('/products/:id', deleteProduct);

// Orders resource
router.get('/orders', listOrders);
router.post('/orders', createOrder);
router.get('/orders/:id', getOrder);
```

## Why

1. **Consistency**: Developers don't need to guess whether a resource uses singular or plural form.

2. **Predictability**: When you know one resource uses `/users`, you can predict others will be `/products`, `/orders`, etc.

3. **Collection Semantics**: Plural names clearly indicate that the endpoint returns or operates on a collection of items.

4. **URI Logic**: `/users/123` reads naturally as "user 123 from the users collection."

5. **Database Alignment**: Most databases use plural table names (users, products, orders), making the API consistent with the data model.

6. **Framework Conventions**: Most REST frameworks and documentation generators expect plural resource names.

7. **Avoid Ambiguity**: `/user` could mean "current user" or "user collection," while `/users` clearly means the collection.

---


## Handle Non-CRUD Actions on Resources

Some operations don't fit standard CRUD patterns. Use sub-resources or action endpoints for operations that represent state transitions or complex actions.

## Bad Example

```json
// Anti-pattern: Verbs in main resource path
POST /activateUser/123
POST /deactivateUser/123
POST /sendEmailToUser/123
POST /approveOrder/456
POST /shipOrder/456

// Anti-pattern: Query parameters for actions
POST /users/123?action=activate
POST /orders/456?action=ship&tracking=ABC123
```

```javascript
// Anti-pattern: Complex PATCH with action semantics
app.patch('/orders/:id', (req, res) => {
  // Trying to use PATCH for everything
  if (req.body.status === 'shipped') {
    // Shipping logic, tracking number, notifications...
  } else if (req.body.status === 'cancelled') {
    // Cancellation logic, refund, restock...
  }
});
```

## Good Example

```javascript
// Sub-resource actions (noun-based)
const router = express.Router();

// User lifecycle actions
router.post('/users/:id/activation', activateUser);      // Activate user
router.delete('/users/:id/activation', deactivateUser);  // Deactivate user
router.post('/users/:id/password-reset', resetPassword); // Reset password
router.post('/users/:id/verification', sendVerification); // Send verification

// Order workflow actions
router.post('/orders/:id/shipment', shipOrder);          // Ship order
router.post('/orders/:id/cancellation', cancelOrder);    // Cancel order
router.post('/orders/:id/refund', refundOrder);          // Refund order

// Controller actions as verbs when necessary
router.post('/orders/:id/ship', async (req, res) => {
  const { trackingNumber, carrier } = req.body;
  const order = await orderService.ship(req.params.id, {
    trackingNumber,
    carrier
  });
  res.json(order);
});

router.post('/orders/:id/cancel', async (req, res) => {
  const { reason } = req.body;
  const order = await orderService.cancel(req.params.id, reason);
  res.json(order);
});
```

```python
# FastAPI with action endpoints
from fastapi import APIRouter, HTTPException
from enum import Enum

router = APIRouter()

class OrderStatus(str, Enum):
    PENDING = "pending"
    CONFIRMED = "confirmed"
    SHIPPED = "shipped"
    DELIVERED = "delivered"
    CANCELLED = "cancelled"

# State transition actions
@router.post("/orders/{order_id}/confirm")
async def confirm_order(order_id: int):
    order = await db.get_order(order_id)
    if order.status != OrderStatus.PENDING:
        raise HTTPException(
            status_code=422,
            detail=f"Cannot confirm order in {order.status} status"
        )
    order.status = OrderStatus.CONFIRMED
    await db.save_order(order)
    await notification_service.send_confirmation(order)
    return order

@router.post("/orders/{order_id}/ship")
async def ship_order(order_id: int, shipment: ShipmentInfo):
    order = await db.get_order(order_id)
    if order.status != OrderStatus.CONFIRMED:
        raise HTTPException(
            status_code=422,
            detail=f"Cannot ship order in {order.status} status"
        )

    order.status = OrderStatus.SHIPPED
    order.tracking_number = shipment.tracking_number
    order.carrier = shipment.carrier
    order.shipped_at = datetime.utcnow()

    await db.save_order(order)
    await notification_service.send_shipping_notification(order)
    return order

@router.post("/orders/{order_id}/cancel")
async def cancel_order(order_id: int, cancellation: CancellationRequest):
    order = await db.get_order(order_id)
    if order.status in [OrderStatus.SHIPPED, OrderStatus.DELIVERED]:
        raise HTTPException(
            status_code=422,
            detail="Cannot cancel shipped or delivered orders"
        )

    order.status = OrderStatus.CANCELLED
    order.cancellation_reason = cancellation.reason

    await db.save_order(order)
    await payment_service.refund(order)
    await inventory_service.restock(order.items)

    return order

# Batch actions
@router.post("/orders/bulk-ship")
async def bulk_ship_orders(request: BulkShipRequest):
    results = []
    for order_id in request.order_ids:
        try:
            result = await ship_order(order_id, request.shipment_info)
            results.append({"order_id": order_id, "status": "shipped"})
        except HTTPException as e:
            results.append({"order_id": order_id, "status": "failed", "error": e.detail})
    return {"results": results}
```

```yaml
# OpenAPI spec for actions
openapi: 3.0.0
paths:
  /orders/{orderId}/ship:
    post:
      summary: Ship an order
      description: Transitions order to shipped status
      parameters:
        - name: orderId
          in: path
          required: true
          schema:
            type: integer
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required:
                - trackingNumber
                - carrier
              properties:
                trackingNumber:
                  type: string
                carrier:
                  type: string
                  enum: [ups, fedex, usps, dhl]
      responses:
        '200':
          description: Order shipped successfully
        '422':
          description: Order cannot be shipped in current state

  /orders/{orderId}/cancel:
    post:
      summary: Cancel an order
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                reason:
                  type: string
      responses:
        '200':
          description: Order cancelled successfully
        '422':
          description: Order cannot be cancelled
```

## Patterns for Actions

| Action Type | Pattern | Example |
|-------------|---------|---------|
| State transition | POST /resource/{id}/action | POST /orders/123/ship |
| Sub-resource creation | POST /resource/{id}/sub-resource | POST /users/123/password-reset |
| Batch operation | POST /resources/bulk-action | POST /orders/bulk-ship |
| Controller action | POST /action (no resource ID) | POST /search, POST /calculate |

## Why

1. **Clear Intent**: Action endpoints explicitly show what operation is being performed.

2. **Validation**: Each action can have specific validation rules and business logic.

3. **State Machines**: Complex state transitions are better modeled as explicit actions than PATCH operations.

4. **Side Effects**: Actions that trigger notifications, payments, or other side effects deserve dedicated endpoints.

5. **Documentation**: OpenAPI/Swagger can document each action with specific parameters and responses.

6. **Permissions**: Fine-grained authorization (can ship orders vs. can cancel orders).

7. **Audit Trail**: Each action creates a clear, auditable record of what happened.

---


## Use Appropriate HTTP Status Codes

Return semantically correct HTTP status codes that accurately describe the result of the operation.

## Bad Example

```javascript
// Anti-pattern: Always returning 200
app.post('/users', async (req, res) => {
  try {
    const user = await db.createUser(req.body);
    res.status(200).json(user); // Should be 201 Created
  } catch (error) {
    res.status(200).json({ error: error.message }); // Error with 200!
  }
});

app.get('/users/:id', async (req, res) => {
  const user = await db.findUser(req.params.id);
  if (!user) {
    res.status(200).json({ error: 'Not found' }); // Should be 404
  }
  res.status(200).json(user);
});

app.delete('/users/:id', async (req, res) => {
  await db.deleteUser(req.params.id);
  res.status(200).json({ message: 'Deleted' }); // 204 is more appropriate
});
```

```json
// Anti-pattern: Error responses with 200 status
HTTP/1.1 200 OK
{
  "success": false,
  "error": "User not found"
}
```

## Good Example

```javascript
const express = require('express');
const router = express.Router();

// 200 OK - Successful GET, PUT, PATCH
router.get('/users/:id', async (req, res) => {
  const user = await db.findUser(req.params.id);
  if (!user) {
    return res.status(404).json({
      error: 'not_found',
      message: 'User not found'
    });
  }
  res.status(200).json(user);
});

// 201 Created - Successful POST that creates a resource
router.post('/users', async (req, res) => {
  const user = await db.createUser(req.body);
  res.status(201)
    .location(`/users/${user.id}`)
    .json(user);
});

// 204 No Content - Successful DELETE or update with no response body
router.delete('/users/:id', async (req, res) => {
  const deleted = await db.deleteUser(req.params.id);
  if (!deleted) {
    return res.status(404).json({
      error: 'not_found',
      message: 'User not found'
    });
  }
  res.status(204).send();
});

// 400 Bad Request - Invalid input
router.post('/users', async (req, res) => {
  if (!req.body.email) {
    return res.status(400).json({
      error: 'validation_error',
      message: 'Email is required',
      field: 'email'
    });
  }
  // ...
});

// 401 Unauthorized - Not authenticated
router.use((req, res, next) => {
  if (!req.headers.authorization) {
    return res.status(401).json({
      error: 'unauthorized',
      message: 'Authentication required'
    });
  }
  next();
});

// 403 Forbidden - Authenticated but not authorized
router.delete('/users/:id', async (req, res) => {
  if (req.user.id !== req.params.id && !req.user.isAdmin) {
    return res.status(403).json({
      error: 'forbidden',
      message: 'You cannot delete other users'
    });
  }
  // ...
});

// 409 Conflict - Resource conflict
router.post('/users', async (req, res) => {
  const exists = await db.userExists(req.body.email);
  if (exists) {
    return res.status(409).json({
      error: 'conflict',
      message: 'User with this email already exists'
    });
  }
  // ...
});

// 422 Unprocessable Entity - Semantic validation error
router.post('/orders', async (req, res) => {
  const product = await db.findProduct(req.body.productId);
  if (product.stock < req.body.quantity) {
    return res.status(422).json({
      error: 'unprocessable_entity',
      message: 'Insufficient stock',
      available: product.stock
    });
  }
  // ...
});
```

## Common Status Codes Reference

### Success (2xx)
| Code | Name | Use Case |
|------|------|----------|
| 200 | OK | Successful GET, PUT, PATCH |
| 201 | Created | Successful POST creating resource |
| 202 | Accepted | Request accepted for async processing |
| 204 | No Content | Successful DELETE or update with no body |

### Client Errors (4xx)
| Code | Name | Use Case |
|------|------|----------|
| 400 | Bad Request | Malformed syntax, invalid JSON |
| 401 | Unauthorized | Missing or invalid authentication |
| 403 | Forbidden | Authenticated but not authorized |
| 404 | Not Found | Resource doesn't exist |
| 405 | Method Not Allowed | HTTP method not supported |
| 409 | Conflict | Resource conflict (duplicate) |
| 422 | Unprocessable Entity | Validation/business logic error |
| 429 | Too Many Requests | Rate limit exceeded |

### Server Errors (5xx)
| Code | Name | Use Case |
|------|------|----------|
| 500 | Internal Server Error | Unexpected server error |
| 502 | Bad Gateway | Upstream service error |
| 503 | Service Unavailable | Server temporarily unavailable |
| 504 | Gateway Timeout | Upstream service timeout |

## Why

1. **Semantic Meaning**: Status codes convey meaning before clients even parse the response body.

2. **Client Behavior**: HTTP clients, browsers, and tools handle different status codes appropriately.

3. **Caching**: 2xx responses can be cached; 4xx/5xx cannot. Correct codes enable proper caching.

4. **Monitoring**: Infrastructure and APM tools use status codes to track error rates and API health.

5. **Debugging**: Correct status codes help developers quickly identify the type of issue.

6. **Standards Compliance**: Following HTTP standards ensures interoperability with tools and services.

7. **Retry Logic**: Clients can implement smart retry logic based on status codes (retry 503, don't retry 400).

---


## Consistent Error Response Format

**Impact: CRITICAL**

Inconsistent error formats force API consumers to handle multiple error structures, leading to fragile client code. A consistent error format makes APIs predictable, easier to debug, and simpler to integrate. Clients can build reusable error handling logic.

## Incorrect

```json
// ❌ Different formats across endpoints
// Endpoint A
{ "error": "Not found" }

// Endpoint B
{ "message": "Invalid email", "status": 400 }

// Endpoint C
{ "errors": ["Field required", "Invalid format"] }

// Endpoint D
{
  "success": false,
  "errorMessage": "Something went wrong"
}

// Endpoint E - just a string
"User not found"
```

**Problems:**
- Clients can't predict error structure
- Different parsing logic needed for each endpoint
- Hard to build generic error handlers
- Inconsistent developer experience

## Correct

### Standard Error Envelope

```json
// ✅ Every error follows the same structure
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable message",
    "details": [],
    "request_id": "req_abc123"
  }
}
```

### Validation Errors (422)

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The request contains invalid data",
    "details": [
      {
        "field": "email",
        "code": "INVALID_FORMAT",
        "message": "Please provide a valid email address"
      },
      {
        "field": "password",
        "code": "TOO_SHORT",
        "message": "Password must be at least 8 characters",
        "meta": {
          "min_length": 8,
          "actual_length": 5
        }
      },
      {
        "field": "age",
        "code": "OUT_OF_RANGE",
        "message": "Age must be between 18 and 120",
        "meta": {
          "min": 18,
          "max": 120,
          "actual": 15
        }
      }
    ],
    "request_id": "req_abc123"
  }
}
```

### Not Found (404)

```json
{
  "error": {
    "code": "NOT_FOUND",
    "message": "User with ID 'usr_123' not found",
    "details": [
      {
        "resource": "user",
        "field": "id",
        "value": "usr_123"
      }
    ],
    "request_id": "req_def456"
  }
}
```

### Authentication Error (401)

```json
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Authentication required",
    "details": [
      {
        "code": "TOKEN_EXPIRED",
        "message": "Your access token has expired"
      }
    ],
    "request_id": "req_ghi789"
  }
}
```

### Authorization Error (403)

```json
{
  "error": {
    "code": "FORBIDDEN",
    "message": "You don't have permission to access this resource",
    "details": [
      {
        "resource": "order",
        "action": "delete",
        "reason": "Only admins can delete orders"
      }
    ],
    "request_id": "req_jkl012"
  }
}
```

### Conflict Error (409)

```json
{
  "error": {
    "code": "CONFLICT",
    "message": "A user with this email already exists",
    "details": [
      {
        "field": "email",
        "code": "DUPLICATE",
        "value": "john@example.com"
      }
    ],
    "request_id": "req_mno345"
  }
}
```

### Rate Limit Error (429)

```json
{
  "error": {
    "code": "RATE_LIMITED",
    "message": "Too many requests. Please retry after 60 seconds.",
    "details": [
      {
        "limit": 100,
        "window": "1 minute",
        "retry_after": 60
      }
    ],
    "request_id": "req_pqr678"
  }
}
```

### Server Error (500)

```json
{
  "error": {
    "code": "INTERNAL_ERROR",
    "message": "An unexpected error occurred. Please try again later.",
    "request_id": "req_stu901"
  }
}
```

**Note:** Never expose stack traces or internal details in production.

## Implementation

### TypeScript/Node.js

```typescript
// Error classes
abstract class AppError extends Error {
  abstract readonly code: string;
  abstract readonly statusCode: number;
  readonly details: ErrorDetail[];

  constructor(message: string, details: ErrorDetail[] = []) {
    super(message);
    this.details = details;
  }

  toJSON() {
    return {
      error: {
        code: this.code,
        message: this.message,
        details: this.details.length > 0 ? this.details : undefined,
      }
    };
  }
}

class ValidationError extends AppError {
  readonly code = 'VALIDATION_ERROR';
  readonly statusCode = 422;
}

class NotFoundError extends AppError {
  readonly code = 'NOT_FOUND';
  readonly statusCode = 404;
}

class UnauthorizedError extends AppError {
  readonly code = 'UNAUTHORIZED';
  readonly statusCode = 401;
}

// Error handler middleware
function errorHandler(err, req, res, next) {
  const requestId = req.id || generateRequestId();

  if (err instanceof AppError) {
    return res.status(err.statusCode).json({
      error: {
        ...err.toJSON().error,
        request_id: requestId,
      }
    });
  }

  // Log unexpected errors
  logger.error('Unexpected error', { error: err, requestId });

  // Generic response for unknown errors
  res.status(500).json({
    error: {
      code: 'INTERNAL_ERROR',
      message: 'An unexpected error occurred',
      request_id: requestId,
    }
  });
}
```

### Laravel/PHP

```php
<?php

// app/Exceptions/AppException.php
abstract class AppException extends Exception
{
    abstract public function getErrorCode(): string;
    abstract public function getStatusCode(): int;

    protected array $details = [];

    public function setDetails(array $details): self
    {
        $this->details = $details;
        return $this;
    }

    public function render(): JsonResponse
    {
        return response()->json([
            'error' => [
                'code' => $this->getErrorCode(),
                'message' => $this->getMessage(),
                'details' => $this->details ?: null,
                'request_id' => request()->id(),
            ],
        ], $this->getStatusCode());
    }
}

class ValidationException extends AppException
{
    public function getErrorCode(): string
    {
        return 'VALIDATION_ERROR';
    }

    public function getStatusCode(): int
    {
        return 422;
    }

    public static function fromValidator(Validator $validator): self
    {
        $details = [];
        foreach ($validator->errors()->toArray() as $field => $messages) {
            $details[] = [
                'field' => $field,
                'code' => 'INVALID',
                'message' => $messages[0],
            ];
        }

        return (new self('The request contains invalid data'))
            ->setDetails($details);
    }
}

// Handler
class Handler extends ExceptionHandler
{
    public function render($request, Throwable $e)
    {
        if ($e instanceof AppException) {
            return $e->render();
        }

        if ($e instanceof ModelNotFoundException) {
            return response()->json([
                'error' => [
                    'code' => 'NOT_FOUND',
                    'message' => 'Resource not found',
                    'request_id' => $request->id(),
                ],
            ], 404);
        }

        // Log and return generic error
        Log::error($e->getMessage(), ['exception' => $e]);

        return response()->json([
            'error' => [
                'code' => 'INTERNAL_ERROR',
                'message' => 'An unexpected error occurred',
                'request_id' => $request->id(),
            ],
        ], 500);
    }
}
```

## Error Code Naming

```
// ✅ Use SCREAMING_SNAKE_CASE
VALIDATION_ERROR
NOT_FOUND
UNAUTHORIZED
FORBIDDEN
RATE_LIMITED
INTERNAL_ERROR

// ✅ Be specific
INVALID_EMAIL_FORMAT
PASSWORD_TOO_SHORT
DUPLICATE_EMAIL
TOKEN_EXPIRED
INSUFFICIENT_FUNDS

// ❌ Avoid vague codes
ERROR
FAILED
BAD_REQUEST
```

## Benefits

- Predictable API behavior
- Reusable client-side error handling
- Easier debugging with request IDs
- Clear error codes for programmatic handling
- Human-readable messages for display
- Detailed validation feedback

---


## Use Machine-Readable Error Codes

Include standardized, machine-readable error codes alongside human-readable messages to enable programmatic error handling.

## Bad Example

```json
// Anti-pattern: Only human-readable messages
{
  "error": "The user was not found"
}

// Anti-pattern: HTTP status codes only
{
  "status": 404
}

// Anti-pattern: Inconsistent or vague codes
{
  "error_code": "ERR001"
}

{
  "code": 1234
}

{
  "error_type": "bad_thing_happened"
}
```

```javascript
// No error codes for programmatic handling
app.get('/users/:id', async (req, res) => {
  const user = await db.findUser(req.params.id);
  if (!user) {
    // Client can't reliably detect "not found" vs other 404s
    res.status(404).json({ message: 'User not found' });
  }
});
```

## Good Example

```javascript
// Define error codes as constants
const ErrorCodes = {
  // Authentication & Authorization
  AUTH_TOKEN_MISSING: 'auth_token_missing',
  AUTH_TOKEN_INVALID: 'auth_token_invalid',
  AUTH_TOKEN_EXPIRED: 'auth_token_expired',
  AUTH_INSUFFICIENT_PERMISSIONS: 'auth_insufficient_permissions',

  // Validation
  VALIDATION_ERROR: 'validation_error',
  VALIDATION_REQUIRED_FIELD: 'validation_required_field',
  VALIDATION_INVALID_FORMAT: 'validation_invalid_format',
  VALIDATION_OUT_OF_RANGE: 'validation_out_of_range',

  // Resources
  RESOURCE_NOT_FOUND: 'resource_not_found',
  RESOURCE_ALREADY_EXISTS: 'resource_already_exists',
  RESOURCE_CONFLICT: 'resource_conflict',
  RESOURCE_DELETED: 'resource_deleted',

  // Rate Limiting
  RATE_LIMIT_EXCEEDED: 'rate_limit_exceeded',

  // Business Logic
  INSUFFICIENT_FUNDS: 'insufficient_funds',
  INVENTORY_UNAVAILABLE: 'inventory_unavailable',
  ORDER_CANNOT_BE_CANCELLED: 'order_cannot_be_cancelled',
  SUBSCRIPTION_EXPIRED: 'subscription_expired',

  // Server Errors
  INTERNAL_ERROR: 'internal_error',
  SERVICE_UNAVAILABLE: 'service_unavailable',
  DEPENDENCY_ERROR: 'dependency_error'
};

// Error factory
class APIError extends Error {
  constructor(code, message, statusCode = 400, details = null) {
    super(message);
    this.code = code;
    this.statusCode = statusCode;
    this.details = details;
  }
}

// Usage in routes
app.get('/users/:id', async (req, res, next) => {
  try {
    const user = await db.findUser(req.params.id);
    if (!user) {
      throw new APIError(
        ErrorCodes.RESOURCE_NOT_FOUND,
        'User not found',
        404,
        { resourceType: 'user', resourceId: req.params.id }
      );
    }
    res.json(user);
  } catch (error) {
    next(error);
  }
});

app.post('/orders', async (req, res, next) => {
  try {
    const product = await db.findProduct(req.body.productId);

    if (product.stock < req.body.quantity) {
      throw new APIError(
        ErrorCodes.INVENTORY_UNAVAILABLE,
        'Not enough items in stock',
        422,
        {
          requested: req.body.quantity,
          available: product.stock,
          productId: req.body.productId
        }
      );
    }

    // Process order...
  } catch (error) {
    next(error);
  }
});

// Error handler
app.use((err, req, res, next) => {
  if (err instanceof APIError) {
    return res.status(err.statusCode).json({
      error: {
        code: err.code,
        message: err.message,
        details: err.details
      }
    });
  }

  // Unknown error
  res.status(500).json({
    error: {
      code: ErrorCodes.INTERNAL_ERROR,
      message: 'An unexpected error occurred'
    }
  });
});
```

```python
# Python with error codes
from enum import Enum
from fastapi import FastAPI, HTTPException
from typing import Optional, Any

class ErrorCode(str, Enum):
    # Authentication
    AUTH_TOKEN_MISSING = "auth_token_missing"
    AUTH_TOKEN_INVALID = "auth_token_invalid"
    AUTH_TOKEN_EXPIRED = "auth_token_expired"
    AUTH_INSUFFICIENT_PERMISSIONS = "auth_insufficient_permissions"

    # Validation
    VALIDATION_ERROR = "validation_error"
    VALIDATION_REQUIRED_FIELD = "validation_required_field"

    # Resources
    RESOURCE_NOT_FOUND = "resource_not_found"
    RESOURCE_ALREADY_EXISTS = "resource_already_exists"
    RESOURCE_CONFLICT = "resource_conflict"

    # Business Logic
    INSUFFICIENT_FUNDS = "insufficient_funds"
    INVENTORY_UNAVAILABLE = "inventory_unavailable"

    # Server
    INTERNAL_ERROR = "internal_error"
    SERVICE_UNAVAILABLE = "service_unavailable"

class APIError(Exception):
    def __init__(
        self,
        code: ErrorCode,
        message: str,
        status_code: int = 400,
        details: Optional[dict] = None
    ):
        self.code = code
        self.message = message
        self.status_code = status_code
        self.details = details

app = FastAPI()

@app.exception_handler(APIError)
async def api_error_handler(request, exc: APIError):
    return JSONResponse(
        status_code=exc.status_code,
        content={
            "error": {
                "code": exc.code.value,
                "message": exc.message,
                "details": exc.details
            }
        }
    )

@app.get("/users/{user_id}")
async def get_user(user_id: int):
    user = await db.get_user(user_id)
    if not user:
        raise APIError(
            code=ErrorCode.RESOURCE_NOT_FOUND,
            message=f"User with ID {user_id} not found",
            status_code=404,
            details={"resource_type": "user", "resource_id": user_id}
        )
    return user
```

```json
// Error response with code
{
  "error": {
    "code": "inventory_unavailable",
    "message": "Not enough items in stock to fulfill your order",
    "details": {
      "productId": "prod_123",
      "productName": "Widget Pro",
      "requested": 10,
      "available": 3
    }
  }
}
```

```typescript
// Client-side error handling
async function createOrder(orderData: OrderData): Promise<Order> {
  const response = await fetch('/api/orders', {
    method: 'POST',
    body: JSON.stringify(orderData)
  });

  if (!response.ok) {
    const error = await response.json();

    // Programmatic handling based on error code
    switch (error.error.code) {
      case 'inventory_unavailable':
        showInventoryWarning(error.error.details);
        break;

      case 'insufficient_funds':
        redirectToPaymentUpdate();
        break;

      case 'auth_token_expired':
        await refreshToken();
        return createOrder(orderData); // Retry

      case 'rate_limit_exceeded':
        await delay(error.error.details.retryAfter * 1000);
        return createOrder(orderData); // Retry

      default:
        showGenericError(error.error.message);
    }

    throw new APIError(error);
  }

  return response.json();
}
```

## Error Code Naming Conventions

| Category | Pattern | Examples |
|----------|---------|----------|
| Auth | `auth_*` | `auth_token_expired`, `auth_invalid_credentials` |
| Validation | `validation_*` | `validation_error`, `validation_invalid_email` |
| Resource | `resource_*` | `resource_not_found`, `resource_conflict` |
| Business | Domain-specific | `insufficient_funds`, `inventory_unavailable` |
| Rate Limit | `rate_limit_*` | `rate_limit_exceeded` |
| Server | `internal_*` or `service_*` | `internal_error`, `service_unavailable` |

## Why

1. **Programmatic Handling**: Code can switch on error codes to take appropriate action.

2. **Stability**: Error codes remain stable even when messages change or are localized.

3. **Documentation**: Error codes can be documented and referenced in API docs.

4. **Monitoring**: Error codes enable precise alerting and dashboards.

5. **Client Logic**: Clients can implement specific recovery strategies per error type.

6. **Testing**: Tests can assert on specific error codes.

7. **Internationalization**: Messages can be translated while codes stay constant.

---


## Provide Meaningful Error Messages

Error messages should be clear, actionable, and help users understand what went wrong and how to fix it.

## Bad Example

```json
// Anti-pattern: Vague or unhelpful messages
{
  "error": "Error"
}

{
  "error": "Bad request"
}

{
  "error": "Invalid input"
}

{
  "error": "Something went wrong"
}

{
  "error": "null"
}

{
  "error": "Error code: 0x8004005"
}

// Technical jargon users can't understand
{
  "error": "SQLITE_CONSTRAINT_FOREIGNKEY"
}

{
  "error": "MongoServerError: E11000 duplicate key error"
}
```

```javascript
// Unhelpful error responses
app.post('/users', async (req, res) => {
  try {
    await db.createUser(req.body);
  } catch (error) {
    res.status(400).json({ error: 'Bad request' }); // What's bad about it?
  }
});
```

## Good Example

```javascript
// Clear, actionable error messages
const errorMessages = {
  email_required: 'Email address is required to create an account',
  email_invalid: 'Please provide a valid email address (e.g., user@example.com)',
  email_taken: 'An account with this email already exists. Try signing in instead',
  password_weak: 'Password must be at least 8 characters with one uppercase, one lowercase, and one number',
  rate_limited: 'Too many requests. Please wait 60 seconds before trying again',
  resource_not_found: 'The requested user could not be found. It may have been deleted',
  permission_denied: 'You do not have permission to access this resource. Contact your administrator',
  payment_failed: 'Your payment could not be processed. Please check your card details and try again'
};

app.post('/users', async (req, res, next) => {
  try {
    const { email, password, name } = req.body;

    // Specific validation messages
    if (!email) {
      return res.status(400).json({
        error: {
          code: 'validation_error',
          message: 'Email address is required to create an account',
          field: 'email'
        }
      });
    }

    if (!isValidEmail(email)) {
      return res.status(400).json({
        error: {
          code: 'validation_error',
          message: 'Please provide a valid email address (e.g., user@example.com)',
          field: 'email',
          provided: email
        }
      });
    }

    const existingUser = await db.findUserByEmail(email);
    if (existingUser) {
      return res.status(409).json({
        error: {
          code: 'resource_conflict',
          message: 'An account with this email already exists. Try signing in or resetting your password',
          field: 'email',
          links: {
            signin: '/auth/signin',
            passwordReset: '/auth/password-reset'
          }
        }
      });
    }

    const user = await db.createUser({ email, password, name });
    res.status(201).json(user);

  } catch (error) {
    next(error);
  }
});
```

```python
# FastAPI with meaningful errors
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, validator, EmailStr

app = FastAPI()

class UserCreate(BaseModel):
    email: EmailStr
    password: str
    name: str

    @validator('password')
    def password_strength(cls, v):
        if len(v) < 8:
            raise ValueError(
                'Password must be at least 8 characters long. '
                'Strong passwords help protect your account.'
            )
        if not any(c.isupper() for c in v):
            raise ValueError(
                'Password must contain at least one uppercase letter (A-Z)'
            )
        if not any(c.islower() for c in v):
            raise ValueError(
                'Password must contain at least one lowercase letter (a-z)'
            )
        if not any(c.isdigit() for c in v):
            raise ValueError(
                'Password must contain at least one number (0-9)'
            )
        return v

    @validator('name')
    def name_not_empty(cls, v):
        if not v or not v.strip():
            raise ValueError(
                'Name is required. Please enter your full name as you would '
                'like it to appear on your profile.'
            )
        if len(v) > 100:
            raise ValueError(
                f'Name must be 100 characters or less. You entered {len(v)} characters.'
            )
        return v.strip()

@app.post("/users")
async def create_user(user: UserCreate):
    existing = await db.find_user_by_email(user.email)
    if existing:
        raise HTTPException(
            status_code=409,
            detail={
                "code": "email_already_registered",
                "message": (
                    f"The email '{user.email}' is already registered. "
                    "If this is your email, try signing in or resetting your password."
                ),
                "suggestions": [
                    "Sign in with your existing account",
                    "Reset your password if you forgot it",
                    "Use a different email address"
                ]
            }
        )
    return await db.create_user(user)
```

```json
// Good error response examples

// Validation error with guidance
{
  "error": {
    "code": "validation_error",
    "message": "The email address format is invalid",
    "details": [
      {
        "field": "email",
        "message": "Please provide a valid email address (e.g., user@example.com)",
        "provided": "not-an-email",
        "suggestion": "Check for typos and ensure the email includes an @ symbol"
      }
    ]
  }
}

// Resource not found with context
{
  "error": {
    "code": "resource_not_found",
    "message": "Order #12345 could not be found",
    "details": [
      {
        "message": "This order may have been deleted or the ID may be incorrect",
        "suggestions": [
          "Verify the order ID is correct",
          "Check your order history for the correct ID",
          "The order may have been archived after 90 days"
        ]
      }
    ]
  }
}

// Permission error with next steps
{
  "error": {
    "code": "permission_denied",
    "message": "You don't have permission to delete this project",
    "details": [
      {
        "message": "Only project owners and administrators can delete projects",
        "currentRole": "member",
        "requiredRoles": ["owner", "admin"],
        "suggestion": "Contact the project owner to request deletion or elevated permissions"
      }
    ]
  }
}

// Rate limit with retry info
{
  "error": {
    "code": "rate_limit_exceeded",
    "message": "You've made too many requests. Please slow down.",
    "details": [
      {
        "limit": 100,
        "window": "1 minute",
        "retryAfter": 45,
        "message": "You can make another request in 45 seconds"
      }
    ]
  }
}
```

## Why

1. **User Experience**: Clear messages help users fix problems without contacting support.

2. **Reduced Support Burden**: Self-explanatory errors mean fewer support tickets.

3. **Developer Productivity**: API consumers can debug issues faster with meaningful messages.

4. **Actionable Guidance**: Good messages tell users what to do next, not just what went wrong.

5. **Trust Building**: Professional error messages build confidence in your API.

6. **Accessibility**: Messages should be understandable by non-technical users when appropriate.

7. **Localization Ready**: Clear messages are easier to translate accurately.

---


## Never Expose Stack Traces in Production

Stack traces and internal error details should never be exposed to API clients in production environments, as they reveal implementation details and potential vulnerabilities.

## Bad Example

```json
// Anti-pattern: Full stack trace in production response
{
  "error": "Cannot read property 'id' of undefined",
  "stack": "TypeError: Cannot read property 'id' of undefined\n    at getUserOrders (/app/src/controllers/orders.js:45:23)\n    at Layer.handle [as handle_request] (/app/node_modules/express/lib/router/layer.js:95:5)\n    at next (/app/node_modules/express/lib/router/route.js:137:13)\n    at authenticate (/app/src/middleware/auth.js:28:5)\n    at /app/node_modules/express/lib/router/index.js:284:15"
}

// Anti-pattern: Database error details exposed
{
  "error": "SequelizeConnectionError: Connection refused to host 'db.internal.company.com' port 5432",
  "sql": "SELECT * FROM users WHERE id = 1 AND deleted_at IS NULL"
}

// Anti-pattern: Internal paths and configuration
{
  "error": "ENOENT: no such file or directory, open '/var/app/config/secrets.json'"
}
```

```javascript
// Dangerous: Exposing all error details
app.use((err, req, res, next) => {
  res.status(500).json({
    error: err.message,
    stack: err.stack,  // Never do this in production!
    code: err.code
  });
});
```

## Good Example

```javascript
const isProduction = process.env.NODE_ENV === 'production';

// Secure error handler
app.use((err, req, res, next) => {
  // Log full error internally
  logger.error('Request error', {
    error: err.message,
    stack: err.stack,
    requestId: req.id,
    path: req.path,
    method: req.method,
    userId: req.user?.id
  });

  // Determine if error is safe to expose
  const isOperationalError = err.isOperational || err.expose;
  const statusCode = err.statusCode || 500;

  // Build safe response
  const errorResponse = {
    error: {
      code: err.code || 'internal_error',
      message: isOperationalError
        ? err.message
        : 'An unexpected error occurred. Please try again later.',
      requestId: req.id
    }
  };

  // Only include details in development
  if (!isProduction && err.stack) {
    errorResponse.error._debug = {
      message: err.message,
      stack: err.stack.split('\n')
    };
  }

  res.status(statusCode).json(errorResponse);
});

// Custom error class for operational errors
class APIError extends Error {
  constructor(message, statusCode = 500, code = 'internal_error') {
    super(message);
    this.statusCode = statusCode;
    this.code = code;
    this.isOperational = true; // Safe to expose
  }
}

// Database error handling
app.get('/users/:id', async (req, res, next) => {
  try {
    const user = await db.findUser(req.params.id);
    if (!user) {
      throw new APIError('User not found', 404, 'resource_not_found');
    }
    res.json(user);
  } catch (error) {
    if (error instanceof APIError) {
      return next(error);
    }

    // Log the actual database error
    logger.error('Database error', {
      error: error.message,
      stack: error.stack,
      query: 'findUser',
      params: { id: req.params.id }
    });

    // Return generic error to client
    next(new APIError(
      'Unable to retrieve user information',
      500,
      'service_error'
    ));
  }
});
```

```python
# FastAPI with secure error handling
import logging
import traceback
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse
import os

app = FastAPI()
logger = logging.getLogger(__name__)
IS_PRODUCTION = os.getenv("ENVIRONMENT") == "production"

class APIError(Exception):
    def __init__(self, message: str, status_code: int = 500, code: str = "internal_error"):
        self.message = message
        self.status_code = status_code
        self.code = code
        self.is_operational = True

@app.exception_handler(APIError)
async def api_error_handler(request: Request, exc: APIError):
    return JSONResponse(
        status_code=exc.status_code,
        content={
            "error": {
                "code": exc.code,
                "message": exc.message,
                "request_id": request.state.request_id
            }
        }
    )

@app.exception_handler(Exception)
async def generic_error_handler(request: Request, exc: Exception):
    # Log full error internally
    logger.error(
        "Unhandled exception",
        extra={
            "error": str(exc),
            "traceback": traceback.format_exc(),
            "request_id": request.state.request_id,
            "path": request.url.path,
            "method": request.method
        }
    )

    # Return safe response
    content = {
        "error": {
            "code": "internal_error",
            "message": "An unexpected error occurred. Please try again later.",
            "request_id": request.state.request_id
        }
    }

    # Include debug info only in development
    if not IS_PRODUCTION:
        content["error"]["_debug"] = {
            "type": type(exc).__name__,
            "message": str(exc),
            "traceback": traceback.format_exc().split("\n")
        }

    return JSONResponse(status_code=500, content=content)
```

```json
// Production error response (safe)
{
  "error": {
    "code": "internal_error",
    "message": "An unexpected error occurred. Please try again later.",
    "requestId": "req-abc123"
  }
}

// Development error response (with debug info)
{
  "error": {
    "code": "internal_error",
    "message": "An unexpected error occurred. Please try again later.",
    "requestId": "req-abc123",
    "_debug": {
      "type": "TypeError",
      "message": "Cannot read property 'id' of undefined",
      "traceback": [
        "Traceback (most recent call last):",
        "  File \"app.py\", line 45, in get_user",
        "    return user.id",
        "TypeError: Cannot read property 'id' of undefined"
      ]
    }
  }
}
```

## What to Log vs. What to Return

| Information | Log Internally | Return to Client |
|-------------|----------------|------------------|
| Error message | Yes | Generic only |
| Stack trace | Yes | Never in production |
| SQL queries | Yes | Never |
| File paths | Yes | Never |
| Internal IPs | Yes | Never |
| Request ID | Yes | Yes |
| Error code | Yes | Yes |
| User ID | Yes | No |
| Timestamps | Yes | Optional |

## Why

1. **Security**: Stack traces reveal file paths, dependencies, and code structure attackers can exploit.

2. **Information Disclosure**: Internal error messages may expose database schemas, API keys, or other secrets.

3. **Attack Surface**: Knowing which frameworks and versions you use helps attackers find known vulnerabilities.

4. **Professionalism**: Clean error messages present a polished API to consumers.

5. **Compliance**: Many security standards (PCI-DSS, SOC 2) require hiding internal error details.

6. **Debugging**: Request IDs allow correlation between client reports and internal logs.

7. **Development Experience**: Debug info in development helps during development without production risk.

---


## Include Request ID in Error Responses

Every API request should have a unique identifier that appears in both the response and server logs, enabling easy correlation for debugging.

## Bad Example

```json
// Anti-pattern: No request identifier
{
  "error": {
    "code": "internal_error",
    "message": "An unexpected error occurred"
  }
}
// User reports error, but support can't find it in logs
```

```javascript
// No request tracking
app.get('/users/:id', async (req, res) => {
  try {
    const user = await db.findUser(req.params.id);
    res.json(user);
  } catch (error) {
    console.log('Error:', error.message); // No way to correlate
    res.status(500).json({ error: 'Something went wrong' });
  }
});
```

## Good Example

```javascript
const { v4: uuidv4 } = require('uuid');

// Request ID middleware
app.use((req, res, next) => {
  // Use client-provided ID or generate new one
  req.id = req.headers['x-request-id'] || uuidv4();

  // Add to response headers
  res.setHeader('X-Request-ID', req.id);

  // Add to logger context
  req.log = logger.child({ requestId: req.id });

  next();
});

// Use in routes
app.get('/users/:id', async (req, res, next) => {
  req.log.info('Fetching user', { userId: req.params.id });

  try {
    const user = await db.findUser(req.params.id);
    if (!user) {
      return res.status(404).json({
        error: {
          code: 'resource_not_found',
          message: 'User not found',
          requestId: req.id
        }
      });
    }
    res.json(user);
  } catch (error) {
    req.log.error('Failed to fetch user', {
      error: error.message,
      stack: error.stack
    });
    next(error);
  }
});

// Error handler includes request ID
app.use((err, req, res, next) => {
  req.log.error('Request failed', {
    error: err.message,
    stack: err.stack,
    statusCode: err.statusCode || 500
  });

  res.status(err.statusCode || 500).json({
    error: {
      code: err.code || 'internal_error',
      message: err.message || 'An unexpected error occurred',
      requestId: req.id
    }
  });
});
```

```python
# FastAPI with request ID
import uuid
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse
import logging
from contextvars import ContextVar

app = FastAPI()
logger = logging.getLogger(__name__)

# Context variable for request ID
request_id_var: ContextVar[str] = ContextVar("request_id", default="")

@app.middleware("http")
async def request_id_middleware(request: Request, call_next):
    # Get or generate request ID
    request_id = request.headers.get("X-Request-ID", str(uuid.uuid4()))
    request.state.request_id = request_id
    request_id_var.set(request_id)

    # Process request
    response = await call_next(request)

    # Add request ID to response
    response.headers["X-Request-ID"] = request_id
    return response

# Custom log filter to include request ID
class RequestIdFilter(logging.Filter):
    def filter(self, record):
        record.request_id = request_id_var.get("")
        return True

# Configure logging
handler = logging.StreamHandler()
handler.addFilter(RequestIdFilter())
handler.setFormatter(logging.Formatter(
    '%(asctime)s [%(request_id)s] %(levelname)s: %(message)s'
))
logger.addHandler(handler)

@app.exception_handler(Exception)
async def error_handler(request: Request, exc: Exception):
    logger.error(f"Request failed: {exc}", exc_info=True)

    return JSONResponse(
        status_code=500,
        content={
            "error": {
                "code": "internal_error",
                "message": "An unexpected error occurred",
                "requestId": request.state.request_id
            }
        },
        headers={"X-Request-ID": request.state.request_id}
    )

@app.get("/users/{user_id}")
async def get_user(user_id: int, request: Request):
    logger.info(f"Fetching user {user_id}")

    user = await db.get_user(user_id)
    if not user:
        return JSONResponse(
            status_code=404,
            content={
                "error": {
                    "code": "resource_not_found",
                    "message": f"User {user_id} not found",
                    "requestId": request.state.request_id
                }
            }
        )
    return user
```

```json
// Error response with request ID
HTTP/1.1 500 Internal Server Error
X-Request-ID: req-550e8400-e29b-41d4-a716-446655440000

{
  "error": {
    "code": "internal_error",
    "message": "An unexpected error occurred. Please try again.",
    "requestId": "req-550e8400-e29b-41d4-a716-446655440000"
  }
}
```

```bash
# Server logs with request ID
2024-01-15 10:30:00 [req-550e8400-e29b-41d4-a716-446655440000] INFO: Fetching user 123
2024-01-15 10:30:00 [req-550e8400-e29b-41d4-a716-446655440000] ERROR: Database connection timeout
2024-01-15 10:30:00 [req-550e8400-e29b-41d4-a716-446655440000] ERROR: Request failed
```

## Distributed Tracing Integration

```javascript
// Integration with OpenTelemetry
const { trace, context } = require('@opentelemetry/api');

app.use((req, res, next) => {
  const span = trace.getActiveSpan();

  // Use trace ID as request ID for distributed tracing
  if (span) {
    const traceId = span.spanContext().traceId;
    req.id = traceId;
    req.spanContext = span.spanContext();
  } else {
    req.id = uuidv4();
  }

  res.setHeader('X-Request-ID', req.id);
  next();
});
```

```yaml
# OpenAPI documentation for request ID
components:
  headers:
    X-Request-ID:
      description: Unique identifier for the request, used for debugging and log correlation
      schema:
        type: string
        format: uuid
      example: "550e8400-e29b-41d4-a716-446655440000"

  schemas:
    Error:
      type: object
      properties:
        error:
          type: object
          properties:
            code:
              type: string
            message:
              type: string
            requestId:
              type: string
              description: Unique request identifier for support correlation
```

## Why

1. **Debugging**: Users can provide the request ID when reporting issues, allowing instant log lookup.

2. **Log Correlation**: Link all log entries for a single request across multiple services.

3. **Distributed Tracing**: Request IDs propagate through microservices for end-to-end tracing.

4. **Support Efficiency**: "Please provide the request ID" is faster than "describe what you did."

5. **Monitoring**: Track individual request paths through your infrastructure.

6. **Compliance**: Audit trails require the ability to trace specific requests.

7. **Client Debugging**: Client applications can include request IDs in their own logs.

---


## Include Validation Error Details

When validation fails, provide specific details about which fields failed and why, enabling clients to display targeted error messages.

## Bad Example

```json
// Anti-pattern: Single vague validation error
{
  "error": "Validation failed"
}

// Anti-pattern: List without field association
{
  "errors": [
    "Invalid email",
    "Password too short",
    "Name required"
  ]
}

// Anti-pattern: Boolean flags without messages
{
  "valid": false,
  "emailValid": false,
  "passwordValid": false
}
```

```javascript
// Unhelpful validation response
app.post('/users', (req, res) => {
  const errors = validate(req.body);
  if (errors.length > 0) {
    res.status(400).json({ error: 'Validation failed' });
  }
});
```

## Good Example

```javascript
// Detailed validation errors
const { body, validationResult } = require('express-validator');

const validateUser = [
  body('email')
    .notEmpty().withMessage('Email is required')
    .isEmail().withMessage('Must be a valid email address')
    .normalizeEmail(),

  body('password')
    .notEmpty().withMessage('Password is required')
    .isLength({ min: 8 }).withMessage('Password must be at least 8 characters')
    .matches(/[A-Z]/).withMessage('Password must contain an uppercase letter')
    .matches(/[a-z]/).withMessage('Password must contain a lowercase letter')
    .matches(/[0-9]/).withMessage('Password must contain a number'),

  body('age')
    .optional()
    .isInt({ min: 0, max: 150 }).withMessage('Age must be between 0 and 150'),

  body('username')
    .notEmpty().withMessage('Username is required')
    .isLength({ min: 3, max: 30 }).withMessage('Username must be 3-30 characters')
    .matches(/^[a-zA-Z0-9_]+$/).withMessage('Username can only contain letters, numbers, and underscores')
];

app.post('/users', validateUser, (req, res) => {
  const errors = validationResult(req);

  if (!errors.isEmpty()) {
    return res.status(400).json({
      error: {
        code: 'validation_error',
        message: 'One or more fields have invalid values',
        details: errors.array().map(err => ({
          field: err.path,
          message: err.msg,
          value: err.value,
          location: err.location  // body, query, params
        }))
      }
    });
  }

  // Create user...
});
```

```json
// Detailed validation error response
{
  "error": {
    "code": "validation_error",
    "message": "One or more fields have invalid values",
    "details": [
      {
        "field": "email",
        "message": "Must be a valid email address",
        "value": "not-an-email",
        "location": "body"
      },
      {
        "field": "password",
        "message": "Password must be at least 8 characters",
        "value": "short",
        "location": "body",
        "constraints": {
          "minLength": 8,
          "actualLength": 5
        }
      },
      {
        "field": "age",
        "message": "Age must be between 0 and 150",
        "value": -5,
        "location": "body",
        "constraints": {
          "min": 0,
          "max": 150
        }
      }
    ]
  }
}
```

```python
# FastAPI with detailed validation
from fastapi import FastAPI, HTTPException, Request
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse
from pydantic import BaseModel, EmailStr, validator, Field
from typing import Optional, List

app = FastAPI()

class UserCreate(BaseModel):
    email: EmailStr
    password: str = Field(..., min_length=8, max_length=100)
    username: str = Field(..., min_length=3, max_length=30, regex=r'^[a-zA-Z0-9_]+$')
    age: Optional[int] = Field(None, ge=0, le=150)

    @validator('password')
    def password_complexity(cls, v):
        errors = []
        if not any(c.isupper() for c in v):
            errors.append('must contain an uppercase letter')
        if not any(c.islower() for c in v):
            errors.append('must contain a lowercase letter')
        if not any(c.isdigit() for c in v):
            errors.append('must contain a number')
        if errors:
            raise ValueError(f"Password {', '.join(errors)}")
        return v

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request: Request, exc: RequestValidationError):
    details = []
    for error in exc.errors():
        field = '.'.join(str(loc) for loc in error['loc'] if loc != 'body')
        details.append({
            'field': field,
            'message': error['msg'],
            'type': error['type'],
            'context': error.get('ctx', {})
        })

    return JSONResponse(
        status_code=422,
        content={
            'error': {
                'code': 'validation_error',
                'message': f'{len(details)} validation error(s) found',
                'details': details
            }
        }
    )

@app.post("/users")
async def create_user(user: UserCreate):
    return {"id": 1, **user.dict()}
```

```typescript
// TypeScript/Zod validation with detailed errors
import { z } from 'zod';
import express from 'express';

const UserSchema = z.object({
  email: z.string()
    .min(1, 'Email is required')
    .email('Must be a valid email address'),

  password: z.string()
    .min(8, 'Password must be at least 8 characters')
    .regex(/[A-Z]/, 'Password must contain an uppercase letter')
    .regex(/[a-z]/, 'Password must contain a lowercase letter')
    .regex(/[0-9]/, 'Password must contain a number'),

  username: z.string()
    .min(3, 'Username must be at least 3 characters')
    .max(30, 'Username cannot exceed 30 characters')
    .regex(/^[a-zA-Z0-9_]+$/, 'Username can only contain letters, numbers, and underscores'),

  age: z.number()
    .int('Age must be a whole number')
    .min(0, 'Age cannot be negative')
    .max(150, 'Age cannot exceed 150')
    .optional()
});

app.post('/users', (req, res) => {
  const result = UserSchema.safeParse(req.body);

  if (!result.success) {
    return res.status(400).json({
      error: {
        code: 'validation_error',
        message: 'Validation failed',
        details: result.error.errors.map(err => ({
          field: err.path.join('.'),
          message: err.message,
          code: err.code
        }))
      }
    });
  }

  // Create user with result.data
});
```

## Nested Object Validation

```json
// Validation errors for nested objects
{
  "error": {
    "code": "validation_error",
    "message": "Validation failed",
    "details": [
      {
        "field": "address.zipCode",
        "message": "ZIP code must be 5 digits",
        "value": "123"
      },
      {
        "field": "address.country",
        "message": "Country is required"
      },
      {
        "field": "contacts[0].email",
        "message": "Invalid email format",
        "value": "bad-email"
      },
      {
        "field": "contacts[1].phone",
        "message": "Phone number must include country code",
        "value": "555-1234"
      }
    ]
  }
}
```

## Why

1. **Field-Level Feedback**: Clients can highlight specific form fields with their errors.

2. **User Experience**: Users see exactly which fields need attention without guessing.

3. **Efficient Debugging**: Developers can quickly identify validation issues during development.

4. **Batch Correction**: Users can fix all issues at once instead of submitting multiple times.

5. **Localization**: Field-specific messages can be translated appropriately.

6. **Form Libraries**: Frontend validation libraries can map errors directly to form fields.

7. **API Contract**: Clear validation responses help document expected input formats.

---


## Implement Secure Authentication

Use industry-standard authentication mechanisms like OAuth 2.0, JWT, or API keys with proper security practices.

## Bad Example

```javascript
// Anti-pattern: Basic auth over HTTP
app.use((req, res, next) => {
  const auth = req.headers.authorization;
  const [user, pass] = Buffer.from(auth.split(' ')[1], 'base64')
    .toString().split(':');
  // Credentials sent in plain text!
  if (user === 'admin' && pass === 'password123') {
    next();
  }
});

// Anti-pattern: Token in URL
app.get('/users?token=secret123', (req, res) => {
  // Token visible in logs, browser history, referrer headers
});

// Anti-pattern: No token expiration
const token = jwt.sign({ userId: 123 }); // No expiration!

// Anti-pattern: Weak secret
const token = jwt.sign({ userId: 123 }, 'secret'); // Easily guessable
```

```json
// Anti-pattern: Credentials in response body
{
  "user": {
    "id": 123,
    "password": "hashedpassword",
    "apiKey": "sk_live_abc123"
  }
}
```

## Good Example

```javascript
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');

// Environment-based secrets
const JWT_SECRET = process.env.JWT_SECRET; // Long, random string
const JWT_EXPIRES_IN = '15m'; // Short-lived access tokens
const REFRESH_EXPIRES_IN = '7d';

// Password hashing
async function hashPassword(password) {
  const saltRounds = 12;
  return bcrypt.hash(password, saltRounds);
}

async function verifyPassword(password, hash) {
  return bcrypt.compare(password, hash);
}

// JWT token generation
function generateTokens(user) {
  const accessToken = jwt.sign(
    {
      sub: user.id,
      email: user.email,
      roles: user.roles
    },
    JWT_SECRET,
    {
      expiresIn: JWT_EXPIRES_IN,
      issuer: 'api.example.com',
      audience: 'example.com'
    }
  );

  const refreshToken = jwt.sign(
    { sub: user.id, type: 'refresh' },
    JWT_SECRET,
    { expiresIn: REFRESH_EXPIRES_IN }
  );

  return { accessToken, refreshToken };
}

// Authentication endpoint
app.post('/auth/login', async (req, res) => {
  const { email, password } = req.body;

  const user = await db.findUserByEmail(email);
  if (!user) {
    // Don't reveal if user exists
    return res.status(401).json({
      error: {
        code: 'invalid_credentials',
        message: 'Invalid email or password'
      }
    });
  }

  const valid = await verifyPassword(password, user.passwordHash);
  if (!valid) {
    return res.status(401).json({
      error: {
        code: 'invalid_credentials',
        message: 'Invalid email or password'
      }
    });
  }

  const tokens = generateTokens(user);

  // Set refresh token as HTTP-only cookie
  res.cookie('refreshToken', tokens.refreshToken, {
    httpOnly: true,
    secure: true,
    sameSite: 'strict',
    maxAge: 7 * 24 * 60 * 60 * 1000
  });

  res.json({
    accessToken: tokens.accessToken,
    expiresIn: 900, // 15 minutes in seconds
    tokenType: 'Bearer'
  });
});

// JWT verification middleware
function authenticate(req, res, next) {
  const authHeader = req.headers.authorization;

  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({
      error: {
        code: 'auth_token_missing',
        message: 'Authorization header with Bearer token required'
      }
    });
  }

  const token = authHeader.slice(7);

  try {
    const payload = jwt.verify(token, JWT_SECRET, {
      issuer: 'api.example.com',
      audience: 'example.com'
    });

    req.user = payload;
    next();
  } catch (error) {
    if (error.name === 'TokenExpiredError') {
      return res.status(401).json({
        error: {
          code: 'auth_token_expired',
          message: 'Access token has expired. Please refresh.'
        }
      });
    }

    return res.status(401).json({
      error: {
        code: 'auth_token_invalid',
        message: 'Invalid access token'
      }
    });
  }
}

// Token refresh endpoint
app.post('/auth/refresh', async (req, res) => {
  const refreshToken = req.cookies.refreshToken;

  if (!refreshToken) {
    return res.status(401).json({
      error: {
        code: 'refresh_token_missing',
        message: 'Refresh token required'
      }
    });
  }

  try {
    const payload = jwt.verify(refreshToken, JWT_SECRET);

    // Check if token is revoked
    const isRevoked = await db.isTokenRevoked(refreshToken);
    if (isRevoked) {
      return res.status(401).json({
        error: {
          code: 'refresh_token_revoked',
          message: 'Refresh token has been revoked'
        }
      });
    }

    const user = await db.findUser(payload.sub);
    const tokens = generateTokens(user);

    // Rotate refresh token
    await db.revokeToken(refreshToken);

    res.cookie('refreshToken', tokens.refreshToken, {
      httpOnly: true,
      secure: true,
      sameSite: 'strict',
      maxAge: 7 * 24 * 60 * 60 * 1000
    });

    res.json({
      accessToken: tokens.accessToken,
      expiresIn: 900
    });
  } catch (error) {
    return res.status(401).json({
      error: {
        code: 'refresh_token_invalid',
        message: 'Invalid refresh token'
      }
    });
  }
});

// Protected route
app.get('/users/me', authenticate, (req, res) => {
  res.json({ userId: req.user.sub });
});
```

```python
# FastAPI with OAuth2 and JWT
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from jose import JWTError, jwt
from passlib.context import CryptContext
from datetime import datetime, timedelta
import os

app = FastAPI()

SECRET_KEY = os.getenv("JWT_SECRET")
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 15

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="auth/token")

def create_access_token(data: dict, expires_delta: timedelta = None):
    to_encode = data.copy()
    expire = datetime.utcnow() + (expires_delta or timedelta(minutes=15))
    to_encode.update({"exp": expire, "iss": "api.example.com"})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

async def get_current_user(token: str = Depends(oauth2_scheme)):
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail={"code": "auth_token_invalid", "message": "Invalid token"},
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        user_id: str = payload.get("sub")
        if user_id is None:
            raise credentials_exception
    except JWTError:
        raise credentials_exception

    user = await db.get_user(user_id)
    if user is None:
        raise credentials_exception
    return user

@app.post("/auth/token")
async def login(form_data: OAuth2PasswordRequestForm = Depends()):
    user = await authenticate_user(form_data.username, form_data.password)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail={"code": "invalid_credentials", "message": "Invalid credentials"}
        )

    access_token = create_access_token(
        data={"sub": str(user.id)},
        expires_delta=timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    )
    return {"access_token": access_token, "token_type": "bearer"}

@app.get("/users/me")
async def read_users_me(current_user = Depends(get_current_user)):
    return current_user
```

## Why

1. **Data Protection**: Proper authentication protects sensitive user data and operations.

2. **Token Security**: Short-lived tokens and refresh rotation limit damage from token theft.

3. **Password Safety**: Bcrypt hashing protects passwords even if the database is compromised.

4. **Standards Compliance**: OAuth 2.0/JWT are well-understood and widely supported.

5. **Secure Transmission**: HTTP-only cookies prevent XSS attacks on refresh tokens.

6. **Audit Trail**: Token-based auth provides clear user identification for logging.

7. **Scalability**: Stateless JWT tokens work well in distributed systems.

---


## Implement Proper Authorization

Authorization verifies what authenticated users can do. Implement role-based (RBAC) or attribute-based (ABAC) access control consistently.

## Bad Example

```javascript
// Anti-pattern: No authorization checks
app.delete('/users/:id', authenticate, async (req, res) => {
  // Anyone authenticated can delete any user!
  await db.deleteUser(req.params.id);
  res.status(204).send();
});

// Anti-pattern: Client-side only authorization
app.get('/admin/users', authenticate, async (req, res) => {
  // Relies on frontend hiding the button
  const users = await db.getAllUsers();
  res.json(users);
});

// Anti-pattern: Inconsistent checks
app.get('/documents/:id', async (req, res) => {
  const doc = await db.findDocument(req.params.id);
  // Sometimes checks, sometimes doesn't
  if (doc.isPublic) {
    res.json(doc);
  }
  // Private docs accessible without check!
  res.json(doc);
});
```

## Good Example

```javascript
// Role-Based Access Control (RBAC)
const ROLES = {
  ADMIN: 'admin',
  MANAGER: 'manager',
  USER: 'user'
};

const PERMISSIONS = {
  // Resource: action -> roles that can perform it
  users: {
    read: [ROLES.ADMIN, ROLES.MANAGER, ROLES.USER],
    create: [ROLES.ADMIN, ROLES.MANAGER],
    update: [ROLES.ADMIN, ROLES.MANAGER],
    delete: [ROLES.ADMIN]
  },
  reports: {
    read: [ROLES.ADMIN, ROLES.MANAGER],
    create: [ROLES.ADMIN, ROLES.MANAGER],
    delete: [ROLES.ADMIN]
  },
  settings: {
    read: [ROLES.ADMIN],
    update: [ROLES.ADMIN]
  }
};

// Authorization middleware
function authorize(resource, action) {
  return (req, res, next) => {
    const userRoles = req.user.roles || [];
    const allowedRoles = PERMISSIONS[resource]?.[action] || [];

    const hasPermission = userRoles.some(role =>
      allowedRoles.includes(role)
    );

    if (!hasPermission) {
      return res.status(403).json({
        error: {
          code: 'forbidden',
          message: `You don't have permission to ${action} ${resource}`,
          requiredRoles: allowedRoles,
          yourRoles: userRoles
        }
      });
    }

    next();
  };
}

// Resource ownership check
async function authorizeOwnership(req, res, next) {
  const resourceId = req.params.id;
  const userId = req.user.sub;

  const resource = await db.findResource(resourceId);

  if (!resource) {
    return res.status(404).json({
      error: { code: 'not_found', message: 'Resource not found' }
    });
  }

  // Admin can access anything
  if (req.user.roles.includes(ROLES.ADMIN)) {
    req.resource = resource;
    return next();
  }

  // Owner can access their own resources
  if (resource.ownerId !== userId) {
    return res.status(403).json({
      error: {
        code: 'forbidden',
        message: 'You can only access your own resources'
      }
    });
  }

  req.resource = resource;
  next();
}

// Usage
app.get('/users',
  authenticate,
  authorize('users', 'read'),
  async (req, res) => {
    const users = await db.getUsers();
    res.json(users);
  }
);

app.delete('/users/:id',
  authenticate,
  authorize('users', 'delete'),
  async (req, res) => {
    await db.deleteUser(req.params.id);
    res.status(204).send();
  }
);

app.get('/documents/:id',
  authenticate,
  authorizeOwnership,
  async (req, res) => {
    res.json(req.resource);
  }
);

app.put('/documents/:id',
  authenticate,
  authorizeOwnership,
  async (req, res) => {
    const updated = await db.updateDocument(req.params.id, req.body);
    res.json(updated);
  }
);
```

```python
# FastAPI with RBAC
from fastapi import FastAPI, Depends, HTTPException, status
from enum import Enum
from typing import List
from functools import wraps

app = FastAPI()

class Role(str, Enum):
    ADMIN = "admin"
    MANAGER = "manager"
    USER = "user"

class Permission(str, Enum):
    READ_USERS = "read:users"
    WRITE_USERS = "write:users"
    DELETE_USERS = "delete:users"
    READ_REPORTS = "read:reports"
    ADMIN_SETTINGS = "admin:settings"

ROLE_PERMISSIONS = {
    Role.ADMIN: [
        Permission.READ_USERS,
        Permission.WRITE_USERS,
        Permission.DELETE_USERS,
        Permission.READ_REPORTS,
        Permission.ADMIN_SETTINGS
    ],
    Role.MANAGER: [
        Permission.READ_USERS,
        Permission.WRITE_USERS,
        Permission.READ_REPORTS
    ],
    Role.USER: [
        Permission.READ_USERS
    ]
}

def get_user_permissions(user) -> List[Permission]:
    permissions = set()
    for role in user.roles:
        permissions.update(ROLE_PERMISSIONS.get(role, []))
    return list(permissions)

def require_permission(permission: Permission):
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, current_user = Depends(get_current_user), **kwargs):
            user_permissions = get_user_permissions(current_user)

            if permission not in user_permissions:
                raise HTTPException(
                    status_code=status.HTTP_403_FORBIDDEN,
                    detail={
                        "code": "forbidden",
                        "message": f"Permission '{permission.value}' required",
                        "your_permissions": [p.value for p in user_permissions]
                    }
                )
            return await func(*args, current_user=current_user, **kwargs)
        return wrapper
    return decorator

def require_ownership_or_admin(resource_type: str):
    async def check_ownership(
        resource_id: int,
        current_user = Depends(get_current_user)
    ):
        resource = await db.get_resource(resource_type, resource_id)

        if not resource:
            raise HTTPException(status_code=404, detail="Not found")

        if Role.ADMIN in current_user.roles:
            return resource

        if resource.owner_id != current_user.id:
            raise HTTPException(
                status_code=403,
                detail={
                    "code": "forbidden",
                    "message": "You can only access your own resources"
                }
            )
        return resource
    return check_ownership

@app.get("/users")
@require_permission(Permission.READ_USERS)
async def list_users(current_user = Depends(get_current_user)):
    return await db.get_users()

@app.delete("/users/{user_id}")
@require_permission(Permission.DELETE_USERS)
async def delete_user(user_id: int, current_user = Depends(get_current_user)):
    await db.delete_user(user_id)
    return {"deleted": True}

@app.get("/documents/{document_id}")
async def get_document(
    document = Depends(require_ownership_or_admin("documents"))
):
    return document
```

```json
// Authorization error response
{
  "error": {
    "code": "forbidden",
    "message": "You don't have permission to delete users",
    "details": {
      "requiredPermission": "delete:users",
      "yourRoles": ["user", "manager"],
      "requiredRoles": ["admin"]
    }
  }
}
```

## Authorization Patterns

| Pattern | Use Case | Example |
|---------|----------|---------|
| RBAC | Role-based access | Admin, Manager, User roles |
| ABAC | Attribute-based | Department, location, time-based |
| Ownership | Resource owners | User owns their documents |
| Hierarchical | Org structure | Managers see team's data |
| Feature flags | Feature access | Premium features |

## Why

1. **Security**: Prevents unauthorized access to sensitive data and operations.

2. **Principle of Least Privilege**: Users only get access they actually need.

3. **Audit Compliance**: Clear authorization rules support compliance requirements.

4. **Separation of Concerns**: Authorization logic is centralized and reusable.

5. **Defense in Depth**: Server-side checks can't be bypassed like client-side.

6. **Scalability**: RBAC/ABAC scales better than per-user permissions.

7. **Maintainability**: Centralized permission definitions are easy to update.

---


## Configure CORS Properly

Cross-Origin Resource Sharing (CORS) must be configured correctly to allow legitimate cross-origin requests while preventing unauthorized access.

## Bad Example

```javascript
// Anti-pattern: Allow all origins
app.use(cors({
  origin: '*',
  credentials: true  // DANGEROUS: Can't use * with credentials!
}));

// Anti-pattern: Reflecting origin without validation
app.use((req, res, next) => {
  // Allows ANY origin - security vulnerability
  res.header('Access-Control-Allow-Origin', req.headers.origin);
  res.header('Access-Control-Allow-Credentials', 'true');
  next();
});

// Anti-pattern: Overly permissive headers
app.use(cors({
  origin: '*',
  methods: '*',
  allowedHeaders: '*',
  exposedHeaders: '*'
}));

// Anti-pattern: Missing CORS entirely for API
app.get('/api/data', (req, res) => {
  // Browser will block cross-origin requests
  res.json({ data: 'value' });
});
```

## Good Example

```javascript
const cors = require('cors');

// Allowed origins whitelist
const allowedOrigins = [
  'https://myapp.com',
  'https://www.myapp.com',
  'https://admin.myapp.com'
];

// Development origins (only in non-production)
if (process.env.NODE_ENV !== 'production') {
  allowedOrigins.push(
    'http://localhost:3000',
    'http://localhost:8080'
  );
}

// CORS configuration
const corsOptions = {
  origin: (origin, callback) => {
    // Allow requests with no origin (mobile apps, curl, etc.)
    if (!origin) {
      return callback(null, true);
    }

    if (allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE', 'OPTIONS'],
  allowedHeaders: [
    'Content-Type',
    'Authorization',
    'X-Request-ID',
    'X-Requested-With'
  ],
  exposedHeaders: [
    'X-Request-ID',
    'X-RateLimit-Limit',
    'X-RateLimit-Remaining',
    'X-RateLimit-Reset'
  ],
  credentials: true,
  maxAge: 86400, // 24 hours - cache preflight requests
  optionsSuccessStatus: 204
};

app.use(cors(corsOptions));

// Handle CORS errors
app.use((err, req, res, next) => {
  if (err.message === 'Not allowed by CORS') {
    return res.status(403).json({
      error: {
        code: 'cors_error',
        message: 'Cross-origin request blocked',
        origin: req.headers.origin
      }
    });
  }
  next(err);
});

// Route-specific CORS (for public endpoints)
const publicCors = cors({
  origin: '*',
  methods: ['GET'],
  maxAge: 86400
});

app.get('/api/public/health', publicCors, (req, res) => {
  res.json({ status: 'ok' });
});

// Strict CORS for sensitive endpoints
const strictCors = cors({
  origin: 'https://admin.myapp.com',
  credentials: true,
  methods: ['GET', 'POST', 'DELETE']
});

app.use('/api/admin', strictCors, adminRouter);
```

```python
# FastAPI CORS configuration
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
import os

app = FastAPI()

# Define allowed origins
allowed_origins = [
    "https://myapp.com",
    "https://www.myapp.com",
    "https://admin.myapp.com"
]

# Add development origins
if os.getenv("ENVIRONMENT") != "production":
    allowed_origins.extend([
        "http://localhost:3000",
        "http://localhost:8080"
    ])

app.add_middleware(
    CORSMiddleware,
    allow_origins=allowed_origins,
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "PATCH", "DELETE"],
    allow_headers=[
        "Content-Type",
        "Authorization",
        "X-Request-ID",
        "X-Requested-With"
    ],
    expose_headers=[
        "X-Request-ID",
        "X-RateLimit-Limit",
        "X-RateLimit-Remaining"
    ],
    max_age=86400
)

# For public endpoints, create a sub-application
public_app = FastAPI()
public_app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # Public API
    allow_methods=["GET"],
    max_age=86400
)

app.mount("/public", public_app)
```

```typescript
// Dynamic CORS based on subdomain pattern
const corsOptions = {
  origin: (origin: string | undefined, callback: Function) => {
    if (!origin) {
      return callback(null, true);
    }

    // Allow all subdomains of myapp.com
    const allowedPattern = /^https:\/\/([a-z0-9-]+\.)?myapp\.com$/;

    if (allowedPattern.test(origin)) {
      callback(null, true);
    } else {
      callback(new Error('CORS not allowed'));
    }
  },
  credentials: true
};
```

## CORS Headers Reference

| Header | Purpose | Example |
|--------|---------|---------|
| Access-Control-Allow-Origin | Allowed origins | `https://myapp.com` |
| Access-Control-Allow-Methods | Allowed HTTP methods | `GET, POST, PUT` |
| Access-Control-Allow-Headers | Allowed request headers | `Content-Type, Authorization` |
| Access-Control-Expose-Headers | Headers client can read | `X-Request-ID` |
| Access-Control-Allow-Credentials | Allow cookies/auth | `true` |
| Access-Control-Max-Age | Preflight cache time | `86400` |

## Common Patterns

```javascript
// Pattern 1: Subdomain allowlist
const isAllowedOrigin = (origin) => {
  if (!origin) return true;
  const url = new URL(origin);
  return url.hostname.endsWith('.myapp.com');
};

// Pattern 2: Environment-based
const origins = {
  production: ['https://myapp.com'],
  staging: ['https://staging.myapp.com'],
  development: ['http://localhost:3000']
};
const allowedOrigins = origins[process.env.NODE_ENV];

// Pattern 3: Database-driven (for multi-tenant)
const corsOptions = {
  origin: async (origin, callback) => {
    const tenant = await db.findTenantByDomain(origin);
    callback(null, tenant?.corsEnabled ?? false);
  }
};
```

## Why

1. **Security**: Prevents malicious websites from making unauthorized API calls.

2. **Credential Protection**: Proper config prevents credential leakage to untrusted origins.

3. **Attack Prevention**: Blocks CSRF attacks that rely on cross-origin requests.

4. **Controlled Access**: Explicitly whitelist which domains can access your API.

5. **Performance**: Preflight caching reduces OPTIONS request overhead.

6. **Flexibility**: Route-specific CORS allows different policies for different endpoints.

7. **Debugging**: Clear CORS errors help developers identify misconfiguration.

---


## Enforce HTTPS Only

All API traffic must use HTTPS to encrypt data in transit. Never allow unencrypted HTTP connections for APIs.

## Bad Example

```javascript
// Anti-pattern: HTTP server without TLS
const http = require('http');
const app = require('./app');

http.createServer(app).listen(80, () => {
  console.log('API running on http://localhost:80');
  // Credentials transmitted in plain text!
});

// Anti-pattern: Optional HTTPS
if (process.env.USE_HTTPS === 'true') {
  // HTTPS is optional, not enforced
}

// Anti-pattern: No redirect from HTTP to HTTPS
app.get('/', (req, res) => {
  // Allows HTTP access
  res.json({ message: 'Welcome' });
});

// Anti-pattern: Insecure cookie settings
res.cookie('session', token, {
  secure: false,  // Sent over HTTP!
  httpOnly: true
});
```

```json
// Anti-pattern: HTTP URLs in responses
{
  "user": {
    "id": 123,
    "avatar": "http://cdn.example.com/avatar.jpg",
    "profile": "http://api.example.com/users/123"
  }
}
```

## Good Example

```javascript
const https = require('https');
const fs = require('fs');
const express = require('express');
const helmet = require('helmet');

const app = express();

// Security headers including HSTS
app.use(helmet({
  hsts: {
    maxAge: 31536000, // 1 year
    includeSubDomains: true,
    preload: true
  }
}));

// Force HTTPS redirect (for direct access)
app.use((req, res, next) => {
  if (!req.secure && req.get('x-forwarded-proto') !== 'https') {
    return res.redirect(301, `https://${req.get('host')}${req.url}`);
  }
  next();
});

// Secure cookie settings
app.use((req, res, next) => {
  res.cookie = function(name, value, options = {}) {
    options.secure = true;      // HTTPS only
    options.httpOnly = true;    // No JavaScript access
    options.sameSite = 'strict'; // CSRF protection
    return res.cookie.call(this, name, value, options);
  };
  next();
});

// HTTPS server
const options = {
  key: fs.readFileSync('/path/to/private.key'),
  cert: fs.readFileSync('/path/to/certificate.crt'),
  ca: fs.readFileSync('/path/to/ca-bundle.crt'),
  minVersion: 'TLSv1.2',  // Minimum TLS version
  ciphers: [
    'ECDHE-ECDSA-AES128-GCM-SHA256',
    'ECDHE-RSA-AES128-GCM-SHA256',
    'ECDHE-ECDSA-AES256-GCM-SHA384',
    'ECDHE-RSA-AES256-GCM-SHA384'
  ].join(':')
};

https.createServer(options, app).listen(443, () => {
  console.log('Secure API running on https://localhost:443');
});

// Also listen on HTTP just to redirect
const http = require('http');
http.createServer((req, res) => {
  res.writeHead(301, { Location: `https://${req.headers.host}${req.url}` });
  res.end();
}).listen(80);
```

```python
# FastAPI with HTTPS enforcement
from fastapi import FastAPI, Request
from fastapi.middleware.httpsredirect import HTTPSRedirectMiddleware
from fastapi.middleware.trustedhost import TrustedHostMiddleware
from starlette.middleware.base import BaseHTTPMiddleware

app = FastAPI()

# Redirect HTTP to HTTPS
app.add_middleware(HTTPSRedirectMiddleware)

# Only allow specific hosts
app.add_middleware(
    TrustedHostMiddleware,
    allowed_hosts=["api.example.com", "*.example.com"]
)

# Add HSTS header
class HSTSMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        response = await call_next(request)
        response.headers["Strict-Transport-Security"] = (
            "max-age=31536000; includeSubDomains; preload"
        )
        return response

app.add_middleware(HSTSMiddleware)

# Secure cookie response
from fastapi.responses import JSONResponse

@app.post("/auth/login")
async def login(credentials: Credentials):
    token = create_token(credentials)
    response = JSONResponse({"status": "logged_in"})
    response.set_cookie(
        key="session",
        value=token,
        httponly=True,
        secure=True,
        samesite="strict",
        max_age=86400
    )
    return response
```

```nginx
# Nginx HTTPS configuration
server {
    listen 80;
    server_name api.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name api.example.com;

    ssl_certificate /etc/ssl/certs/certificate.crt;
    ssl_certificate_key /etc/ssl/private/private.key;

    # Modern TLS configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
    ssl_prefer_server_ciphers on;

    # HSTS
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;

    # Security headers
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "DENY" always;
    add_header X-XSS-Protection "1; mode=block" always;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header X-Forwarded-Proto https;
    }
}
```

```yaml
# Docker Compose with Traefik for automatic HTTPS
version: '3.8'
services:
  traefik:
    image: traefik:v2.10
    command:
      - "--providers.docker=true"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--entrypoints.web.http.redirections.entryPoint.to=websecure"
      - "--certificatesresolvers.letsencrypt.acme.httpchallenge.entrypoint=web"
      - "--certificatesresolvers.letsencrypt.acme.email=admin@example.com"
      - "--certificatesresolvers.letsencrypt.acme.storage=/letsencrypt/acme.json"
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - letsencrypt:/letsencrypt

  api:
    image: my-api
    labels:
      - "traefik.http.routers.api.rule=Host(`api.example.com`)"
      - "traefik.http.routers.api.tls.certresolver=letsencrypt"
      - "traefik.http.middlewares.hsts.headers.stsSeconds=31536000"
      - "traefik.http.routers.api.middlewares=hsts"
```

## TLS Configuration Checklist

| Setting | Recommendation |
|---------|----------------|
| Minimum TLS | TLSv1.2 |
| Preferred TLS | TLSv1.3 |
| HSTS max-age | 31536000 (1 year) |
| includeSubDomains | Yes |
| HSTS preload | Yes (after testing) |
| Certificate | Valid, not self-signed |
| Certificate chain | Complete |

## Why

1. **Encryption**: HTTPS encrypts all data in transit, protecting credentials and sensitive data.

2. **Authentication**: TLS certificates verify server identity, preventing MITM attacks.

3. **Integrity**: HTTPS ensures data isn't modified in transit.

4. **Compliance**: PCI-DSS, HIPAA, and GDPR require encryption of data in transit.

5. **SEO/Trust**: Browsers mark HTTP as "Not Secure," damaging user trust.

6. **Modern Features**: HTTP/2 and many web APIs require HTTPS.

7. **Cookie Security**: Secure cookies only work over HTTPS.

---


## Validate All Input Data

Never trust client input. Validate, sanitize, and constrain all incoming data to prevent security vulnerabilities.

## Bad Example

```javascript
// Anti-pattern: No validation
app.post('/users', async (req, res) => {
  // Directly using user input!
  const user = await db.createUser(req.body);
  res.json(user);
});

// Anti-pattern: SQL injection vulnerability
app.get('/users', async (req, res) => {
  const query = `SELECT * FROM users WHERE name = '${req.query.name}'`;
  const users = await db.raw(query);
  res.json(users);
});

// Anti-pattern: NoSQL injection
app.post('/login', async (req, res) => {
  const user = await db.users.findOne({
    email: req.body.email,
    password: req.body.password  // Could be { $gt: '' }
  });
});

// Anti-pattern: Path traversal
app.get('/files/:filename', (req, res) => {
  const path = `./uploads/${req.params.filename}`;
  res.sendFile(path); // Could access ../../../etc/passwd
});
```

## Good Example

```javascript
const { body, param, query, validationResult } = require('express-validator');
const sanitizeHtml = require('sanitize-html');
const path = require('path');

// Validation middleware
const validateUser = [
  body('email')
    .trim()
    .notEmpty().withMessage('Email is required')
    .isEmail().withMessage('Must be a valid email')
    .normalizeEmail()
    .isLength({ max: 255 }).withMessage('Email too long'),

  body('password')
    .notEmpty().withMessage('Password is required')
    .isLength({ min: 8, max: 100 }).withMessage('Password must be 8-100 characters')
    .matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/).withMessage('Password too weak'),

  body('name')
    .trim()
    .notEmpty().withMessage('Name is required')
    .isLength({ min: 1, max: 100 }).withMessage('Name must be 1-100 characters')
    .matches(/^[a-zA-Z\s'-]+$/).withMessage('Name contains invalid characters')
    .customSanitizer(value => sanitizeHtml(value, { allowedTags: [] })),

  body('age')
    .optional()
    .isInt({ min: 0, max: 150 }).withMessage('Age must be 0-150')
    .toInt(),

  body('website')
    .optional()
    .trim()
    .isURL({ protocols: ['http', 'https'] }).withMessage('Invalid URL')
];

// Validation error handler
const handleValidation = (req, res, next) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({
      error: {
        code: 'validation_error',
        message: 'Invalid input data',
        details: errors.array().map(e => ({
          field: e.path,
          message: e.msg,
          value: e.value
        }))
      }
    });
  }
  next();
};

app.post('/users', validateUser, handleValidation, async (req, res) => {
  const user = await db.createUser(req.body);
  res.status(201).json(user);
});

// Parameterized queries (prevent SQL injection)
app.get('/users', async (req, res) => {
  const users = await db.query(
    'SELECT id, name, email FROM users WHERE name = ?',
    [req.query.name]
  );
  res.json(users);
});

// Safe MongoDB queries
app.post('/login', async (req, res) => {
  // Ensure email and password are strings
  const email = String(req.body.email || '');
  const password = String(req.body.password || '');

  const user = await db.users.findOne({ email });
  if (!user || !await bcrypt.compare(password, user.passwordHash)) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  // ...
});

// Path traversal prevention
app.get('/files/:filename',
  param('filename')
    .matches(/^[a-zA-Z0-9_-]+\.[a-zA-Z0-9]+$/)
    .withMessage('Invalid filename'),
  handleValidation,
  (req, res) => {
    const uploadsDir = path.resolve('./uploads');
    const filePath = path.join(uploadsDir, req.params.filename);

    // Ensure path is within uploads directory
    if (!filePath.startsWith(uploadsDir)) {
      return res.status(400).json({ error: 'Invalid path' });
    }

    res.sendFile(filePath);
  }
);

// Array size limits
app.post('/batch',
  body('items')
    .isArray({ min: 1, max: 100 })
    .withMessage('Items must be array of 1-100 elements'),
  body('items.*.id')
    .isUUID()
    .withMessage('Each item must have valid UUID'),
  handleValidation,
  async (req, res) => {
    const results = await processBatch(req.body.items);
    res.json(results);
  }
);

// JSON depth/size limits
const jsonParser = express.json({
  limit: '100kb', // Max request body size
  strict: true    // Only accept arrays and objects
});

app.use('/api', jsonParser);
```

```python
# FastAPI with Pydantic validation
from fastapi import FastAPI, Query, Path, Body, HTTPException
from pydantic import BaseModel, EmailStr, Field, validator, HttpUrl
from typing import Optional, List
import re
import bleach

app = FastAPI()

class UserCreate(BaseModel):
    email: EmailStr
    password: str = Field(..., min_length=8, max_length=100)
    name: str = Field(..., min_length=1, max_length=100)
    age: Optional[int] = Field(None, ge=0, le=150)
    website: Optional[HttpUrl] = None

    @validator('password')
    def password_strength(cls, v):
        if not re.search(r'[A-Z]', v):
            raise ValueError('Must contain uppercase letter')
        if not re.search(r'[a-z]', v):
            raise ValueError('Must contain lowercase letter')
        if not re.search(r'\d', v):
            raise ValueError('Must contain digit')
        return v

    @validator('name')
    def sanitize_name(cls, v):
        # Only allow letters, spaces, hyphens, apostrophes
        if not re.match(r"^[a-zA-Z\s'-]+$", v):
            raise ValueError('Name contains invalid characters')
        # Sanitize HTML
        return bleach.clean(v, tags=[], strip=True)

class BatchRequest(BaseModel):
    items: List[str] = Field(..., min_items=1, max_items=100)

    @validator('items', each_item=True)
    def validate_uuid(cls, v):
        if not re.match(r'^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$', v, re.I):
            raise ValueError('Invalid UUID format')
        return v

@app.post("/users")
async def create_user(user: UserCreate):
    return await db.create_user(user.dict())

@app.get("/users/{user_id}")
async def get_user(
    user_id: int = Path(..., ge=1, le=2147483647, description="User ID")
):
    return await db.get_user(user_id)

@app.get("/search")
async def search(
    q: str = Query(..., min_length=1, max_length=100),
    page: int = Query(1, ge=1, le=1000),
    limit: int = Query(20, ge=1, le=100)
):
    # q is automatically validated and constrained
    return await db.search(q, page, limit)
```

## Input Validation Checklist

| Check | Why |
|-------|-----|
| Type validation | Prevent type confusion attacks |
| Length limits | Prevent buffer overflows, DoS |
| Character whitelist | Prevent injection attacks |
| Range validation | Ensure business logic integrity |
| Format validation | Email, URL, UUID patterns |
| Sanitization | Remove/escape dangerous content |
| Array size limits | Prevent memory exhaustion |
| Nested depth limits | Prevent stack overflow |

## Why

1. **SQL/NoSQL Injection**: Validation prevents malicious query manipulation.

2. **XSS Prevention**: Sanitizing input stops script injection attacks.

3. **Path Traversal**: Validating filenames prevents unauthorized file access.

4. **DoS Prevention**: Size limits prevent memory exhaustion attacks.

5. **Data Integrity**: Ensures only valid data enters your system.

6. **Business Logic**: Enforces domain rules at the API boundary.

7. **Error Prevention**: Catches bad data before it causes downstream errors.

---


## Implement Rate Limiting

Protect your API from abuse by limiting the number of requests clients can make within a time window.

## Bad Example

```javascript
// Anti-pattern: No rate limiting
app.post('/login', async (req, res) => {
  // Vulnerable to brute force attacks
  const user = await authenticate(req.body);
  res.json(user);
});

// Anti-pattern: Rate limit only on response
app.get('/api/data', async (req, res) => {
  const data = await expensiveQuery(); // Query runs every time!
  if (requestCount > 100) {
    res.status(429).json({ error: 'Too many requests' });
  }
  res.json(data);
});

// Anti-pattern: No rate limit headers
app.use((req, res, next) => {
  if (isRateLimited(req)) {
    res.status(429).send('Too many requests');
    // Client doesn't know when to retry
  }
  next();
});
```

## Good Example

```javascript
const rateLimit = require('express-rate-limit');
const RedisStore = require('rate-limit-redis');
const Redis = require('ioredis');

const redis = new Redis(process.env.REDIS_URL);

// General API rate limiter
const apiLimiter = rateLimit({
  store: new RedisStore({
    client: redis,
    prefix: 'rl:api:'
  }),
  windowMs: 60 * 1000, // 1 minute
  max: 100, // 100 requests per minute
  standardHeaders: true, // Return rate limit info in headers
  legacyHeaders: false,
  message: {
    error: {
      code: 'rate_limit_exceeded',
      message: 'Too many requests. Please slow down.',
      retryAfter: 60
    }
  },
  keyGenerator: (req) => {
    // Rate limit by user ID if authenticated, otherwise by IP
    return req.user?.id || req.ip;
  }
});

// Stricter limiter for auth endpoints
const authLimiter = rateLimit({
  store: new RedisStore({
    client: redis,
    prefix: 'rl:auth:'
  }),
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // 5 attempts per 15 minutes
  standardHeaders: true,
  message: {
    error: {
      code: 'rate_limit_exceeded',
      message: 'Too many login attempts. Please try again later.',
      retryAfter: 900
    }
  },
  keyGenerator: (req) => req.body.email || req.ip // Per email address
});

// Expensive endpoint limiter
const expensiveLimiter = rateLimit({
  store: new RedisStore({
    client: redis,
    prefix: 'rl:expensive:'
  }),
  windowMs: 60 * 60 * 1000, // 1 hour
  max: 10, // 10 requests per hour
  standardHeaders: true,
  message: {
    error: {
      code: 'rate_limit_exceeded',
      message: 'Export limit reached. Try again in an hour.',
      retryAfter: 3600
    }
  }
});

// Apply rate limiters
app.use('/api/', apiLimiter);
app.use('/auth/login', authLimiter);
app.use('/auth/password-reset', authLimiter);
app.post('/api/export', expensiveLimiter, exportHandler);

// Custom sliding window implementation
class SlidingWindowRateLimiter {
  constructor(redis, options) {
    this.redis = redis;
    this.windowMs = options.windowMs;
    this.maxRequests = options.max;
  }

  async isAllowed(key) {
    const now = Date.now();
    const windowStart = now - this.windowMs;

    const pipeline = this.redis.pipeline();

    // Remove old entries
    pipeline.zremrangebyscore(key, 0, windowStart);

    // Count requests in window
    pipeline.zcard(key);

    // Add current request
    pipeline.zadd(key, now, `${now}-${Math.random()}`);

    // Set expiry
    pipeline.expire(key, Math.ceil(this.windowMs / 1000));

    const results = await pipeline.exec();
    const count = results[1][1];

    return {
      allowed: count < this.maxRequests,
      remaining: Math.max(0, this.maxRequests - count - 1),
      resetAt: new Date(now + this.windowMs)
    };
  }
}

// Tiered rate limiting based on plan
const tierLimits = {
  free: { requestsPerMinute: 60, requestsPerDay: 1000 },
  basic: { requestsPerMinute: 300, requestsPerDay: 10000 },
  pro: { requestsPerMinute: 1000, requestsPerDay: 100000 },
  enterprise: { requestsPerMinute: 5000, requestsPerDay: 1000000 }
};

async function tieredRateLimiter(req, res, next) {
  const user = req.user;
  const tier = user?.plan || 'free';
  const limits = tierLimits[tier];

  const minuteKey = `rl:${user?.id || req.ip}:minute`;
  const dayKey = `rl:${user?.id || req.ip}:day`;

  // Check minute limit
  const minuteCount = await redis.incr(minuteKey);
  if (minuteCount === 1) {
    await redis.expire(minuteKey, 60);
  }

  // Check daily limit
  const dayCount = await redis.incr(dayKey);
  if (dayCount === 1) {
    await redis.expire(dayKey, 86400);
  }

  // Set rate limit headers
  res.set('X-RateLimit-Limit', limits.requestsPerMinute);
  res.set('X-RateLimit-Remaining', Math.max(0, limits.requestsPerMinute - minuteCount));
  res.set('X-RateLimit-Reset', Math.ceil(Date.now() / 1000) + 60);

  if (minuteCount > limits.requestsPerMinute) {
    return res.status(429).json({
      error: {
        code: 'rate_limit_exceeded',
        message: 'Minute rate limit exceeded',
        limit: limits.requestsPerMinute,
        window: '1 minute',
        retryAfter: await redis.ttl(minuteKey)
      }
    });
  }

  if (dayCount > limits.requestsPerDay) {
    return res.status(429).json({
      error: {
        code: 'rate_limit_exceeded',
        message: 'Daily rate limit exceeded',
        limit: limits.requestsPerDay,
        window: '24 hours',
        upgradeUrl: '/pricing'
      }
    });
  }

  next();
}
```

```python
# FastAPI with rate limiting
from fastapi import FastAPI, Request, HTTPException
from slowapi import Limiter, _rate_limit_exceeded_handler
from slowapi.util import get_remote_address
from slowapi.errors import RateLimitExceeded

limiter = Limiter(key_func=get_remote_address)
app = FastAPI()
app.state.limiter = limiter
app.add_exception_handler(RateLimitExceeded, _rate_limit_exceeded_handler)

@app.get("/api/data")
@limiter.limit("100/minute")
async def get_data(request: Request):
    return {"data": "value"}

@app.post("/auth/login")
@limiter.limit("5/15minutes")
async def login(request: Request):
    return {"token": "..."}

# Custom user-based rate limiting
async def get_rate_limit_key(request: Request):
    if request.user:
        return f"user:{request.user.id}"
    return f"ip:{request.client.host}"

@app.get("/api/premium")
@limiter.limit("1000/minute", key_func=get_rate_limit_key)
async def premium_endpoint(request: Request):
    return {"data": "premium"}
```

## Rate Limit Headers

```
HTTP/1.1 200 OK
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1705312800
Retry-After: 60
```

## Why

1. **DDoS Protection**: Prevents denial-of-service attacks from overwhelming your servers.

2. **Brute Force Prevention**: Limits password guessing and credential stuffing attacks.

3. **Fair Usage**: Ensures all users get fair access to API resources.

4. **Cost Control**: Prevents runaway API usage that could increase infrastructure costs.

5. **Service Stability**: Protects backend services from traffic spikes.

6. **Business Model**: Enables tiered pricing based on usage limits.

7. **Client Guidance**: Headers help clients implement proper backoff strategies.

---


## Protect Sensitive Data in Responses

Never expose sensitive information like passwords, tokens, internal IDs, or PII in API responses.

## Bad Example

```json
// Anti-pattern: Exposing password hash
{
  "id": 123,
  "email": "user@example.com",
  "passwordHash": "$2b$12$LQv3c1yqBWVHxkd0LHAkCOYz6TtxMQJqhN8/X4.n",
  "name": "John Doe"
}

// Anti-pattern: Exposing API keys
{
  "user": {
    "id": 123,
    "apiKey": "sk_live_abc123xyz789",
    "secretKey": "whsec_secret_key_here"
  }
}

// Anti-pattern: Exposing internal IDs
{
  "user": {
    "id": 123,
    "internalDatabaseId": "mongo_507f1f77bcf86cd799439011",
    "stripeCustomerId": "cus_abc123"
  }
}

// Anti-pattern: Exposing other users' data
{
  "order": {
    "id": 456,
    "customer": {
      "ssn": "123-45-6789",
      "creditCard": "4111111111111111",
      "dateOfBirth": "1990-01-15"
    }
  }
}
```

```javascript
// Dangerous: Returning entire database record
app.get('/users/:id', async (req, res) => {
  const user = await db.findUser(req.params.id);
  res.json(user); // Exposes ALL fields including sensitive ones
});
```

## Good Example

```javascript
// Define what fields to expose per context
const userPublicFields = ['id', 'name', 'avatar', 'createdAt'];
const userPrivateFields = [...userPublicFields, 'email', 'settings'];
const userAdminFields = [...userPrivateFields, 'roles', 'lastLoginAt', 'status'];

function sanitizeUser(user, context = 'public') {
  const allowedFields = {
    public: userPublicFields,
    private: userPrivateFields,
    admin: userAdminFields
  }[context];

  return Object.fromEntries(
    Object.entries(user).filter(([key]) => allowedFields.includes(key))
  );
}

// Use explicit field selection
app.get('/users/:id', authenticate, async (req, res) => {
  const user = await db.findUser(req.params.id);

  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }

  // Determine context based on requester
  let context = 'public';
  if (req.user.id === user.id) {
    context = 'private';
  } else if (req.user.roles.includes('admin')) {
    context = 'admin';
  }

  res.json(sanitizeUser(user, context));
});

// Use DTOs/serializers
class UserResponse {
  constructor(user, includePrivate = false) {
    this.id = user.id;
    this.name = user.name;
    this.avatar = user.avatar;
    this.createdAt = user.createdAt;

    if (includePrivate) {
      this.email = user.email;
      this.settings = user.settings;
    }
  }
}

app.get('/users/:id', async (req, res) => {
  const user = await db.findUser(req.params.id);
  const includePrivate = req.user?.id === user.id;
  res.json(new UserResponse(user, includePrivate));
});

// Mask sensitive data
function maskEmail(email) {
  const [local, domain] = email.split('@');
  const maskedLocal = local[0] + '*'.repeat(local.length - 2) + local.slice(-1);
  return `${maskedLocal}@${domain}`;
}

function maskPhone(phone) {
  return phone.replace(/(\d{3})\d{4}(\d{4})/, '$1****$2');
}

function maskCreditCard(number) {
  return '**** **** **** ' + number.slice(-4);
}

// Apply masking in responses
app.get('/users/:id', async (req, res) => {
  const user = await db.findUser(req.params.id);

  res.json({
    id: user.id,
    name: user.name,
    email: req.user.id === user.id ? user.email : maskEmail(user.email),
    phone: maskPhone(user.phone),
    paymentMethod: {
      type: 'card',
      last4: user.creditCard.slice(-4),
      brand: user.cardBrand
    }
  });
});
```

```python
# FastAPI with Pydantic response models
from pydantic import BaseModel, EmailStr, Field
from typing import Optional

# Internal model (full data)
class UserInternal(BaseModel):
    id: int
    email: str
    password_hash: str
    name: str
    ssn: Optional[str]
    api_key: str
    stripe_customer_id: str

# Public response model (safe to expose)
class UserPublic(BaseModel):
    id: int
    name: str
    avatar_url: Optional[str]

    class Config:
        # Only include fields defined in this model
        extra = 'forbid'

# Private response model (for account owner)
class UserPrivate(UserPublic):
    email: EmailStr
    settings: dict

# Admin response model
class UserAdmin(UserPrivate):
    status: str
    roles: list
    last_login_at: Optional[str]

@app.get("/users/{user_id}", response_model=UserPublic)
async def get_user(user_id: int, current_user = Depends(get_current_user)):
    user = await db.get_user(user_id)

    if current_user.id == user_id:
        return UserPrivate(**user.dict())
    elif "admin" in current_user.roles:
        return UserAdmin(**user.dict())
    else:
        return UserPublic(**user.dict())

# Mask sensitive data in logs
import logging

class SensitiveDataFilter(logging.Filter):
    PATTERNS = [
        (r'"password":\s*"[^"]*"', '"password": "[REDACTED]"'),
        (r'"token":\s*"[^"]*"', '"token": "[REDACTED]"'),
        (r'"apiKey":\s*"[^"]*"', '"apiKey": "[REDACTED]"'),
        (r'\b\d{3}-\d{2}-\d{4}\b', '[SSN REDACTED]'),
        (r'\b\d{16}\b', '[CARD REDACTED]'),
    ]

    def filter(self, record):
        import re
        message = record.getMessage()
        for pattern, replacement in self.PATTERNS:
            message = re.sub(pattern, replacement, message)
        record.msg = message
        return True
```

```json
// Safe user response
{
  "id": 123,
  "name": "John Doe",
  "avatar": "https://cdn.example.com/avatars/123.jpg",
  "email": "j***n@example.com",
  "memberSince": "2023-01-15"
}

// Safe payment method response
{
  "paymentMethods": [
    {
      "id": "pm_abc123",
      "type": "card",
      "brand": "visa",
      "last4": "4242",
      "expiryMonth": 12,
      "expiryYear": 2025
    }
  ]
}
```

## Sensitive Data Checklist

| Field | Action | Reason |
|-------|--------|--------|
| Password hash | Never expose | Security |
| API keys | Never expose | Security |
| SSN/Tax ID | Never/mask | PII/Compliance |
| Full credit card | Never expose | PCI-DSS |
| Internal IDs | Usually hide | Information disclosure |
| Email (other users) | Mask | Privacy |
| Phone | Mask | Privacy |
| Address | Context-dependent | Privacy |

## Why

1. **Security**: Exposed credentials enable account takeover.

2. **Privacy**: PII exposure violates privacy regulations (GDPR, CCPA).

3. **Compliance**: PCI-DSS requires protecting cardholder data.

4. **Trust**: Users expect their data to be protected.

5. **Attack Surface**: Internal IDs help attackers enumerate resources.

6. **Least Privilege**: Only expose data the client actually needs.

7. **Audit**: Proper data handling simplifies compliance audits.

---

