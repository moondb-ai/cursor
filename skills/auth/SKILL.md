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

### Change password

```
POST /p/{project_id}/auth/change-password
Authorization: Bearer {token}
Content-Type: application/json

{ "current_password": "oldpass", "new_password": "newpass123" }
```

Revokes all existing refresh tokens and returns `{ data: { ok: true } }`.

### Forgot password

```
POST /p/{project_id}/auth/forgot-password
Content-Type: application/json

{ "email": "user@example.com", "redirect_url": "https://myapp.com/reset-password" }
```

Sends a password reset email with a link to `redirect_url?token=...`. Always returns 200 (never reveals whether the email exists).

**Important:** `redirect_url` must use an origin that's in the project's `allowed_redirect_origins` allow-list — otherwise the endpoint returns 400 `VALIDATION_REDIRECT_URL`. Configure once per project (admin key):

```
GET  /p/{project_id}/v1/redirect-origins
PUT  /p/{project_id}/v1/redirect-origins
X-Admin-Key: sk_...
Content-Type: application/json

{ "allowed_redirect_origins": ["https://myapp.com", "https://staging.myapp.com"] }
```

The `redirect_url` is your app's password reset page. Build a form there that collects the new password and calls the reset endpoint.

### Reset password

```
POST /p/{project_id}/auth/reset-password
Content-Type: application/json

{ "token": "...", "new_password": "newpass123" }
```

Token expires in 1 hour. On success, all refresh tokens are revoked AND every outstanding access token for this user is invalidated (the token carries a per-user password version that bumps on every change/reset).

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

## External auth (Clerk, Auth0, Supabase, Firebase, Cognito, Kinde, WorkOS, Stytch)

Instead of built-in auth, validate RS256-signed JWTs from an external provider:

```
PUT /p/{project_id}/v1/auth-config
X-Admin-Key: sk_...
Content-Type: application/json

{
  "provider": "external",
  "jwks_url": "https://your-provider.com/.well-known/jwks.json",
  "user_id_claim": "sub",
  "audience": "your-audience"
}
```

`jwks_url` must be `https://` and use a known IdP host suffix (the API rejects unknown hosts to prevent SSRF). `audience` defaults to the project_id if you don't set it — this blocks cross-project JWT replay when two MoonDB projects share the same IdP tenant. To remove external auth: `DELETE /p/{id}/v1/auth-config`.

## Key rotation

```
POST /p/{project_id}/v1/rotate-keys          # rotate sk_ / pk_
X-Admin-Key: sk_...
{ "admin": true }   # or { "public": true } or { "both": true }
```

Returns new admin_key and/or public_key. Previous keys are immediately invalidated.

```
POST /p/{project_id}/v1/rotate-jwt-secret    # invalidate ALL outstanding end-user JWTs
X-Admin-Key: sk_...
```

Use after a suspected compromise or after a critical schema/access-rule change. The secret itself is never returned in the response — only `{ rotated: true }`. Existing refresh_tokens are unaffected; users continue with refresh or re-login.
