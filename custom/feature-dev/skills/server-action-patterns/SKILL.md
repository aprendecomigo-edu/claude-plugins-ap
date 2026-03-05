# Server Action Patterns

Third-person description: Guides the developer through creating Next.js server actions with proper authorization, validation, error handling, and service delegation patterns used in the Aprende Comigo codebase. Activates when the developer needs to "create a server action", "add a form action", "implement a mutation", or "add a new action".

---

**IMPORTANT**: Before writing any server action, read the actual source files to verify current API signatures:
- `lib/permissions/withAuth.ts` — auth guard functions and their return types
- `lib/utils/errors.ts` — response helper signatures
- `lib/schemas/` — existing validation schemas and conventions

## Server Action Structure

Every server action follows this consistent 6-step pattern:

1. **`"use server"` directive** at the top of the file
2. **Authenticate** — call `requireAuth()` to get a permission context (tuple return: `[ctx, error]`)
3. **Validate** — parse inputs against a Zod schema from `lib/schemas/`
4. **Authorize** — call `requireSchoolRole(ctx, schoolId, level)` for school-scoped actions
5. **Delegate** — call a service function for business logic (services handle DB queries)
6. **Respond** — return standardized success/error responses via helpers in `lib/utils/errors.ts`

## Authorization Guards

Two guards exist, used in sequence. Read `lib/permissions/withAuth.ts` for their exact signatures:

1. **Authentication guard** — verifies the user is logged in and returns a permission context (tuple return pattern: context-or-error). The context carries the user's `profileId`, school memberships, and role-check methods.
2. **School role guard** — checks whether the user has a required role at a specific school. Role levels are hierarchical: `"admin"` (strictest) > `"staff"` > `"teacher"` > `"member"` (most permissive).

Read `lib/permissions/context.ts` for available methods on the permission context — includes role checks, relationship checks, and school membership queries.

## Input Validation with Zod

- Define schemas in `lib/schemas/` (shared between client forms and server actions)
- Import from the barrel: `import { studentSchema } from "@/lib/schemas"`
- Use `.parse()` for throwing validation or `.safeParse()` for graceful handling
- Read `lib/schemas/common.ts` for shared schemas (UUID, email, phone)

## Error Handling

Read `lib/utils/errors.ts` for current helper signatures. Key patterns:

- **Success responses** — wrap a message string with optional data fields
- **Error responses** — take the actual error object (for logging/Sentry) plus a user-friendly fallback message
- **Zod error formatting** — converts Zod validation issues into a readable string
- Never expose internal error details (stack traces, SQL errors) to the client

## Service Delegation

Server actions should be thin — delegate business logic to service functions:

- Services live in `lib/services/` directory
- Services handle database queries (via Drizzle ORM), transactions, and complex logic
- Always pass `schoolId` to enforce multi-tenant data isolation
- Use `db.transaction(async (tx) => { ... })` for operations modifying multiple tables

## References

- [auth-patterns.md](references/auth-patterns.md) — authorization principles and role hierarchy
- [error-handling.md](references/error-handling.md) — error response principles and patterns

## Examples

- [crud-action.md](examples/crud-action.md) — structural shape of a server action with business logic comments
