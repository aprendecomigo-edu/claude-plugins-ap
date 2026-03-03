# Supabase Plugin

Supabase MCP integration with a Drizzle ORM expert agent for database operations, schema design, migrations, multi-tenant data isolation, authentication, and storage.

## Features

- **Supabase MCP Server**: Direct access to Supabase operations (SQL queries, schema inspection, table management) via the hosted MCP endpoint
- **Supabase + Drizzle Expert Agent**: Specialized agent for database schema design, migrations, connection management, TypeScript permission guards, and Supabase Auth integration

## Prerequisites

- A Supabase project
- Supabase MCP authentication (OAuth flow will be triggered on first use)

## Installation

Install via the Claude Code plugin marketplace or add directly:

```bash
claude plugin add supabase
```

## Setup

After installation, update the `project_ref` in `.mcp.json` with your Supabase project reference:

```json
{
  "supabase": {
    "type": "http",
    "url": "https://mcp.supabase.com/mcp?project_ref=YOUR_PROJECT_REF"
  }
}
```

You can find your project reference in the Supabase dashboard under Project Settings > General.

## Usage

### MCP Server

The Supabase MCP server is automatically available after installation. Use it for:

- Running SQL queries against your database
- Inspecting table schemas and relationships
- Managing database objects

### Supabase + Drizzle Expert Agent

The `supabase-drizzle-expert` agent activates when you ask about:

- Drizzle ORM schema design and relations
- Database migrations with drizzle-kit
- Supabase connection pooling configuration
- TypeScript permission guards and multi-tenant data isolation
- Supabase Auth (magic links, OTP, JWT claims)
- Supabase Storage bucket configuration

**Example prompts:**

- "Design a Drizzle schema for the new scheduling feature"
- "Help me write a migration to add a notifications table"
- "How should I configure Supabase connection pooling with Drizzle for serverless?"
- "Add a permission check for the payments query so teachers can only see their own records"

## License

Apache 2.0
