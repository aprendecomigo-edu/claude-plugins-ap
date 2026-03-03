---
description: Autonomous bug diagnosis and fix workflow - accepts bug descriptions, GitHub issue numbers, or issue URLs
argument-hint: Bug description, GitHub issue number, or issue URL
allowed-tools: Bash(gh issue view:*), Bash(gh issue list:*), Bash(git diff:*), Bash(git log:*), Bash(git status:*), Bash(git blame:*), Bash(npx vitest:*), Bash(npx playwright:*)
---

# Bug Fix

You are autonomously diagnosing and fixing a bug. Work through all phases without waiting for user input. Be thorough in diagnosis, minimal in fix scope, and verify your work with tests.

## Core Principles

- **Autonomous execution**: Complete all phases without user checkpoints. Only pause if the bug is fundamentally ambiguous (multiple equally likely root causes with different fixes).
- **Minimal fix**: Change the least amount of code necessary. Bug fixes should not refactor, improve, or extend beyond what is needed.
- **Verify with evidence**: Use git history, error messages, and tests to confirm diagnosis before fixing.
- **Use TodoWrite**: Track all progress throughout.
- **CRITICAL RULE**: You may ONLY output it when the bug is at least partially fixed and a PR is open! Do not output false promises to escape the loop, even if you think you're stuck or should exit for other reasons. YOU HAVE TO continue until genuine completion.

---

## Before Starting

Read `CLAUDE.md` (or equivalent project instructions) to understand the project's conventions, architecture, and established patterns. This is the source of truth for how fixes should be applied.

**Git** 
1. Create new fix branch inside a new worktree.
---

## Phase 1: Bug Understanding

**Goal**: Parse the input and produce a structured bug summary

Initial request: $ARGUMENTS

**Actions**:
1. Create todo list with all phases
2. Detect the input type and gather bug details:
   - **GitHub issue number** (e.g., `#123`): Run `gh issue view 123` to fetch title, body, labels, and comments
   - **GitHub issue URL** (e.g., `https://github.com/.../issues/123`): Extract the issue number and run `gh issue view`
   - **Free text description**: Use directly as the bug report
3. Produce a structured bug summary:
   - **Symptom**: What is happening?
   - **Expected behavior**: What should happen?
   - **Reproduction steps**: How to trigger the bug (if known)
   - **Error messages**: Any error output, stack traces, or logs mentioned
   - **Affected area**: Which part of the app is impacted (if identifiable)

---

## Phase 2: Bug Localization

**Goal**: Find the code responsible for the bug

**Actions**:
1. Launch 2-3 `code-explorer` agents in parallel, each targeting a different diagnostic angle:
   - "Trace the code path for [affected feature/area], focusing on where [symptom] could originate. Return the 5-10 most suspect files."
   - "Search for error messages, keywords, or patterns matching '[error text / symptom description]'. Check git history for recent changes to related files. Return the 5-10 most relevant files."
   - "Analyze the [specific subsystem — e.g., auth flow, data layer, API route] that handles [affected functionality]. Return the 5-10 most relevant files."

2. Read all suspect files returned by agents to build detailed understanding
3. Run `git log --oneline -20 -- <suspect-files>` to check recent changes
4. Run `git blame` on the most suspect sections to identify when the bug was likely introduced
5. Produce a localization report:
   - **Suspect files**: Ranked by likelihood
   - **Recent changes**: Any commits that may have introduced the bug
   - **Key observations**: Patterns or anomalies found in the code

---

## Phase 3: Root Cause Analysis

**Goal**: Identify and verify the root cause

**Actions**:
1. Form a hypothesis based on Phase 2 findings
2. Verify the hypothesis by reading the specific code paths and checking against the symptom
3. Check common Aprende Comigo bug patterns:
   - **Missing `school_id` filter**: Multi-tenant query returning data from other schools
   - **Missing auth guard**: Server action or API route lacking `requireAuth` / `requireSchoolRole`
   - **Zod schema mismatch**: Validation schema not matching the actual form data or API payload
   - **i18n key issues**: Missing translation key, wrong namespace, or hardcoded string
   - **Drizzle misconfiguration**: Wrong relation, missing join, incorrect column reference
   - **App Router caching**: Stale data from aggressive caching, missing `revalidatePath`
   - **Error handling gap**: Unhandled error case, missing try/catch, wrong error response format
4. Write a clear root cause statement:
   - **Root cause**: What exactly is wrong and why
   - **Evidence**: Specific code references and reasoning
   - **Confidence**: High / Medium / Low

If confidence is Low, state what additional information would be needed and proceed with the most likely hypothesis.

---

## Phase 4: Fix Design

**Goal**: Design the minimal fix

**Actions**:
1. List each file to modify with the specific change needed
2. Assess side effects:
   - Could this change break other features?
   - Are there other call sites that depend on the current (buggy) behavior?
   - Does this fix require changes to tests, translations, or schemas?
3. Flag high-risk areas (shared utilities, auth logic, database schema changes)
4. Proceed immediately to implementation — no user approval needed

---

## Phase 5: Implementation

**Goal**: Apply the fix

**Actions**:
1. Fresh-read all files to modify (do not rely on stale context from earlier phases)
2. Run through the pre-implementation checklist — verify each item is addressed where relevant:
   - [ ] Auth guards present on affected server actions and API routes
   - [ ] Zod validation schemas correct for affected inputs
   - [ ] Translation keys present for any new or changed user-facing strings
   - [ ] School-scoped queries (`school_id` filtering) correct for affected queries
   - [ ] Error handling using established response helpers
3. Apply the fix following codebase conventions strictly
4. Run related tests to verify:
   - `npx vitest run --reporter=verbose <related-test-files>` for unit/integration tests
   - `npx playwright test <related-test-files>` for E2E tests (if applicable)
   - Use playwright mcp to manually test in the browser
5. If tests fail, diagnose and fix — do not leave failing tests
6. Update todos as you progress

---

## Phase 6: Verification & Summary

**Goal**: Review the fix and present results

**Actions**:
1. Launch 2 `code-reviewer` agents in parallel:
   - **Correctness review**: "Review this bug fix for correctness. Verify the root cause analysis is sound, the fix addresses the actual problem, and no new bugs are introduced. Check edge cases."
   - **Security & conventions review**: "Review this bug fix for security issues and project convention adherence. Check auth guards, multi-tenant isolation, input validation, i18n usage, error handling patterns, and Drizzle ORM conventions."

2. Process review findings:
   - **Critical issues** (confidence >= 80%): Fix them immediately without asking
   - **Non-critical suggestions**: ask the code architect agent for opinion and then act on it.

3. If critical issues were fixed, re-run related tests to confirm nothing broke


## Phase 7: Summary and Cleanup
**Goal**: Document what was accomplished and issue PR

   - **Bug**: One-line description of what was wrong
   - **Root cause**: What caused it and why
   - **Fix**: What was changed and how
   - **Files modified**: List with brief description of each change
   - **Tests**: Which tests were run and their results
   - **Risk assessment**: Low / Medium / High — potential for regressions
   - **Follow-ups**: Any non-critical improvements or related issues noticed

 **Git** 
   - Commit and push changes
   - Open PR with appropriate description of work done and summary above.
   - Comment on bug issue with work done.
   - Delete worktree.
