# GitHub Plugin

Official GitHub MCP server integration extended with Aprende Comigo project management workflow support.

## Features

- **GitHub MCP Server**: Full GitHub API access for issues, PRs, code review, and repository management
- **Project Board Management**: Skill and agent for managing the Aprende Comigo GitHub Projects board (ProjectV2)
- **Workflow Automation**: Defined transitions for the development lifecycle (Backlog -> In Progress -> In Review -> Done)

## Prerequisites

### GitHub MCP Server

Set the `GITHUB_PERSONAL_ACCESS_TOKEN` environment variable with a token that has `repo` scope.

### Project Board Operations

The `gh` CLI must have the `read:project` scope for board queries and mutations:

```bash
gh auth refresh -h github.com -s read:project
```

Required CLI tools:
- `gh` (GitHub CLI)
- `jq` (JSON processing)

## Components

### MCP Server

Configured in `.mcp.json` -- connects to the official GitHub MCP server via HTTP for standard repository operations (issues, PRs, comments, labels, assignments).

### Skill: `ac-project-workflow`

Teaches Claude the Aprende Comigo project board workflow:
- Board structure (4 columns: Backlog, In Progress, In Review, Done)
- Team roles (developer, product manager)
- Transition rules and validation
- Dynamic discovery of field/option IDs via `gh` CLI

### Agent: `project-manager`

Autonomous agent for executing board operations:
- Start work on issues (Backlog -> In Progress)
- Post-merge handoff (In Progress -> In Review, reassign to PM)
- Board status queries
- Backlog triage

## Usage

### Check board status

```
Show me the current board status
```

### Start working on an issue

```
Start work on issue #452
```

### After merging a PR to staging

```
I just merged PR #456 to staging, update the board for issue #123
```

### Triage backlog

```
What's in the backlog?
```

## License

Apache 2.0
