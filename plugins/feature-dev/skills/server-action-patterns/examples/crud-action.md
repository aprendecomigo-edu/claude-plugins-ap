# Server Action Shape

This example shows the **structural pattern** for server actions. Do not copy verbatim — read the actual source files for current API signatures.

## Before writing any action, read:
- `lib/permissions/withAuth.ts` — current auth guard API
- `lib/utils/errors.ts` — current response helper API
- `lib/schemas/` — current validation schema patterns
- An existing action in `app/actions/` — to see the pattern in practice

## Create Action Shape

```typescript
"use server";

// -> Read lib/permissions/withAuth.ts for actual import path and signatures
// -> Read lib/utils/errors.ts for actual response helper signatures
// -> Read lib/schemas/ for actual validation schema patterns

export async function createRecord(input: CreateRecordInput) {
  // 1. AUTHENTICATE — verify the user is logged in
  //    requireAuth() returns a tuple: [UserPermissionContext, null] or [null, error]
  //    The context contains profileId, school memberships, and role-check methods
  const [ctx, authError] = await requireAuth();
  if (authError) return authError;

  // 2. VALIDATE — parse user input against a Zod schema
  //    Schemas live in lib/schemas/ and are shared with client-side forms
  //    Use .parse() to throw on failure, or .safeParse() for graceful handling
  const validated = schema.safeParse(input);
  if (!validated.success) {
    // -> Read lib/utils/errors.ts for formatZodErrors signature
    return { success: false, error: formatZodErrors(validated.error) };
  }

  // 3. AUTHORIZE — check the user has the required role at this school
  //    Role levels are hierarchical: admin > staff > teacher > member
  //    A "teacher" check allows teacher, staff, and admin
  //    requireSchoolRole(ctx, schoolId, level) returns null if OK, error if not
  const roleError = requireSchoolRole(ctx, schoolId, "teacher");
  if (roleError) return roleError;

  try {
    // 4. DELEGATE — call a service function for business logic
    //    Services handle DB queries via Drizzle, transactions, validation rules
    //    ALWAYS pass schoolId to enforce multi-tenant data isolation
    const result = await someService.create(schoolId, validated.data);

    // 5. REVALIDATE — invalidate Next.js cache for affected paths
    revalidatePath("/affected-route");

    // 6. RESPOND — use standardized response helpers
    //    -> Read lib/utils/errors.ts for createSuccessResponse signature
    return createSuccessResponse("Record created", { id: result.id });
  } catch (error) {
    // -> Read lib/utils/errors.ts for createErrorResponse signature
    //    Takes an error object + user-friendly default message
    //    Automatically reports to Sentry in production
    return createErrorResponse(error, "Failed to create record");
  }
}
```

## Update Action Shape

Same 6-step pattern. Key differences:

```typescript
export async function updateRecord(recordId: string, input: UpdateRecordInput) {
  const [ctx, authError] = await requireAuth();
  if (authError) return authError;

  // Validate with the UPDATE schema (typically .partial() of the create schema)
  const validated = updateSchema.safeParse(input);
  if (!validated.success) { /* format and return errors */ }

  // Authorize — same pattern, different role level if needed
  const roleError = requireSchoolRole(ctx, schoolId, "teacher");
  if (roleError) return roleError;

  try {
    const result = await someService.update(schoolId, recordId, validated.data);
    revalidatePath("/affected-route");
    return createSuccessResponse("Record updated");
  } catch (error) {
    return createErrorResponse(error, "Failed to update record");
  }
}
```

## Delete Action Shape

Key difference: typically requires higher permissions (admin).

```typescript
export async function deleteRecord(schoolId: string, recordId: string) {
  const [ctx, authError] = await requireAuth();
  if (authError) return authError;

  // Delete often requires admin-level access
  const roleError = requireSchoolRole(ctx, schoolId, "admin");
  if (roleError) return roleError;

  try {
    await someService.delete(schoolId, recordId);
    revalidatePath("/affected-route");
    return createSuccessResponse("Record deleted");
  } catch (error) {
    return createErrorResponse(error, "Failed to delete record");
  }
}
```

## Why this pattern matters

- **Tuple auth returns**: The `[ctx, error]` pattern avoids exceptions for expected failures (unauthenticated users). The context carries everything needed for downstream checks.
- **School-scoped everything**: Every data operation passes `schoolId` through the entire chain (action -> service -> query) to enforce multi-tenant isolation.
- **Thin actions, thick services**: Actions handle auth + validation + response formatting. Business logic lives in services so it can be reused across multiple actions.
- **Standardized responses**: Consistent `{ success, message/error }` shape lets client components handle results uniformly.
