# Authorization Patterns

**IMPORTANT**: Read these source files before implementing auth patterns:
- `lib/permissions/withAuth.ts` — `requireAuth()` and `requireSchoolRole()` signatures
- `lib/permissions/context.ts` — `UserPermissionContext` class and its methods

## Principles

### Two-step authorization

Every server action performs two checks in sequence:

1. **Authentication** (`requireAuth()`) — Is this a logged-in user? Returns a permission context with the user's `profileId`, school memberships, and role information.
2. **Authorization** (`requireSchoolRole()`) — Does this user have the required role at this specific school? This is optional for actions that any authenticated user can perform (e.g., updating own profile).

### Tuple return pattern

`requireAuth()` returns a tuple `[context, error]` rather than throwing. This is deliberate:
- Avoids try/catch for expected auth failures
- Makes the "early return on error" pattern clean and readable
- The context is guaranteed non-null when error is null

### Role hierarchy

Roles are school-scoped — a user can be admin at one school and teacher at another. The hierarchy (from most to least restrictive):

- **`"admin"`** — owner and admin roles only
- **`"staff"`** — staff, admin, or owner
- **`"teacher"`** — teacher role OR staff-or-higher
- **`"member"`** — any school member (student, guardian, teacher, staff, admin)

When you check for `"teacher"`, it allows teachers AND anyone higher (staff, admin, owner). This is the hierarchical enforcement.

### Permission context capabilities

The `UserPermissionContext` (from `lib/permissions/context.ts`) provides methods beyond basic role checks:
- `isAdminInSchool(schoolId)`, `isStaffOrHigherInSchool(schoolId)`, etc.
- `isOwnProfile(targetProfileId)` — for self-service actions
- `isGuardianOf(studentId)` — async, for guardian-specific permissions
- `canManageFinancesFor(studentId)` — async, granular guardian permissions

Read the actual file for the full API — these methods evolve as the permission system grows.

### School-scoped data filtering

After authorization, always filter queries by `schoolId`:
- Every tenant-scoped query must include a `school_id` condition
- Never return data from a school the user doesn't belong to
- Pass `schoolId` through the entire chain: action -> service -> query
- RLS is NOT used — isolation is enforced in TypeScript

### Global vs school-scoped tables

Some tables are NOT school-scoped (e.g., `profiles`, system config). These don't need `schoolId` filtering but still need authentication. The permission context methods handle this distinction.
