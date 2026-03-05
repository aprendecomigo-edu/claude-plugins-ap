# browser-control

Browser automation plugin for Claude Code, providing the `playwright-cli` skill for controlling browsers via Anthropic's official Playwright CLI.

## Overview

This plugin equips Claude with knowledge of `playwright-cli`, Anthropic's command-line interface for browser automation. Use it to automate web interactions, scrape pages, fill forms, take screenshots, mock network requests, and generate Playwright test code.

## Features

- **playwright-cli skill** — Comprehensive reference covering all CLI commands, workflows, and advanced patterns
- Navigation, clicking, typing, form filling, drag-and-drop, keyboard/mouse events
- Session management with isolated browser contexts (`-s` flag)
- Storage management: cookies, localStorage, sessionStorage, IndexedDB, state save/load
- Network request mocking and interception
- Multi-tab support
- Tracing and video recording for debugging
- Automatic Playwright TypeScript test code generation

## Installation

Add to your Claude Code project:

```bash
claude plugin add browser-control
```

## Prerequisites

Install `playwright-cli`:

```bash
npm install -g playwright-cli
```

## Usage

Once installed, Claude will automatically apply the `playwright-cli` skill when you ask it to:

- Automate browser interactions
- Scrape web pages
- Fill out forms in a browser
- Take screenshots
- Mock network requests
- Generate Playwright tests
- Manage browser sessions

### Example prompts

- "Open example.com and take a screenshot"
- "Log into app.example.com and save the auth state"
- "Mock the /api/users endpoint to return test data"
- "Record a video of the checkout flow"

## Skills

### playwright-cli

Full reference for all `playwright-cli` commands with references covering:

- [`references/request-mocking.md`](skills/playwright-cli/references/request-mocking.md) — Network interception and mocking
- [`references/running-code.md`](skills/playwright-cli/references/running-code.md) — Custom Playwright code via `run-code`
- [`references/session-management.md`](skills/playwright-cli/references/session-management.md) — Isolated browser sessions
- [`references/storage-state.md`](skills/playwright-cli/references/storage-state.md) — Cookies, localStorage, sessionStorage
- [`references/test-generation.md`](skills/playwright-cli/references/test-generation.md) — Auto-generating Playwright tests
- [`references/tracing.md`](skills/playwright-cli/references/tracing.md) — Execution tracing for debugging
- [`references/video-recording.md`](skills/playwright-cli/references/video-recording.md) — Video capture of browser sessions

## Attribution

The `playwright-cli` skill is adapted from [microsoft/playwright-cli](https://github.com/microsoft/playwright-cli/tree/main/skills/playwright-cli).

## License

Apache 2.0 — see [LICENSE](LICENSE).
