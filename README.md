# MoonDB Cursor Plugin

The official [MoonDB](https://moondb.ai) plugin for [Cursor](https://cursor.com). Get a working backend in 3 API calls — declarative schema, auto-generated REST API, built-in auth, file storage, and AI endpoints.

## Installation

Install from the Cursor Marketplace: search for **MoonDB** in Cursor Settings > Plugins.

Or clone and install locally:
```bash
git clone https://github.com/moondb-ai/cursor.git ~/.cursor/plugins/moondb
```

## Setup

1. Sign up at [moondb.ai](https://moondb.ai) or via the API:
   ```
   POST https://api.moondb.ai/v1/accounts/signup
   { "email": "you@example.com", "password": "..." }
   ```

2. Set your API key in Cursor Settings > MCP > moondb > Environment Variables:
   ```
   MOONDB_API_KEY=mk_...
   ```

## What's included

### MCP Server

Connects to MoonDB's remote MCP endpoint. On `initialize` the server returns
an `instructions` field that briefs the model on MoonDB conventions
(auth_table, owner_field, ref types, no SQL columns, confirm_destructive)
plus the REST surfaces (`/auth/*`, `/storage/*`, `/ai/*`) the generated app
code should call — so the agent doesn't churn through wrong-column-type
retries.

**Project management**

| Tool             | Description                                      |
|------------------|--------------------------------------------------|
| `create_project` | Create a new project; returns admin + public keys (admin shown once) |
| `list_projects`  | List every project the calling account owns       |
| `rotate_keys`    | Rotate `admin_key` and/or `public_key`            |

**Schema**

| Tool             | Description                                      |
|------------------|--------------------------------------------------|
| `set_schema`     | Apply a JSON schema with auto-diff & migration   |
| `validate_schema`| Dry-run: parse + diff without mutating           |
| `get_schema`     | Fetch current schema + tables_info summary       |
| `seed`           | Bulk insert with topological order and `@table.index` cross-refs |
| `get_reference`  | Full agent reference (same source as `/v1/llm-context`) |

**Data CRUD**

| Tool         | Description                                          |
|--------------|------------------------------------------------------|
| `query`      | Filtered SELECT (full operator set, sort, select)    |
| `get_row`    | Fetch one row by id                                  |
| `insert`     | Insert one or many records                           |
| `update_row` | PATCH-semantics update by id                         |
| `delete_row` | Delete by id (soft if `schema.options.soft_delete`)  |

**AI**

| Tool      | Description                                           |
|-----------|-------------------------------------------------------|
| `ai_call` | Invoke an AI endpoint declared under `ai_endpoints`   |

End-user auth (`/auth/*`), file upload/download (`/storage/*`), and runtime
AI calls from the user-facing app are intentionally NOT MCP tools — the
agent generates `fetch()` calls in the user's app code that hit those REST
endpoints. The `initialize.instructions` primer documents that contract.

### Rules

- **moondb-conventions** — Schema and API conventions (always active)
- **moondb-api** — REST API patterns and query syntax (active in code files)

### Skills

- **setup-project** — Create a project, get keys, configure environment
- **define-schema** — Design tables, columns, auth, and access rules
- **crud-operations** — Query, create, update, and delete records
- **auth** — Signup, login, tokens, email verification, external auth
- **file-storage** — Upload, download, and manage files
- **ai-endpoints** — Define and call AI text/image generation endpoints

## Quick start

Tell the Cursor agent:

> "Create a MoonDB backend for a todo app with user authentication"

The agent will use the MCP tools and skills to:
1. Create a project
2. Define a schema with users and todos tables
3. Set up auth and access rules
4. Return the API keys and endpoints

## Links

- [Documentation](https://docs.moondb.ai)
- [Dashboard](https://moondb.ai/dashboard)
- [GitHub](https://github.com/moondb-ai)
