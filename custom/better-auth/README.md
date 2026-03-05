# better-auth Plugin

Better Auth integration skills for Claude Code. Covers core setup, sessions, security, hooks, plugins, and organization/multi-tenancy patterns.

## Features

- **`better-auth-practices` skill** — Core integration reference: setup workflow, config options, database adapters, session management, email flows, security settings, hooks API, plugin list, client usage, type safety, and common gotchas
- **`better-auth-organization` skill** — Organization plugin reference: multi-tenant orgs, member management, invitations, teams, dynamic RBAC, and lifecycle hooks

## Installation

Add this plugin to your Claude Code project or global config.

## Prerequisites

None. Skills provide reference knowledge only — no external services or environment variables required.

## Usage

Skills trigger automatically based on context:

- Ask about Better Auth setup, config, sessions, or auth-related code → `better-auth-practices`
- Ask about organization features, RBAC, teams, or member invitations → `better-auth-organization`
- Before modifying `lib/auth.ts`, `lib/auth-client.ts`, or any auth-related file → `better-auth-practices`

For code examples and the latest API, always consult [better-auth.com/docs](https://better-auth.com/docs).
