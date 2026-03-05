---
name: playwright-cli
description: >
  Provides playwright-cli knowledge for browser automation tasks including navigation, form filling,
  screenshots, web scraping, session management, network request mocking, and Playwright test generation.
  Use whenever browser automation, UI testing, or web scraping is needed — even if the user doesn't
  mention playwright-cli by name, such as "open this URL and fill the form", "scrape this page",
  "take a screenshot of the app", "record a browser session", or "generate a Playwright test".
---

# playwright-cli

Anthropic's official CLI for browser automation. Automate navigation, form interactions, screenshots, and data extraction through straightforward commands.

## Installation

```bash
npx playwright-cli
```

Or install globally:

```bash
npm install -g playwright-cli
```

## Core Concepts

### Snapshots

After each command, playwright-cli automatically provides a page snapshot showing the current state. Each interactive element gets a reference ID (e.g., `e1`, `e2`, `e3`) that you can use in subsequent commands.

```bash
playwright-cli snapshot
# Output shows: e1 [textbox "Email"], e2 [textbox "Password"], e3 [button "Sign In"]
```

### Element References

Use element references from snapshots to interact with page elements:

```bash
playwright-cli click e3        # click element with ref e3
playwright-cli fill e1 "text"  # fill input with ref e1
```

## Navigation

```bash
# Open URL in browser
playwright-cli open https://example.com

# Navigate to URL in current tab
playwright-cli goto https://example.com

# Go back / forward
playwright-cli go-back
playwright-cli go-forward

# Reload page
playwright-cli reload
```

## Element Interactions

```bash
# Click element
playwright-cli click e3
playwright-cli click e3 --button=right   # right click
playwright-cli click e3 --double         # double click

# Type text (append to existing)
playwright-cli type e1 "hello"

# Fill input (replace existing value)
playwright-cli fill e1 "user@example.com"

# Hover over element
playwright-cli hover e5

# Drag and drop
playwright-cli drag e1 e2

# Select dropdown option
playwright-cli select e4 "Option Value"

# Handle dialogs
playwright-cli dialog-accept
playwright-cli dialog-dismiss
playwright-cli dialog-accept --prompt-text="my input"
```

## Keyboard & Mouse

```bash
# Press keyboard keys
playwright-cli key "Enter"
playwright-cli key "Control+A"
playwright-cli key "Tab"

# Scroll
playwright-cli scroll e1 --delta-y=500   # scroll element
playwright-cli scroll --delta-y=500      # scroll page
```

## Screenshots & Snapshots

```bash
# Take screenshot
playwright-cli screenshot
playwright-cli screenshot output.png
playwright-cli screenshot --full-page

# Get accessibility snapshot (shows element refs)
playwright-cli snapshot
```

## Console & DevTools

```bash
# View console logs
playwright-cli console

# Evaluate JavaScript in page context
playwright-cli eval "() => document.title"
playwright-cli eval "() => window.location.href"

# Run custom Playwright code
playwright-cli run-code "async page => { return await page.title(); }"
```

See [references/running-code.md](references/running-code.md) for advanced `run-code` usage.

## Tabs

```bash
# Open new tab
playwright-cli new-tab https://example.com

# List all tabs
playwright-cli tabs

# Switch to tab by index
playwright-cli switch-tab 1

# Close current tab
playwright-cli close-tab
```

## Storage Management

```bash
# Cookies
playwright-cli cookie-list
playwright-cli cookie-get session_id
playwright-cli cookie-set session abc123 --domain=example.com --httpOnly
playwright-cli cookie-delete session_id
playwright-cli cookie-clear

# localStorage
playwright-cli localstorage-list
playwright-cli localstorage-get token
playwright-cli localstorage-set theme dark
playwright-cli localstorage-delete token
playwright-cli localstorage-clear

# sessionStorage
playwright-cli sessionstorage-list
playwright-cli sessionstorage-get form_data
playwright-cli sessionstorage-set step 3
playwright-cli sessionstorage-clear

# Save/restore full browser state
playwright-cli state-save auth.json
playwright-cli state-load auth.json
```

See [references/storage-state.md](references/storage-state.md) for detailed storage management.

## Network / Request Mocking

```bash
# Block requests by pattern
playwright-cli route "**/*.jpg" --status=404

# Mock with JSON body
playwright-cli route "**/api/users" --body='[{"id":1}]' --content-type=application/json

# List active routes
playwright-cli route-list

# Remove route
playwright-cli unroute "**/*.jpg"
playwright-cli unroute   # remove all
```

See [references/request-mocking.md](references/request-mocking.md) for advanced mocking.

## Session Management

```bash
# Use named browser sessions (isolated cookies, storage, history)
playwright-cli -s=auth open https://app.example.com/login
playwright-cli -s=public open https://example.com

# List all active sessions
playwright-cli list

# Close sessions
playwright-cli close              # close default session
playwright-cli -s=auth close      # close named session
playwright-cli close-all          # close all sessions

# Persistent profile
playwright-cli open https://example.com --persistent
playwright-cli open https://example.com --profile=/path/to/profile
```

See [references/session-management.md](references/session-management.md) for detailed session usage.

## Debugging & Recording

```bash
# Tracing
playwright-cli tracing-start
# ... perform actions ...
playwright-cli tracing-stop

# Video recording
playwright-cli video-start
# ... perform actions ...
playwright-cli video-stop recording.webm
```

See [references/tracing.md](references/tracing.md) and [references/video-recording.md](references/video-recording.md).

## Test Generation

Every action automatically generates corresponding Playwright TypeScript code in the output. Collect this code into test files.

See [references/test-generation.md](references/test-generation.md) for details.

## Common Workflows

### Web Scraping

```bash
playwright-cli open https://example.com/products
playwright-cli snapshot
playwright-cli eval "() => Array.from(document.querySelectorAll('.product-name')).map(el => el.textContent)"
```

### Login and Save Auth State

```bash
playwright-cli open https://app.example.com/login
playwright-cli snapshot
playwright-cli fill e1 "user@example.com"
playwright-cli fill e2 "password123"
playwright-cli click e3
playwright-cli state-save auth.json
```

### Reuse Auth State

```bash
playwright-cli state-load auth.json
playwright-cli open https://app.example.com/dashboard
# Already logged in
```
