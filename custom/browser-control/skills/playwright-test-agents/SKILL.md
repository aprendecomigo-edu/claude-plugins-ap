---
description: >
  Provides knowledge of Playwright's test agent workflow for automated test generation: plan tests, generate test code,
  and heal failing tests using three chained agents. Use when the user asks to "plan tests", "generate Playwright tests",
  "create a test plan", "heal failing tests", "fix broken tests", "generate E2E tests", "create test coverage",
  "set up Playwright test agents", or "auto-generate tests for my app".
---

# Playwright Test Agents

## Overview

Playwright provides three specialized agents that chain together to automate test development:

1. **Planner** — explores the app and produces a markdown test plan
2. **Generator** — transforms plans into executable Playwright test files
3. **Healer** — runs tests and automatically repairs failures

These agents use the `playwright-test` MCP server (`npx playwright run-test-mcp-server`) for browser interaction and test lifecycle management.

## Project Structure

```
project/
├── specs/               # Human-readable test plans (markdown)
│   └── feature-name.md
├── tests/               # Generated Playwright tests
│   ├── seed.spec.ts     # Environment bootstrap (fixtures, setup)
│   └── feature/
│       └── scenario.spec.ts
└── playwright.config.ts
```

## Seed Tests

Seed tests bootstrap the test environment — they set up fixtures, authentication, navigation, and any shared state. Every test plan references a seed file that the agents use to initialize the page context.

```typescript
import { test, expect } from '@playwright/test';

test.describe('Test group', () => {
  test('seed', async ({ page }) => {
    // Setup: navigate, authenticate, initialize state
  });
});
```

The planner runs the seed test first via `planner_setup_page` to understand the app's starting state before exploring.

## Specs (Test Plans)

Specs are structured markdown documents saved to `specs/`. Each spec describes test scenarios with:

- Scenario title
- Seed file reference
- Step-by-step instructions
- Expected outcomes

Example structure:

```markdown
### 1. User Authentication
**Seed:** `tests/seed.spec.ts`

#### 1.1 Login with Valid Credentials
**Steps:**
1. Navigate to login page
2. Enter valid email in email field
3. Enter valid password in password field
4. Click Sign In button
**Expected:** User is redirected to dashboard
```

## Agent Workflow

### Chained Pipeline: Plan → Generate → Heal

```
User request
    ↓
[Planner] → explores app via browser → saves spec to specs/
    ↓
[Generator] → reads spec → executes steps live → writes test files
    ↓
[Healer] → runs tests → debugs failures → patches test code → re-runs
```

### Using Individual Agents

**Plan tests:**
Dispatch the `playwright-test-planner` agent with a description of what to test. It will explore the app and save a test plan to `specs/`.

**Generate tests from a plan:**
Dispatch the `playwright-test-generator` agent with the spec file path. It reads the plan, executes each step live against the app, captures the generated code, and writes test files.

**Fix failing tests:**
Dispatch the `playwright-test-healer` agent. It runs the test suite, identifies failures, debugs them interactively, patches the test code, and re-runs until passing (or marks as `test.fixme()` if the app itself is broken).

## Prerequisites

- Playwright installed in the project (`npm install -D @playwright/test`)
- `playwright.config.ts` configured with at least one project
- The `playwright-test` MCP server (provided by this plugin via `.mcp.json`)

## When to Regenerate Agents

Agent definitions should be regenerated when Playwright is updated to access new tools and instructions:

```bash
npx playwright init-agents --loop=claude
```

## Integration with playwright-cli Skill

The test agents and the `playwright-cli` skill serve different purposes:

- **playwright-cli** — interactive browser automation (scraping, form filling, screenshots, manual testing)
- **Test agents** — automated test lifecycle (plan → generate → heal)

Both can coexist. Use playwright-cli for ad-hoc browser tasks. Use the test agents when building systematic test coverage.
