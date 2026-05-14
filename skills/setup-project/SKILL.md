---
name: setup-project
description: Create a new MoonDB project and get API keys. Use when starting a new app that needs a backend.
---

# Set Up a MoonDB Project

## When to use

- Starting a new application that needs a backend
- The user says "add a backend", "set up a database", or "create a MoonDB project"

## Prerequisites

The user needs a MoonDB account. If they don't have one:

```
POST https://api.moondb.ai/v1/accounts/signup
Content-Type: application/json

{ "email": "user@example.com", "password": "..." }
```

This returns an `api_key` (starts with `mk_`) used for all management operations.

## Steps

### 1. Create a project

Use the MCP `create_project` tool, or call the API directly:

```
POST https://api.moondb.ai/v1/projects
X-API-Key: mk_...
Content-Type: application/json

{ "name": "my-app" }
```

Response:

```json
{
  "data": {
    "id": "proj_...",
    "name": "my-app",
    "admin_key": "sk_...",
    "public_key": "pk_..."
  }
}
```

Save these keys:
- `admin_key` (sk_...) — server-side only, for schema management and admin operations
- `public_key` (pk_...) — safe for client-side, for public-read endpoints

### 2. Store keys

Add to the project's `.env`:

```
MOONDB_PROJECT_ID=proj_...
MOONDB_ADMIN_KEY=sk_...
MOONDB_PUBLIC_KEY=pk_...
MOONDB_API_URL=https://api.moondb.ai/p/proj_...
```

### 3. Define a schema

Proceed to the `define-schema` skill to set up tables.

### 4. Fetch LLM context (optional)

For full API documentation tailored to the project's schema:

```
GET https://api.moondb.ai/p/{project_id}/v1/llm-context
X-Public-Key: pk_...
```

This returns a plain-text reference of all tables, columns, endpoints, and auth flows.
