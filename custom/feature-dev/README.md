# Feature Development Plugin

Feature development and bug-fixing workflows with specialized agents for codebase exploration, architecture design, and quality review. Optimized for Next.js + Neon Serverless Postgres projects with awareness of authorization, multi-tenancy, internationalization, and validation patterns.

## Overview

The Feature Development Plugin provides two workflows:

- **`/feature-dev`** — A guided 7-phase approach to building new features, with user checkpoints for clarification, architecture decisions, and review
- **`/feature-dev:bug-fix`** — An autonomous 6-phase workflow for diagnosing and fixing bugs, running end-to-end without user intervention

## Philosophy

Building features requires more than just writing code. You need to:
- **Understand the codebase** before making changes
- **Ask questions** to clarify ambiguous requirements
- **Design thoughtfully** before implementing
- **Review for quality** after building

This plugin embeds these practices into structured workflows that run automatically when you use the `/feature-dev` or `/feature-dev:bug-fix` commands.

## Command: `/feature-dev`

Launches a guided feature development workflow with 7 distinct phases.

**Usage:**
```bash
/feature-dev Add student enrollment management
```

Or simply:
```bash
/feature-dev
```

The command will guide you through the entire process interactively.

## The 7-Phase Workflow

### Phase 1: Discovery

**Goal**: Understand what needs to be built

**What happens:**
- Clarifies the feature request if it's unclear
- Asks what problem you're solving
- Identifies constraints and requirements
- Summarizes understanding and confirms with you

### Phase 2: Codebase Exploration

**Goal**: Understand relevant existing code and patterns

**What happens:**
- Launches 2-3 `code-explorer` agents in parallel
- Each agent explores different aspects (similar features, architecture, authorization patterns, data models)
- Agents return comprehensive analyses with key files to read
- Claude reads all identified files to build deep understanding
- Presents comprehensive summary of findings

### Phase 3: Clarifying Questions

**Goal**: Fill in gaps and resolve all ambiguities

**What happens:**
- Reviews codebase findings and feature request
- Checks a structured checklist covering: permissions, multi-tenancy, account types, i18n, validation, error cases, testing
- Presents all questions in an organized list
- **Waits for your answers before proceeding**

### Phase 4: Architecture Design

**Goal**: Design multiple implementation approaches

**What happens:**
- Launches 2-3 `code-architect` agents with different focuses
- Each architect checks established patterns for auth, i18n, validation, and error handling
- Reviews all approaches and forms a recommendation
- Presents comparison with trade-offs
- **Asks which approach you prefer**

### Phase 5: Implementation

**Goal**: Build the feature

**What happens:**
- Runs a pre-implementation checklist (auth guards, Zod schemas, translation keys, error handling, school-scoped queries)
- **Waits for explicit approval** before starting
- Implements following chosen architecture and codebase conventions
- Tracks progress with todos

### Phase 6: Quality Review

**Goal**: Ensure code is simple, DRY, elegant, and functionally correct

**What happens:**
- Launches 3 `code-reviewer` agents with specific focuses:
  - **Simplicity & DRY**: Code quality and maintainability
  - **Security & Authorization**: Auth guards, multi-tenant isolation, input validation
  - **Project Conventions**: i18n, Zod, error handling, DaisyUI, Drizzle patterns
- Consolidates findings and identifies highest severity issues
- **Presents findings and asks what you want to do**

### Phase 7: Summary

**Goal**: Document what was accomplished

**What happens:**
- Summarizes what was built, key decisions, files modified, and next steps

## Agents

### `code-explorer`

**Purpose**: Deeply analyzes existing codebase features by tracing execution paths, mapping architecture layers, and documenting dependencies.

**Stack-aware**: Traces through App Router pages, server actions, services, Drizzle schemas, permissions, i18n, and components systematically.

**When triggered**: Automatically in Phase 2, or manually when exploring code.

### `code-architect`

**Purpose**: Designs feature architectures with comprehensive implementation blueprints.

**Architecture checklist**: Covers data layer (`lib/db/schema/` + `drizzle/migrations/` + `lib/schemas/`), authorization (`lib/permissions/`), server actions, service layer (`lib/services/`), UI (DaisyUI + i18n + notifications), testing, and observability.

**When triggered**: Automatically in Phase 4, or manually for architecture design.

### `code-reviewer`

**Purpose**: Reviews code for bugs, security vulnerabilities, and project convention adherence using confidence-based filtering.

**Project-specific criteria**: Applies baseline confidence scores for multi-tenant isolation (95+), authorization guards (90+), input validation (85+), error handling (85+), i18n (80+), Drizzle patterns (75+), and DaisyUI conventions (70+).

**When triggered**: Automatically in Phase 6, or manually after writing code.

## Skills

### `server-action-patterns`

Guides creation of server actions with proper authorization guards, Zod validation, error handling, and service delegation.

**Triggers**: "create a server action", "add a form action", "implement a mutation"

**Covers**: requireAuth/requireSchoolRole, Zod validation, createErrorResponse/createSuccessResponse, service delegation, transaction patterns

### `db-schema-patterns`

Guides Drizzle ORM schema design with multi-tenant isolation, migrations, relations, and indexes.

**Triggers**: "add a database table", "create a schema", "add a migration"

**Covers**: Drizzle schema definitions, school_id isolation, two-file migration structure, relations, indexes, student account types

### `page-component-patterns`

Guides creation of App Router pages with DaisyUI components, next-intl integration, form handling, and notification patterns.

**Triggers**: "add a new page", "create a form", "add a UI component"

**Covers**: App Router page structure, DaisyUI v5, next-intl, Zod form validation, useNotification, responsive design, loading/error states

## Command: `/feature-dev:bug-fix`

Launches an autonomous bug diagnosis and fix workflow with 6 phases. Accepts flexible input: free text, GitHub issue numbers, or issue URLs.

**Usage:**
```bash
/feature-dev:bug-fix Login page shows blank screen after auth
/feature-dev:bug-fix #42
/feature-dev:bug-fix https://github.com/aprendecomigo-edu/aprendecomigo/issues/42
```

### The 6-Phase Workflow

#### Phase 1: Bug Understanding
Parses input (detects issue numbers, URLs, or free text), fetches issue details via `gh issue view` if applicable, and produces a structured bug summary.

#### Phase 2: Bug Localization
Launches 2-3 `code-explorer` agents targeting different diagnostic angles. Analyzes git history (`git log`, `git blame`) on suspect files. Produces a localization report.

#### Phase 3: Root Cause Analysis
Forms and verifies a hypothesis. Checks common Aprende Comigo bug patterns (missing `school_id` filter, missing auth guard, Zod schema mismatch, i18n key issues, Drizzle misconfiguration, App Router caching, error handling gaps).

#### Phase 4: Fix Design
Designs the minimal fix, lists files to modify, assesses side effects, and proceeds directly to implementation.

#### Phase 5: Implementation
Applies the fix following codebase conventions. Runs pre-implementation checklist (auth guards, Zod validation, translation keys, school-scoped queries, error handling). Executes related tests.

#### Phase 6: Verification & Summary
Launches 2 `code-reviewer` agents (correctness + security/conventions). Auto-fixes critical issues. Re-runs tests. Presents final summary with risk assessment.

**Key differences from `/feature-dev`:**
- Fully autonomous — no user checkpoints
- No architecture phase — bug fixes should be minimal, not architectural
- GitHub integration for input parsing
- Git analysis for diagnosis
- Self-correcting — auto-fixes critical review findings

## Usage Patterns

### Full workflow (recommended for new features):
```bash
/feature-dev Add teacher schedule management with availability and booking
```

### Bug fix (recommended for bug reports and issues):
```bash
/feature-dev:bug-fix Students from school A can see school B data on the dashboard
```

### Manual agent invocation:

**Explore a feature:**
```
"Launch code-explorer to trace how student enrollment works"
```

**Design architecture:**
```
"Launch code-architect to design the payment processing flow"
```

**Review code:**
```
"Launch code-reviewer to check my recent changes"
```

## Best Practices

1. **Use the full workflow for complex features**: The 7 phases ensure thorough planning
2. **Use bug-fix for bugs**: The 6 autonomous phases handle diagnosis through verification
3. **Answer clarifying questions thoughtfully**: Phase 3 of `/feature-dev` prevents future confusion
4. **Choose architecture deliberately**: Phase 4 gives you options for a reason
5. **Don't skip code review**: Both workflows include review phases
6. **Read the suggested files**: Phase 2 identifies key files—read them to understand context

## When to Use This Plugin

**Use `/feature-dev` for:**
- New features that touch multiple files
- Features requiring architectural decisions
- Complex integrations with existing code
- Features where requirements are somewhat unclear

**Use `/feature-dev:bug-fix` for:**
- Bug reports from users or QA
- GitHub issues that need investigation
- Unexpected behavior that needs diagnosis
- Regressions after recent changes

**Don't use either for:**
- Single-line typo fixes
- Trivial config changes

## Requirements

- Claude Code installed
- Git repository (for code review)
- Project with existing codebase (workflow assumes existing code to learn from)

## Author

Aprende Comigo

## Version

2.0.0
