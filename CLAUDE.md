# Aprende Comigo Plugin Marketplace

Curated fork of Anthropic's official Claude Code plugin marketplace, maintained by **Aprende Comigo**.

Upstream: `anthropics/claude-plugins-official`
Fork: `aprendecomigo-edu/claude-plugins-ap`

## Project Structure

```
.claude-plugin/marketplace.json  # Marketplace manifest (all plugin listings)
plugins/                         # Internal plugins (source code lives here)
external_plugins/                # External plugin configs (point to third-party repos)
```

## Plugin Development Conventions

### Adding a new internal plugin
1. Create a directory under `plugins/<plugin-name>/`
2. Add a `plugin.json` manifest inside the plugin directory
3. Add an Apache 2.0 `LICENSE` file
4. Add a `README.md` describing the plugin
5. Register the plugin in `.claude-plugin/marketplace.json`

### Adding an external plugin
1. Create a directory under `external_plugins/<plugin-name>/`
2. Add a `plugin.json` with the `source` pointing to the external repo
3. Register in `.claude-plugin/marketplace.json`

### Marketplace manifest rules
- Plugin `name` must be kebab-case (e.g., `my-plugin`, not `My Plugin`)
- Internal plugin `homepage` URLs must point to `aprendecomigo-edu/claude-plugins-ap`
- External plugin `homepage` URLs must not end in `.git`
- The `owner` object should not contain an `email` field

### Naming conventions
- Directories: kebab-case
- Plugin names in manifest: kebab-case
- Agent files: kebab-case with `.md` extension
- Skill files: kebab-case with `.md` extension

## Workflow

Use the `plugin-dev` plugin agents and skills for creating new plugins:
- `/plugin-dev:create-plugin` for guided end-to-end plugin creation
- `plugin-dev:plugin-validator` agent to validate plugin structure
- `plugin-dev:skill-reviewer` agent to review skill quality

## License

Apache 2.0 - see [LICENSE](./LICENSE)
