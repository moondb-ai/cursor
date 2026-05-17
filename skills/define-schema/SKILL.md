---
name: define-schema
description: Define or update a MoonDB schema with tables, columns, auth, and access rules. Use when creating or modifying the data model.
---

# Define a MoonDB Schema

## When to use

- Creating the initial data model for an app
- Adding, modifying, or removing tables or columns
- Setting up authentication or access control

## Schema format

Send the full schema as JSON. MoonDB auto-diffs against the current state and generates migrations.

```
PUT https://api.moondb.ai/p/{project_id}/v1/schema
X-Admin-Key: sk_...
Content-Type: application/json
```

### Column types

| Type       | Description                    | Notes                                     |
|------------|--------------------------------|-------------------------------------------|
| string     | Short text (max 255)           |                                           |
| text       | Long text (unlimited)          |                                           |
| int        | Integer                        |                                           |
| float      | Floating point                 |                                           |
| bool       | Boolean                        |                                           |
| date       | Date (YYYY-MM-DD)              |                                           |
| datetime   | Date + time (ISO 8601)         |                                           |
| timestamp  | Unix timestamp                 |                                           |
| json       | JSON object/array              |                                           |
| enum       | Fixed set of values            | Requires `values` array                   |
| ref        | Foreign key (UUID)             | Requires `table` pointing to another table|
| uuid       | UUID v4                        | Auto-generated if not provided            |
| file       | File reference                 | Use with /storage/upload                  |

### Short-form syntax

Instead of verbose objects, use space-separated strings:

```json
{
  "email": "string required unique",
  "age": "int default 0",
  "user_id": "ref users required",
  "published": "bool default false",
  "role": { "type": "enum", "values": ["user", "admin"], "default": "user" }
}
```

### Full example

```json
{
  "tables": {
    "users": {
      "columns": {
        "display_name": "string",
        "avatar": "file",
        "role": { "type": "enum", "values": ["user", "admin"], "default": "user" }
      },
      "auth_table": true,
      "verify_email": true
    },
    "posts": {
      "columns": {
        "user_id": "ref users required",
        "title": "string required",
        "body": "text",
        "published": "bool default false",
        "cover_image": "file"
      },
      "access": {
        "read": "public",
        "create": "authenticated",
        "update": "owner",
        "delete": "owner"
      },
      "owner_field": "user_id"
    },
    "comments": {
      "columns": {
        "post_id": { "type": "ref", "table": "posts", "required": true, "on_delete": "cascade" },
        "user_id": "ref users required",
        "body": "text required"
      },
      "access": {
        "read": "public",
        "create": "authenticated",
        "update": "owner",
        "delete": "owner"
      },
      "owner_field": "user_id"
    }
  },
  "ai_endpoints": {
    "summarize": {
      "model": "gemma",
      "prompt": "Summarize this text in 2 sentences: {{text}}",
      "access": "auth"
    }
  }
}
```

### Built-in columns (never define these)

- `id` — UUID v4, auto-generated primary key
- `created_at` — ISO 8601 timestamp, auto-set
- `updated_at` — ISO 8601 timestamp, auto-updated
- `email` — auto-created on auth tables (string, required, unique)
- `password_hash` — auto-created on auth tables (hidden from API responses)

### Access levels

- `public` — no auth required
- `authenticated` — any logged-in user (default)
- `owner` — only the record owner (requires `owner_field`)
- `admin` — admin key only

### owner_field

Required when any access rule is `"owner"`. Set it to a ref column pointing to the auth table:

```json
"posts": {
  "columns": {
    "user_id": "ref users required",
    "title": "string required"
  },
  "access": { "read": "public", "update": "owner", "delete": "owner" },
  "owner_field": "user_id"
}
```

How it works:
- **Read/Update/Delete**: MoonDB compares `user_id` with the user id from the JWT — users only access their own rows
- **Insert**: if `user_id` is not in the request body, MoonDB auto-fills it from the Bearer token
- For the auth table itself, use `"owner_field": "id"` (a user owns their own row)

### Destructive changes

Dropping columns, changing types, or narrowing enums are destructive. MoonDB returns a preview first:

```json
{ "confirm_destructive": true }
```

Add this to the request body to apply destructive migrations.

### Validate without applying

Returns the same migration plan `set_schema` would, without touching the
database. Use to iterate on a complex schema or preview a destructive change.

REST:
```
POST https://api.moondb.ai/p/{project_id}/v1/schema/validate
X-Admin-Key: sk_...
```

MCP:
```
validate_schema { project_id, schema }
```

### Seed data after schema

Bulk insert with topological ordering and cross-reference resolution. Use
`"@<table>.<index>"` to point at a row inserted earlier in the same call.

REST:
```
POST https://api.moondb.ai/p/{project_id}/v1/schema/seed
X-Admin-Key: sk_...

{
  "users": [
    { "email": "a@x.io", "password": "secretpass1" },
    { "email": "b@x.io", "password": "secretpass2" }
  ],
  "posts": [
    { "title": "Hello",   "user_id": "@users.0" },
    { "title": "Another", "user_id": "@users.1" }
  ]
}
```

MCP:
```
seed { project_id, data: { table_name: [rows], ... } }
```

For `auth_table` rows send a plain `"password"` field — MoonDB hashes it
server-side into `password_hash`.

## Pre-flight: inspect before mutating

`set_schema` REPLACES the schema (auto-migrating non-destructive parts). Call
`get_schema` first when you're not sure what's already there — it returns the
current schema plus a compact `tables_info` listing per-table column names,
access rules, and `auth_table` / `owner_field` flags. Cheaper than diffing
in your head.

If you need a feature beyond what these skill notes cover, call
`get_reference` — it returns the full canonical MoonDB agent reference,
project-scoped when you pass `project_id` (includes the live schema).
