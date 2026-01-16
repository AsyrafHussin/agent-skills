---
title: Use Nouns, Not Verbs for Resource Names
impact: CRITICAL
impactDescription: Foundation of REST architecture
tags: rest, resources, naming, http-methods
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
