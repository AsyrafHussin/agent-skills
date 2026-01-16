---
title: Provide Meaningful Error Messages
impact: HIGH
impactDescription: Reduces support burden and improves developer experience
tags: errors, messages, user-experience, actionable
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
