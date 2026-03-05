# Aprende Comigo Plugins

Bespoke Claude Code plugins for the [Aprende Comigo](https://github.com/aprendecomigo-edu) educational platform.

For upstream plugins maintained by Anthropic, install directly from [`anthropics/claude-plugins-official`](https://github.com/anthropics/claude-plugins-official).

## Plugins

| Plugin | Description |
|--------|-------------|
| [feature-dev](custom/feature-dev/) | Feature development and bug-fixing workflows with specialized agents for codebase exploration, architecture design, and quality review |
| [neon](custom/neon/) | Neon Serverless Postgres integration with MCP tools, Drizzle ORM skill, and a database workflow agent |
| [pr-review-toolkit](custom/pr-review-toolkit/) | PR review agents for comments, tests, error handling, type design, and code quality |
| [github](custom/github/) | GitHub MCP server with Aprende Comigo project board workflow (Backlog, In Progress, In Review, Done) |

## Installation

Install plugins via Claude Code's plugin system:

```
/plugin install {plugin-name}@aprende-comigo-plugins
```

Or browse in `/plugin > Discover`.

## Structure

```
custom/                          # All bespoke plugins
  feature-dev/                   # Feature dev & bug-fix workflows
  neon/                          # Neon Postgres + Drizzle ORM
  pr-review-toolkit/             # PR review agents
  github/                        # GitHub MCP + project board
.claude-plugin/marketplace.json  # Marketplace manifest
```

Each plugin follows the standard Claude Code plugin layout:

```
plugin-name/
├── .claude-plugin/
│   └── plugin.json      # Plugin metadata (required)
├── .mcp.json            # MCP server configuration (optional)
├── commands/            # Slash commands (optional)
├── agents/              # Agent definitions (optional)
├── skills/              # Skill definitions (optional)
├── LICENSE              # Apache 2.0
└── README.md            # Documentation
```

## License

All plugins are licensed under Apache 2.0. See each plugin's LICENSE file for details.
