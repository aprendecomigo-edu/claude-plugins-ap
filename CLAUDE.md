# Aprende Comigo Plugins

Bespoke Claude Code plugins by **Aprende Comigo**. For upstream plugins, install directly from `anthropics/claude-plugins-official`.

## Project Structure

```
.claude-plugin/marketplace.json  # Marketplace manifest (all plugin listings)
custom/                          # Bespoke plugins (our own code)
```

Each plugin follows a standardized directory layout with auto-discovery:

```
plugin-name/
├── .claude-plugin/
│   └── plugin.json          # Required: plugin manifest (name is the only required field)
├── commands/                # Slash commands (.md files with YAML frontmatter)
├── agents/                  # Subagent definitions (.md files with YAML frontmatter)
├── skills/                  # Agent skills (subdirectories, each with SKILL.md)
│   └── skill-name/
│       ├── SKILL.md         # Required for each skill
│       ├── references/      # Detailed guides and patterns
│       ├── examples/        # Working code examples
│       └── scripts/         # Utility scripts
├── hooks/
│   ├── hooks.json           # Event handler configuration
│   └── scripts/             # Hook scripts
├── .mcp.json                # MCP server definitions (stdio, SSE, HTTP, WebSocket)
├── LICENSE                  # Apache 2.0
└── README.md                # Plugin documentation
```

Component directories must be at plugin root level, not nested inside `.claude-plugin/`. Only create directories for components the plugin actually uses.

## Plugin Development Conventions

### Adding a new plugin

1. Create a directory under `custom/<plugin-name>/`
2. Add `.claude-plugin/plugin.json` manifest (minimum: `{"name": "plugin-name"}`)
3. Add an Apache 2.0 `LICENSE` file
4. Add a `README.md` with: overview, features, installation, prerequisites, usage
5. Implement components (commands, agents, skills, hooks, MCP) as needed
6. Register the plugin in `.claude-plugin/marketplace.json`

### Marketplace manifest rules

- Plugin `name` must be kebab-case (e.g., `my-plugin`, not `My Plugin`)
- Plugin `homepage` URLs must point to `aprendecomigo-edu/aprende-comigo-plugins`
- The `owner` object should not contain an `email` field
- Validate JSON after edits: `cat .claude-plugin/marketplace.json | python3 -m json.tool`

### Naming conventions

- Directories: kebab-case
- Plugin names in manifest: kebab-case
- Command files: kebab-case `.md` (e.g., `review-pr.md` becomes `/review-pr`)
- Agent files: kebab-case `.md` describing role (e.g., `code-reviewer.md`)
- Skill directories: kebab-case topic names (e.g., `api-testing/`)
- Scripts: descriptive kebab-case with appropriate extensions
- Config files: standard names (`hooks.json`, `.mcp.json`, `plugin.json`)

### Component guidelines

**Commands**: Markdown files with YAML frontmatter (`description`, `argument-hint`, `allowed-tools`). Write instructions FOR Claude, not TO the user. Specify minimal `allowed-tools`.

**Agents**: Markdown files with YAML frontmatter (`description` with `<example>` blocks for reliable triggering, `model`, `color`, `tools`). Validate with `validate-agent.sh`.

**Skills**: Each skill lives in its own subdirectory with a `SKILL.md`. Follow progressive disclosure: lean core doc (1,500-2,000 words), detailed references and examples in subdirectories. Descriptions must be third-person with specific trigger phrases. Body text in imperative/infinitive form.

**Hooks**: Configure in `hooks/hooks.json`. Prefer prompt-based hooks for complex logic. Use `${CLAUDE_PLUGIN_ROOT}` for portable paths. Events: PreToolUse, PostToolUse, Stop, SubagentStop, SessionStart, SessionEnd, UserPromptSubmit, PreCompact, Notification.

**MCP Servers**: Configure in `.mcp.json` at plugin root. Server types: stdio (local), SSE (hosted/OAuth), HTTP (REST), WebSocket (real-time). Use `${CLAUDE_PLUGIN_ROOT}` in command args. Document required environment variables in README.

### Portability

Always use `${CLAUDE_PLUGIN_ROOT}` for intra-plugin path references. Never use hardcoded absolute paths, home directory shortcuts, or relative paths from working directory.

## Aprende Comigo Overview

Aprende Comigo is a comprehensive platform designed for educational institutions and individual tutors to manage their operations. Key features include:

- Multi-tenant architecture with school-based isolation via specialized role tables
- Role-based access control for school admins, teachers, and staff
- Student and teacher management
- Scheduling and calendar management
- Financial operations (payments, teacher compensation)
- Notifications
- Multi-language support (English UK, Portuguese PT)

### Current Stack (Next.js + Supabase)

- **Framework**: Next.js 15.5.x with App Router and TypeScript 5.9.x
- **Runtime**: Node.js 22 LTS
- **Database**: PostgreSQL via Drizzle ORM 0.45.x (hosted on Supabase)
- **Authentication**: Supabase Auth with magic links and OTP (email/SMS)
- **Styling**: Tailwind CSS v4 + DaisyUI v5
- **Icons**: Lucide React
- **Internationalization**: next-intl v4.6.x
- **Validation**: Zod v4.2.x (schemas in `lib/schemas/`)
- **Testing**: Playwright (E2E/BDD) + Vitest/React Testing Library (components)
