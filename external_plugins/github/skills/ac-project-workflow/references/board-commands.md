# Board Commands Reference

All commands use the `gh` CLI. The GitHub MCP server does not support ProjectV2 operations.

## Prerequisites

The `gh` CLI must have the `read:project` scope:

```bash
gh auth refresh -h github.com -s read:project
```

## Discovery Commands

These commands fetch dynamic values (field IDs, option IDs, milestones, labels) that change over time. Always run discovery before board operations.

### List projects for the org

```bash
gh project list --owner aprendecomigo-edu --format json
```

### Get project field IDs

Fetch all fields and their option IDs for project #2:

```bash
gh project field-list 2 --owner aprendecomigo-edu --format json
```

This returns all fields. Look for the "Status" field to get:
- The field ID (needed for updates)
- Option IDs for each column (Backlog, In Progress, In Review, Done)

### Parse Status field options with jq

```bash
gh project field-list 2 --owner aprendecomigo-edu --format json | jq '.fields[] | select(.name == "Status")'
```

### Get milestones for the repo

```bash
gh api repos/aprendecomigo-edu/aprende-comigo/milestones --jq '.[] | {number, title, state}'
```

### Get labels for the repo

```bash
gh api repos/aprendecomigo-edu/aprende-comigo/labels --jq '.[] | {name, color, description}'
```

## Board Operations

### List all items on the board

```bash
gh project item-list 2 --owner aprendecomigo-edu --format json
```

### Filter items by status column

Use jq to filter. Example for "In Progress":

```bash
gh project item-list 2 --owner aprendecomigo-edu --format json | jq '.items[] | select(.status == "In Progress")'
```

### Move an item to a different column

Requires: the project item ID, the Status field ID, and the target option ID (all discovered dynamically).

```bash
gh project item-edit --project-id <PROJECT_ID> --id <ITEM_ID> --field-id <STATUS_FIELD_ID> --single-select-option-id <OPTION_ID>
```

**Full workflow to move issue #N to "In Progress":**

```bash
# 1. Get the project ID (number -> node ID)
PROJECT_ID=$(gh project list --owner aprendecomigo-edu --format json | jq -r '.projects[] | select(.number == 2) | .id')

# 2. Get the Status field ID and option IDs
STATUS_FIELD=$(gh project field-list 2 --owner aprendecomigo-edu --format json | jq -r '.fields[] | select(.name == "Status")')
FIELD_ID=$(echo "$STATUS_FIELD" | jq -r '.id')
OPTION_ID=$(echo "$STATUS_FIELD" | jq -r '.options[] | select(.name == "In Progress") | .id')

# 3. Find the item ID for the issue
ITEM_ID=$(gh project item-list 2 --owner aprendecomigo-edu --format json | jq -r '.items[] | select(.content.number == N) | .id')

# 4. Move it
gh project item-edit --project-id "$PROJECT_ID" --id "$ITEM_ID" --field-id "$FIELD_ID" --single-select-option-id "$OPTION_ID"
```

### Add an issue to the project board

If an issue is not yet on the board:

```bash
gh project item-add 2 --owner aprendecomigo-edu --url https://github.com/aprendecomigo-edu/aprende-comigo/issues/N
```

## Issue Operations (via gh CLI)

### Assign a user to an issue

```bash
gh issue edit N --repo aprendecomigo-edu/aprende-comigo --add-assignee anaoaktree
```

### Remove a user from an issue

```bash
gh issue edit N --repo aprendecomigo-edu/aprende-comigo --remove-assignee anaoaktree
```

### Add a label to an issue

```bash
gh issue edit N --repo aprendecomigo-edu/aprende-comigo --add-label "label-name"
```

### Add a comment to an issue

```bash
gh issue comment N --repo aprendecomigo-edu/aprende-comigo --body "Comment text here"
```

### View issue details

```bash
gh issue view N --repo aprendecomigo-edu/aprende-comigo
```

## Notes

- Replace `N` with the actual issue number in all commands
- All IDs (project, field, option, item) are opaque strings -- always discover them, never hardcode
- The `--format json` flag is essential for programmatic parsing
- Use `jq` for filtering and extracting values from JSON output
- If a command fails with a scope error, run `gh auth refresh -h github.com -s read:project`
