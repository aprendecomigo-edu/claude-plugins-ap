# Workflow Transitions Decision Table

## Transition Matrix

| Current Status | Target Status | Trigger | Actor | Steps |
|---|---|---|---|---|
| Backlog | In Progress | Dev starts work on issue | Developer (anaoaktree) | Assign dev, move card |
| In Progress | In Review | PR merged to staging | Developer (anaoaktree) | Move card, reassign to PM, comment |
| In Review | Done | PM approves on staging | PM (joana-aprendecomigo) | Move card (PM action, not automated) |
| In Review | In Progress | PM requests changes | PM (joana-aprendecomigo) | Reassign to dev, move card back |

## Detailed Transition Steps

### Backlog -> In Progress

**Who initiates**: Developer says "start work on issue #N" or similar.

**Pre-checks**:
- Issue exists on the project board
- Issue is currently in Backlog
- Issue is not already assigned to someone else actively working on it

**Execute**:
1. `gh issue edit N --repo aprendecomigo-edu/aprende-comigo --add-assignee anaoaktree`
2. Discover field/option IDs (see board-commands.md)
3. Move item to "In Progress" via `gh project item-edit`

**Post-check**: Query the board to confirm status changed.

### In Progress -> In Review

**Who initiates**: Developer says "PR merged to staging" or "issue #N is ready for review" or similar.

**Pre-checks**:
- Issue is currently in In Progress
- A PR linked to this issue has been merged (verify if possible)

**Execute**:
1. Discover field/option IDs
2. Move item to "In Review" via `gh project item-edit`
3. `gh issue edit N --repo aprendecomigo-edu/aprende-comigo --remove-assignee anaoaktree`
4. `gh issue edit N --repo aprendecomigo-edu/aprende-comigo --add-assignee joana-aprendecomigo`
5. `gh issue comment N --repo aprendecomigo-edu/aprende-comigo --body "Moved to In Review. Ready for testing on staging. @joana-aprendecomigo"`

**Post-check**: Query the board to confirm status and assignee changed.

### In Review -> Done

**Note**: This is a PM action. Claude should not execute this automatically. If the developer requests it, confirm with them first since this bypasses PM review.

### In Review -> In Progress (Rework)

**Who initiates**: PM requests changes, or developer recognizes rework is needed.

**Execute**:
1. Move item back to "In Progress"
2. `gh issue edit N --repo aprendecomigo-edu/aprende-comigo --remove-assignee joana-aprendecomigo`
3. `gh issue edit N --repo aprendecomigo-edu/aprende-comigo --add-assignee anaoaktree`
4. Comment noting the rework reason if provided

## Invalid Transitions

These transitions should be rejected or require explicit confirmation:

| Attempted | Why Invalid | Action |
|---|---|---|
| Done -> any | Reopening completed work | Ask user to confirm and provide reason |
| Backlog -> In Review | Skipping development | Reject; must go through In Progress |
| Backlog -> Done | Skipping all stages | Reject; must follow the pipeline |
| In Progress -> Done | Skipping PM review | Warn user this bypasses PM review; require confirmation |

## Batch Operations

When multiple issues are involved (e.g., sprint planning):

- **Triage backlog**: List backlog items, let user select which to move to In Progress
- **Post-merge batch**: If multiple issues are linked to a merged PR, transition all of them
- **Sprint review**: List all "In Review" items for PM status check
