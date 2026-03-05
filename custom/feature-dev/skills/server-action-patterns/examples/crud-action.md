# Server Action Shape

This example shows the **structural pattern** for server actions. Do not copy verbatim — read the actual source files for current API signatures.

## Before writing any action, read:
- `lib/permissions/withAuth.ts` — current auth guard API and return types
- `lib/utils/errors.ts` — current response helper API
- `lib/schemas/` — current validation schema conventions
- An existing action in `app/actions/` — to see the pattern in practice

## Create Action Shape

```
"use server"

// 1. AUTHENTICATE — call the auth guard
//    Returns a tuple: permission-context or error
//    The context carries profileId, school memberships, role-check methods
//    -> Read lib/permissions/withAuth.ts for the current guard function and return type

// 2. VALIDATE — parse user input against a Zod schema
//    Schemas live in lib/schemas/ and are shared with client-side forms
//    Use .parse() to throw on failure, or .safeParse() for graceful handling
//    -> Read lib/schemas/ for existing schemas and naming conventions

// 3. AUTHORIZE — check the user has the required role at this school
//    Role levels are hierarchical: admin > staff > teacher > member
//    A "teacher" check allows teacher, staff, admin, and owner
//    -> Read lib/permissions/withAuth.ts for the role guard function

// 4. DELEGATE — call a service function for business logic
//    Services handle DB queries via Drizzle, transactions, and validation rules
//    ALWAYS pass schoolId to enforce multi-tenant data isolation
//    -> Read lib/services/ for existing service patterns

// 5. REVALIDATE — invalidate Next.js cache for affected paths

// 6. RESPOND — return standardized success/error response
//    -> Read lib/utils/errors.ts for response helpers and Sentry integration
```

## Update Action Shape

Same 6-step pattern. Key differences:
- Validate with the UPDATE schema (typically a `.partial()` of the create schema)
- May require a different role level
- Service function receives the record ID plus the validated partial data

## Delete Action Shape

Same 6-step pattern. Key differences:
- Typically requires higher permissions (e.g., admin-level access)
- No input validation needed beyond the record ID
- Service function handles soft-delete vs hard-delete logic

## Why this pattern matters

- **Tuple auth returns**: The context-or-error pattern avoids exceptions for expected failures (unauthenticated users). The context carries everything needed for downstream checks.
- **School-scoped everything**: Every data operation passes `schoolId` through the entire chain (action -> service -> query) to enforce multi-tenant isolation.
- **Thin actions, thick services**: Actions handle auth + validation + response formatting. Business logic lives in services so it can be reused across multiple actions.
- **Standardized responses**: Consistent success/error shape lets client components handle results uniformly.
