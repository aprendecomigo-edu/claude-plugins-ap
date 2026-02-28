---
name: ac-project-workflow
description: This skill should be used when the user asks to "check the board", "start work on issue", "update board after merge", "move issue to in review", "triage backlog", "show what's in progress", "reassign to PM", "update project board", or needs guidance on the Aprende Comigo GitHub Projects workflow, board transitions, or issue lifecycle management.
version: 0.1.0
---

# Aprende Comigo Project Board Workflow

## Overview

Manage the Aprende Comigo development workflow using GitHub Projects (ProjectV2). The project board tracks all development work through a kanban-style pipeline with defined team roles and transition rules.

**Key facts:**
- **Organization**: `aprendecomigo-edu`
- **Repository**: `aprendecomigo-edu/aprende-comigo`
- **Project board**: Project #2 (URL: `https://github.com/orgs/aprendecomigo-edu/projects/2`)
- **Board columns** (Status field): Backlog, In Progress, In Review, Done

## Team Roles

| GitHub Handle | Role | Responsibilities |
|---|---|---|
| `anaoaktree` | Developer | Writes code, creates PRs, moves issues to In Progress |
| `joana-aprendecomigo` | Product Manager | Reviews completed work on staging, approves/rejects, moves to Done |

Claude acts as a development partner to `anaoaktree`, executing board operations on their behalf.

## Tool Selection

Two tools are available for GitHub operations. Use the right one for each task:

**`gh` CLI (via Bash)** -- Required for all board/project operations:
- Moving cards between columns
- Querying board status
- Fetching field IDs, option IDs, milestones, labels
- Any ProjectV2 GraphQL operation

**GitHub MCP server** -- Use for standard repo operations:
- Creating/editing issues and PRs
- Adding comments
- Assigning users
- Adding labels
- Searching issues

The GitHub MCP server does NOT support ProjectV2 operations. Always use `gh` CLI for anything board-related.

## Board Transitions

There are two primary workflow transitions. Both require specific steps executed in order.

### Transition 1: Start Work on Issue

**Trigger**: Developer picks an issue from Backlog to begin working on it.

**Steps**:
1. Assign the issue to `anaoaktree` (if not already assigned)
2. Move the issue from **Backlog** to **In Progress** on the project board
3. Report the before/after state

### Transition 2: Post-Merge Handoff to PM

**Trigger**: A PR has been merged to the `staging` branch, and the linked issue needs review.

**Steps**:
1. Move the issue from **In Progress** to **In Review** on the project board
2. Remove `anaoaktree` as assignee
3. Assign `joana-aprendecomigo` as the new assignee
4. Add a comment on the issue noting it's ready for review on staging
5. Report the before/after state

## Dynamic Discovery

Field IDs, option IDs, milestones, and labels change over time. Never hardcode them. Always discover them at runtime before performing operations.

See `references/board-commands.md` for the exact commands to discover and use these values.

## Querying Board Status

Common queries the developer will request:
- "What's in progress?" -- List all items in the In Progress column
- "Show the board" -- List items in each column with assignees
- "What's in the backlog?" -- List Backlog items, optionally filtered by milestone or label

All queries use the `gh project item-list` command with `--format json` and `jq` for filtering.

## Validation Rules

Before executing any transition:
1. **Verify the issue exists** on the project board
2. **Check current status** matches expected source column
3. **Confirm the transition makes sense** (don't move Done items to Backlog without asking)
4. **Report what will change** before executing if the action is ambiguous

After executing:
1. **Verify the new state** by querying the board
2. **Report the result** to the user with before/after summary

## Reference Documents

- `references/board-commands.md` -- CLI commands for discovery, board ops, and issue ops
- `references/workflow-transitions.md` -- Decision table for all transitions
