# Aprende Comigo Plugin Marketplace

Curated fork of Anthropic's official Claude Code plugin marketplace, maintained by **Aprende Comigo**.

Upstream: `anthropics/claude-plugins-official`
Fork: `aprendecomigo-edu/aprende-comigo-plugins`

## Project Structure

```
.claude-plugin/marketplace.json  # Marketplace manifest (all plugin listings)
plugins/                         # Internal plugins (source code lives here)
external_plugins/                # External plugin configs (point to third-party repos)
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

### Adding a new internal plugin

1. Create a directory under `plugins/<plugin-name>/`
2. Add `.claude-plugin/plugin.json` manifest (minimum: `{"name": "plugin-name"}`)
3. Add an Apache 2.0 `LICENSE` file
4. Add a `README.md` with: overview, features, installation, prerequisites, usage
5. Implement components (commands, agents, skills, hooks, MCP) as needed
6. Register the plugin in `.claude-plugin/marketplace.json`
7. Run the `plugin-dev:plugin-validator` agent to validate before committing

### Adding an external plugin

1. Create a directory under `external_plugins/<plugin-name>/`
2. Add a `plugin.json` with the `source` pointing to the external repo
3. Register in `.claude-plugin/marketplace.json`

### Marketplace manifest rules

- Plugin `name` must be kebab-case (e.g., `my-plugin`, not `My Plugin`)
- Internal plugin `homepage` URLs must point to `aprendecomigo-edu/aprende-comigo-plugins`
- External plugin `homepage` URLs must not end in `.git`
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

**Agents**: Markdown files with YAML frontmatter (`description` with `<example>` blocks for reliable triggering, `model`, `color`, `tools`). Use the `plugin-dev:agent-creator` agent for AI-assisted generation. Validate with `validate-agent.sh`.

**Skills**: Each skill lives in its own subdirectory with a `SKILL.md`. Follow progressive disclosure: lean core doc (1,500-2,000 words), detailed references and examples in subdirectories. Descriptions must be third-person with specific trigger phrases. Body text in imperative/infinitive form.

**Hooks**: Configure in `hooks/hooks.json`. Prefer prompt-based hooks for complex logic. Use `${CLAUDE_PLUGIN_ROOT}` for portable paths. Events: PreToolUse, PostToolUse, Stop, SubagentStop, SessionStart, SessionEnd, UserPromptSubmit, PreCompact, Notification.

**MCP Servers**: Configure in `.mcp.json` at plugin root. Server types: stdio (local), SSE (hosted/OAuth), HTTP (REST), WebSocket (real-time). Use `${CLAUDE_PLUGIN_ROOT}` in command args. Document required environment variables in README.

### Portability

Always use `${CLAUDE_PLUGIN_ROOT}` for intra-plugin path references. Never use hardcoded absolute paths, home directory shortcuts, or relative paths from working directory.

## Workflow

Use the `plugin-dev` plugin (at `plugins/plugin-dev/`) for creating new plugins. It provides:

### Guided creation workflow
- `/plugin-dev:create-plugin` - 8-phase end-to-end workflow: Discovery, Component Planning, Detailed Design, Structure Creation, Implementation, Validation, Testing, Documentation

### Specialized skills (auto-activate on relevant queries)
- `plugin-structure` - Directory layout, manifest config, auto-discovery
- `hook-development` - Event hooks, prompt-based hooks, validation scripts
- `mcp-integration` - MCP server configuration, authentication patterns
- `command-development` - Slash commands, frontmatter, arguments
- `agent-development` - Agent creation, triggering, system prompts
- `skill-development` - Progressive disclosure, trigger descriptions
- `plugin-settings` - Configuration via `.claude/plugin-name.local.md`

### Validation agents
- `plugin-dev:plugin-validator` - Validate plugin structure, manifest, components
- `plugin-dev:skill-reviewer` - Review skill quality and triggering
- `plugin-dev:agent-creator` - AI-assisted agent generation

## License

Apache 2.0 - see [LICENSE](./LICENSE)
