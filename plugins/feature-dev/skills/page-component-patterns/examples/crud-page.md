# CRUD Page — Structural Shape

This example shows the **structural pattern** for CRUD pages. Do not copy verbatim — read the actual source files for current conventions.

## Before creating any page, read:
- `app/` — existing pages to see route groups, layout patterns, and conventions
- `lib/permissions/withAuth.ts` — current auth guard API for server components
- `lib/contexts/NotificationContext.tsx` — current notification hook API
- `messages/en.json` — existing translation key structure and naming

## Server Page Shape (page.tsx)

```typescript
// -> Read app/ for actual page structure and route group conventions
// -> Read lib/permissions/withAuth.ts for auth guard used in server pages

interface Props {
  params: { schoolId: string; locale: string };
}

export default async function RecordsPage({ params }: Props) {
  const { schoolId } = await params;

  // 1. AUTHORIZE — check the user has access to this school
  //    Server pages can use auth guards directly (they're server-side)
  //    -> Read lib/permissions/withAuth.ts for the actual guard API
  const [ctx, authError] = await requireAuth();
  if (authError) redirect("/signin");
  const roleError = requireSchoolRole(ctx, schoolId, "teacher");
  if (roleError) redirect("/unauthorized");

  // 2. TRANSLATE — get server-side translations
  const t = await getTranslations("records");

  // 3. FETCH DATA — query via Drizzle (never via Supabase client)
  //    Always filter by schoolId for multi-tenant isolation
  //    -> Read lib/db/ for the db client and query patterns
  const records = await someService.getBySchool(schoolId);

  // 4. RENDER — server component renders the layout
  //    Pass data as props to client components for interactivity
  return (
    <div className="flex flex-col gap-6">
      <div className="flex items-center justify-between">
        <h1 className="text-2xl font-bold">{t("title")}</h1>
        {/* Client component for interactive create button/modal */}
        <CreateRecordButton schoolId={schoolId} />
      </div>

      {records.length === 0 ? (
        <div className="text-center py-12 text-base-content/60">
          {t("empty")}
        </div>
      ) : (
        {/* Client component for interactive table (delete, edit, etc.) */}
        <RecordTable records={records} schoolId={schoolId} />
      )}
    </div>
  );
}
```

## Loading Skeleton Shape (loading.tsx)

```typescript
// Skeleton should match the page layout structure
// Use DaisyUI's skeleton class for placeholder elements
export default function Loading() {
  return (
    <div className="flex flex-col gap-6">
      <div className="flex items-center justify-between">
        <div className="skeleton h-8 w-32" />
        <div className="skeleton h-10 w-36" />
      </div>
      {/* Match the data display structure (table, cards, etc.) */}
      <div className="overflow-x-auto">
        <table className="table">
          <thead>
            <tr>
              <th><div className="skeleton h-4 w-24" /></th>
              <th><div className="skeleton h-4 w-32" /></th>
              <th><div className="skeleton h-4 w-20" /></th>
            </tr>
          </thead>
          <tbody>
            {Array.from({ length: 5 }).map((_, i) => (
              <tr key={i}>
                <td><div className="skeleton h-4 w-full" /></td>
                <td><div className="skeleton h-4 w-full" /></td>
                <td><div className="skeleton h-4 w-16" /></td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </div>
  );
}
```

## Client Table Component Shape (_components/record-table.tsx)

```typescript
"use client";

// -> Read lib/contexts/NotificationContext.tsx for actual notification hook
// -> Read messages/en.json for translation key structure

export function RecordTable({
  records,
  schoolId,
}: {
  records: RecordType[];  // -> Read lib/db/schema/ for actual type
  schoolId: string;
}) {
  const t = useTranslations("records");
  // -> Read lib/contexts/NotificationContext.tsx for actual destructured methods
  const { showSuccess, showError } = useNotification();

  async function handleDelete(recordId: string) {
    // Call the server action
    // -> Read app/actions/ for actual action import paths
    const result = await deleteRecord(schoolId, recordId);
    if (result.success) {
      showSuccess(t("messages.deleteSuccess"));
    } else {
      showError(result.error);
    }
  }

  return (
    <div className="overflow-x-auto">
      <table className="table">
        <thead>
          <tr>
            {/* All column headers use translation keys — no hardcoded text */}
            <th>{t("table.name")}</th>
            <th>{t("table.email")}</th>
            <th>{t("table.status")}</th>
            <th>{t("table.actions")}</th>
          </tr>
        </thead>
        <tbody>
          {records.map((record) => (
            <tr key={record.id}>
              <td>{record.name}</td>
              <td>{record.email}</td>
              <td>
                <span className="badge badge-sm">{record.status}</span>
              </td>
              <td className="flex gap-1">
                {/* Icons from lucide-react */}
                <button className="btn btn-ghost btn-xs">
                  <Pencil className="h-4 w-4" />
                </button>
                <button
                  className="btn btn-ghost btn-xs text-error"
                  onClick={() => handleDelete(record.id)}
                >
                  <Trash2 className="h-4 w-4" />
                </button>
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

## Why this pattern matters

- **Server/client split**: Server components handle auth + data fetching (secure, fast). Client components handle interactivity only. Data flows down as props — never expose `db` to client components.
- **Translation keys everywhere**: No hardcoded English text. Every user-facing string goes through `t()` so the app works in both English (UK) and Portuguese (PT).
- **School-scoped from the top**: The `schoolId` from URL params is authorized at the page level and passed down to all components and actions.
- **Loading skeletons match layout**: The skeleton shape mirrors the actual page structure so the transition feels smooth.
