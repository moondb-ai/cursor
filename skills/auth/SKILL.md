---
name: auth
description: Set up and use MoonDB built-in authentication — signup, login, tokens, email verification, external auth. Use when implementing user auth flows.
---

# Authentication

## When to use

- Adding user signup/login to an app
- Implementing protected routes or owner-based access
- Setting up email verification
- Integrating external auth (Clerk, Auth0)

## Prerequisites

Your schema must have a table with `auth_table: true`:

```json
{
  "tables": {
    "users": {
      "columns": {
        "display_name": "string"
      },
      "auth_table": true,
      "verify_email": true
    }
  }
}
```

This auto-creates `email` (string, required, unique) and `password_hash` (hidden from API), and enables all `/auth/*` endpoints.

## Auth endpoints

### Signup

```
POST /p/{project_id}/auth/signup
Content-Type: application/json

{ "email": "user@example.com", "password": "securepassword" }
```

Response:
```json
{
  "data": {
    "token": "eyJ...",
    "refresh_token": "rt_...",
    "user": { "id": "...", "email": "user@example.com", "display_name": null }
  }
}
```

If `verify_email: true`, a verification email is sent automatically and the response includes a `_warning`.

### Login

```
POST /p/{project_id}/auth/login
Content-Type: application/json

{ "email": "user@example.com", "password": "securepassword" }
```

Same response shape as signup.

### Get current user

```
GET /p/{project_id}/auth/me
Authorization: Bearer {token}
```

### Update current user

```
PATCH /p/{project_id}/auth/me
Authorization: Bearer {token}
Content-Type: application/json

{ "display_name": "New Name" }
```

### Refresh token

```
POST /p/{project_id}/auth/refresh
Content-Type: application/json

{ "refresh_token": "rt_..." }
```

Token expires in 1 hour. Refresh token lasts 30 days.

### Logout

```
POST /p/{project_id}/auth/logout
Authorization: Bearer {token}
```

### Email verification

When `verify_email: true` on the auth table:

```
GET  /p/{project_id}/auth/verify?token=...          (link from email)
POST /p/{project_id}/auth/resend-verification        (requires Bearer token)
```

User object includes `email_verified` (0 or 1).

## Using auth in API requests

- `Authorization: Bearer {token}` for authenticated/owner endpoints
- `X-Public-Key: pk_...` for public-read endpoints from client code
- `X-Admin-Key: sk_...` for schema and admin operations (server-side only, never expose to client)

## External auth (Clerk, Auth0)

Instead of built-in auth, validate JWTs from an external provider:

```
PUT /p/{project_id}/v1/auth-config
X-Admin-Key: sk_...
Content-Type: application/json

{
  "provider": "external",
  "jwks_url": "https://your-provider.com/.well-known/jwks.json",
  "user_id_claim": "sub"
}
```

## Key rotation

```
POST /p/{project_id}/v1/rotate-keys
X-Admin-Key: sk_...
```

Returns new admin_key and public_key. Previous keys are immediately invalidated.
