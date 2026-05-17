---
name: setup-project
description: Create a new MoonDB project and get API keys. Use when starting a new app that needs a backend.
---

# Set Up a MoonDB Project

## When to use

- Starting a new application that needs a backend
- The user says "add a backend", "set up a database", or "create a MoonDB project"

## Prerequisites

The user needs a MoonDB API key (`mk_...`) for all management operations. Ask them for it.

How to get the key:
- **New user**: `POST https://api.moondb.ai/v1/accounts/signup { "email": "...", "password": "..." }` — the response includes `api_key`
- **Existing user**: go to https://api.moondb.ai/dashboard → Account → click "Regenerate key"

The `mk_` key is shown only once (at signup or regeneration). The user must save it.

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

When running via MCP, you can call `get_reference` (with or without `project_id`)
to fetch the same content — useful for looking up a specific feature beyond
what tool descriptions cover.

## Related management tools

When the user already has projects on MoonDB but you've lost track of which
one (or which API URL to use), prefer these over re-creating:

- **`list_projects`** — enumerate every project owned by the account, with
  `api_url` and `public_key` per project. Call before `create_project` to
  avoid duplicates.
- **`rotate_keys`** — recovery path when the user has lost the `admin_key`.
  Returns a fresh `sk_...` (shown once). Old key is invalidated immediately.
  Use either `{ admin: true }`, `{ public: true }`, or `{ both: true }`.
