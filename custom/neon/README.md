# Neon Plugin

Neon Serverless Postgres integration for Claude Code. Provides direct database management via MCP, best-practice guidance via skills, and a specialized Drizzle ORM + Neon database expert agent.

## Features

- **MCP Integration**: Direct access to Neon's platform API for project, branch, and database management
- **Neon Postgres Skill**: Comprehensive guidance on Neon architecture, connection methods, branching, scaling, and more
- **Drizzle ORM Skill**: Type-safe schema declaration, query patterns, migration workflows, Neon drivers, and Zod integration
- **DB Expert Agent**: Specialized agent for Drizzle ORM + Neon database workflows (schema design, migrations, queries, performance tuning)

## Prerequisites

- A [Neon](https://neon.tech) account
- Neon MCP server authentication (configured automatically via OAuth)

## Installation

```bash
claude --plugin-dir /path/to/plugins/neon
```

Or add to your project's plugin configuration.

## Usage

### MCP Tools

The Neon MCP server provides tools for:
- Creating and managing projects and branches
- Running SQL queries and transactions
- Schema migrations with branch-based testing
- Query performance tuning
- Database schema comparison

### Skills

Ask Neon or Drizzle questions and the relevant skill activates automatically:
- "How do I connect to Neon from a serverless function?"
- "Set up Neon branching for preview deployments"
- "How do I define relations in Drizzle?"
- "What's the difference between drizzle-kit push and migrate?"

### Agent

Use the `neon:db-expert` agent for Drizzle ORM + Neon tasks:
- Schema design with Drizzle's type-safe definitions
- Migration workflows using `drizzle-kit` + Neon branching
- Connection setup (HTTP vs WebSocket drivers)
- Query building with Drizzle's relational API
- Performance tuning with EXPLAIN analysis

## Components

| Component | Description |
|-----------|-------------|
| `skills/neon-postgres/` | Neon platform guidance and documentation references |
| `skills/drizzle-orm/` | Drizzle ORM schema, queries, migrations, and driver patterns |
| `agents/db-expert.md` | Drizzle ORM + Neon specialized database expert agent |
| `.mcp.json` | Neon MCP server configuration |
