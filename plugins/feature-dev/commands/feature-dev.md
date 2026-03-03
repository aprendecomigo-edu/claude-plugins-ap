---
description: Guided feature development with codebase understanding and architecture focus
argument-hint: Optional feature description
---

# Feature Development

You are an independent engineer implementing a new feature. Follow a systematic approach: understand the codebase deeply, identify all underspecified details, design elegant architectures, then implement.

## Core Principles

- **Identify whats not said.**: Identify all ambiguities, edge cases, and underspecified behaviors. You can use your subagents to debate ideas.
- **Understand before acting**: Read and comprehend existing code patterns first
- **Read files identified by agents**: When launching agents, ask them to return lists of the most important files to read. After agents complete, read those files to build detailed context before proceeding.
- **Simple and elegant**: Prioritize readable, maintainable, architecturally sound code
- **Use TodoWrite**: Track all progress throughout
- **CRITICAL RULE**: You may ONLY output it when the feature is implement and a PR is open! Do not output false promises to escape the loop, even if you think you're stuck or should exit for other reasons. YOU HAVE TO continue until genuine completion.


---

## Before Starting

Read `CLAUDE.md` (or equivalent project instructions) to understand the project's conventions, architecture, and established patterns. This is the source of truth for how features should be built.
**Git** 
1. Create new feat branch inside a new worktree.
2. Move feat issue in AP project to in progress.


---

## Phase 1: Discovery

**Goal**: Understand what needs to be built

Initial request: $ARGUMENTS

**Actions**:
1. Create todo list with all phases
2. If feature unclear, try to understand from codebase or github:
   - What problem are they solving?
   - What should the feature do?
   - Any constraints or requirements?
3. if feature seems too big, breakdown in smaller components and address each at a time. You may use and manage team of agents (as opposed to subagents) for features that involve multiple parts of the codebase.

---

## Phase 2: Codebase Exploration

**Goal**: Understand relevant existing code and patterns at both high and low levels

**Actions**:
1. Launch 2-3 code-explorer agents in parallel. Each agent should:
   - Trace through the code comprehensively and focus on getting a comprehensive understanding of abstractions, architecture and flow of control
   - Target a different aspect of the codebase (eg. similar features, high level understanding, architectural understanding, user experience, etc)
   - Include a list of 5-10 key files to read

   **Example agent prompts**:
   - "Find features similar to [feature] and trace through their implementation comprehensively"
   - "Map the architecture and abstractions for [feature area], tracing through the code comprehensively"
   - "Analyze the current implementation of [existing feature/area], tracing through the code comprehensively"
   - "Identify UI patterns, testing approaches, or extension points relevant to [feature]"

   **Project-specific exploration prompts** (use when working on a Next.js + Supabase project):
   - "Trace the authorization patterns — read `lib/permissions/withAuth.ts` and `lib/permissions/context.ts`, how are routes and server actions protected?"
   - "Map the data models related to [feature] — read `lib/db/schema/` for Drizzle schemas and `lib/schemas/` for Zod validation schemas"
   - "Analyze how similar CRUD features handle the full stack: page → server action → service → schema → i18n → error handling. Start from an existing page in `app/`"

2. Once the agents return, please read all files identified by agents to build deep understanding
3. Present comprehensive summary of findings and patterns discovered

---

## Phase 3: Clarifying Questions

**Goal**: Fill in gaps and resolve ambiguities if they exist.

**CRITICAL**: This is one of the most important phases. DO NOT SKIP.

**Actions**:
1. Review the codebase findings and original feature request
2. Identify underspecified aspects: edge cases, error handling, integration points, scope boundaries, design preferences, backward compatibility, performance needs
3. Use the agents available to get more insights into ambiguity, related issues documentation or PRs.

**Structured checklist** — consider each area and find your own answers:

- **Permissions**: Who can access this feature? Which roles (admin, teacher, staff)? Are there school-scoped restrictions?
- **Multi-tenancy**: Does this data belong to a specific school? How is cross-tenant isolation enforced?
- **User account types**: Does this involve students? Are there different account types (minor vs adult, self-managed vs parent-managed)?
- **Internationalization**: What user-facing text is needed? Are there locale-specific behaviors (date formats, currency)?
- **Validation**: What are the input constraints? Which fields are required vs optional? What are valid ranges/formats?
- **Error cases**: What happens on failure? What feedback does the user see? Are there retry scenarios?
- **Testing**: What are the critical paths to test? Are there E2E flows that need Playwright coverage?

---

## Phase 4: Architecture Design

**Goal**: Design multiple implementation approaches with different trade-offs

**Actions**:
1. Launch 2-3 code-architect agents in parallel with different focuses: minimal changes (smallest change, maximum reuse), clean architecture (maintainability, elegant abstractions), or pragmatic balance (speed + quality)

   **Important**: Before designing, each architect should check established patterns in the codebase for auth guards, i18n integration, Zod validation, and error handling. The architecture should incorporate these cross-cutting concerns from the start, not as an afterthought. No need to keep backward compatibility, this is a new project that should be efficient with no unused code.

2. Review all approaches and form your opinion on which fits best for this specific task (consider: small fix vs large feature, urgency, complexity, team context)
3. Present to user: brief summary of each approach, trade-offs comparison, **your recommendation with reasoning**, concrete implementation differences
4. Ask 1 code-architect agents what they think and take that into consideration. At the end of the day, you are the expert in this codebase and it is your decision.

---

## Phase 5: Implementation

**Goal**: Build the feature


**Pre-implementation checklist** — verify each item is addressed in the chosen architecture before writing code. Read the actual source files to confirm current API conventions:

- [ ] Auth guards on every server action and API route — read `lib/permissions/withAuth.ts` for current guard functions
- [ ] Zod validation schemas for all user inputs — read `lib/schemas/` for conventions
- [ ] Translation keys for all user-facing strings (no hardcoded English) — read `messages/en.json` for structure
- [ ] Error handling using established response helpers — read `lib/utils/errors.ts` for current API
- [ ] School-scoped queries (school_id filtering) for multi-tenant data
- [ ] Loading and error states for UI components

**Actions**:
1. Read all relevant files identified in previous phases
2. Implement following chosen architecture
3. Follow codebase conventions strictly
4. Write clean, well-documented code
5. Update todos as you progress

---

## Phase 6: Quality Review

**Goal**: Ensure code is simple, DRY, elegant, easy to read, and functionally correct

**Actions**:
1. Launch 3 code-reviewer agents in parallel with these specific focuses:
   - **Simplicity & DRY**: Code quality, duplication, unnecessary complexity, readability
   - **Security & Authorization**: Auth guards present on all actions/routes (read `lib/permissions/withAuth.ts`), multi-tenant isolation, input validation, no data leaks across schools
   - **Project Conventions**: i18n usage (no hardcoded strings), Zod validation on inputs, error handling via response helpers (read `lib/utils/errors.ts`), DaisyUI component usage, Drizzle ORM patterns (validate using supabase subagent.)
2. Consolidate findings and identify highest severity issues that you recommend fixing
3. Address issues before proceeding

---

## Phase 7: Summary and Cleanup

**Goal**: Document what was accomplished and issue PR

**Actions**:
1. Mark all todos complete
2. Summarize:
   - What was built
   - Key decisions made
   - Files modified
   - Suggested next steps
3. **Git** 
   - Commit and push changes
   - Open PR with appropriate description of work done and summary above.
   - Comment on feat issue with work done and anything that might have been left out for the future.
   - Delete worktree.

---
