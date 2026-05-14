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

Connects to MoonDB's remote MCP endpoint, giving the AI agent direct access to:

| Tool             | Description                                      |
|------------------|--------------------------------------------------|
| `create_project` | Create a new project and get API keys             |
| `set_schema`     | Apply a declarative JSON schema with auto-migration |
| `get_schema`     | Fetch current schema and table info               |
| `query`          | Run filtered SELECT queries                       |
| `insert`         | Insert one or many records                        |
| `ai_call`        | Invoke AI endpoints defined in the schema         |

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
