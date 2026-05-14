---
name: crud-operations
description: Query, create, update, and delete records via MoonDB REST API. Use when implementing data operations in app code.
---

# CRUD Operations

## When to use

- Implementing data fetching, creation, updates, or deletion in application code
- Building API integration with a MoonDB backend

## Base URL

```
https://api.moondb.ai/p/{project_id}/api/{table}
```

## Authentication headers

- Public reads: `X-Public-Key: pk_...`
- Authenticated operations: `Authorization: Bearer {user_token}`
- Admin operations: `X-Admin-Key: sk_...`

## List records

```
GET /api/{table}
GET /api/{table}?field=op.value&sort=field.desc&limit=20&offset=0
```

### Filter operators

| Operator  | Example                  | Description          |
|-----------|--------------------------|----------------------|
| eq        | ?status=eq.active        | Equal (default)      |
| neq       | ?status=neq.draft        | Not equal            |
| gt        | ?age=gt.18               | Greater than         |
| gte       | ?price=gte.10            | Greater than or equal|
| lt        | ?stock=lt.5              | Less than            |
| lte       | ?rating=lte.3            | Less than or equal   |
| like      | ?name=like.%john%        | SQL LIKE pattern     |
| in        | ?status=in.active,draft  | In list              |
| is_null   | ?deleted_at=is_null      | Is NULL              |
| not_null  | ?avatar=not_null         | Is not NULL          |

### Sorting and pagination

```
?sort=created_at.desc
?limit=20&offset=40
```

### Field selection and includes

```
?select=id,title,created_at
?include=user_id
```

`include` expands ref columns into full related objects.

### Response format

```json
{
  "data": [{ "id": "...", "title": "...", "created_at": "..." }],
  "meta": { "total": 42, "limit": 20, "offset": 0, "has_more": true }
}
```

## Get single record

```
GET /api/{table}/{id}
```

Response: `{ "data": { ...row } }`

## Create record

```
POST /api/{table}
Content-Type: application/json

{ "title": "New post", "body": "Content here" }
```

Response: `{ "data": { "id": "...", ...row } }`

## Update record

```
PATCH /api/{table}/{id}
Content-Type: application/json

{ "title": "Updated title" }
```

## Delete record

```
DELETE /api/{table}/{id}
```

## Bulk insert

```
POST /api/{table}/bulk
Content-Type: application/json

[
  { "title": "Post 1", "body": "..." },
  { "title": "Post 2", "body": "..." }
]
```

Bulk inserts are atomic — all succeed or all fail. Max 100 rows per request.

## Error handling

Every error includes a `suggestion` field:

```json
{
  "error": {
    "code": "VALIDATION_REQUIRED_FIELD",
    "message": "Field 'title' is required",
    "suggestion": "Add 'title' to the request body"
  }
}
```

Always read the `suggestion` to self-correct.
