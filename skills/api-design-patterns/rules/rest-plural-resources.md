---
title: Use Plural Nouns for Resource Collections
impact: HIGH
impactDescription: Improves API consistency and predictability
tags: rest, resources, naming, conventions
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
