# Supabase Plugin

Supabase MCP integration with a Drizzle ORM expert agent for database operations, schema design, migrations, RLS policies, authentication, and storage.

## Features

- **Supabase MCP Server**: Direct access to Supabase operations (SQL queries, schema inspection, table management) via the hosted MCP endpoint
- **Supabase + Drizzle Expert Agent**: Specialized agent for database schema design, migrations, connection management, RLS policies, and Supabase Auth integration

## Prerequisites

- A Supabase project (the MCP URL is pre-configured with the Aprende Comigo project reference)
- Supabase MCP authentication (OAuth flow will be triggered on first use)

## Installation

Install via the Claude Code plugin marketplace or add directly:

```bash
claude plugin add supabase
```

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
- Row Level Security policies
- Supabase Auth (magic links, OTP, JWT claims)
- Supabase Storage bucket configuration

**Example prompts:**

- "Design a Drizzle schema for the new scheduling feature"
- "Help me write a migration to add a notifications table"
- "How should I configure Supabase connection pooling with Drizzle for serverless?"
- "Create RLS policies for the payments table"

## Configuration

The MCP server URL includes the project reference for the Aprende Comigo Supabase project. To use a different project, update the `project_ref` parameter in `.mcp.json`.

## License

Apache 2.0
