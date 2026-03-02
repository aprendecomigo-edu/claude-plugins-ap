# Multi-Tenant Patterns

**IMPORTANT**: Read these source files for current implementation:
- `lib/db/schema/` — see which tables have `school_id` and which are global
- `lib/permissions/withAuth.ts` — how school membership is verified
- `lib/permissions/context.ts` — `UserPermissionContext` and school-related methods

## Principles

### School-scoped data isolation

All tenant-specific data is isolated by `school_id`. This is enforced in application code (TypeScript), NOT via database RLS.

### Schema level

Every tenant-scoped table includes a `school_id` column with a foreign key to the schools table:

```typescript
// -> Read lib/db/schema/ to see how school_id is defined on existing tables
schoolId: uuid("school_id")
  .notNull()
  .references(() => schools.id, { onDelete: "cascade" }),
```

### Query level

Every query for tenant data **must** filter by `schoolId`. A query without `schoolId` is a data leak across tenants:

```typescript
// CORRECT — always filter by school
const records = await db.select().from(table).where(eq(table.schoolId, schoolId));

// WRONG — returns data from ALL schools
const records = await db.select().from(table);
```

### Service level

Services receive `schoolId` as a parameter and pass it to all queries. This creates a clear contract: the caller (server action) is responsible for getting and authorizing the `schoolId`, the service is responsible for using it in every query.

### Action level

Server actions extract `schoolId` from the request (URL params, form data, etc.) and authorize it via `requireSchoolRole()` before passing it to services.

## Global (non-school-scoped) tables

Some tables are NOT school-scoped:

- **`profiles`** — a user can belong to multiple schools
- **System configuration** — shared across all tenants
- **Shared reference data** — subjects, etc.

These tables do NOT have a `school_id` column. Access is controlled by authentication and context-specific checks (e.g., `ctx.isOwnProfile()`).

Read `lib/db/schema/` to identify which tables are school-scoped and which are global.

## Why this matters

Multi-tenant data isolation is the highest-severity issue in code review (confidence 95+). A missing `school_id` filter means one school's users can see another school's data. This is a critical security vulnerability in an educational platform handling student information.
