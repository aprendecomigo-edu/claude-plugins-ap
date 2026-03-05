# CRUD Page — Structural Shape

This example shows the **structural pattern** for CRUD pages. Do not copy verbatim — read the actual source files for current conventions.

## Before creating any page, read:
- An existing page in `app/(dashboard)/` — for the current page auth pattern and route structure
- An existing server action in `app/actions/` — to see how actions handle auth (may differ from pages)
- An existing `loading.tsx` in `app/` — for the current skeleton pattern and classes
- `lib/contexts/NotificationContext.tsx` — current notification hook API
- `messages/en.json` — existing translation key structure and naming

## Server Page Shape (page.tsx)

```
// -> Read app/ for actual page structure and route group conventions

// 1. AUTHORIZE — check the user has access to this school
//    Server pages can use auth guards directly (they're server-side)
//    NOTE: Pages may use a different auth pattern than server actions
//    -> Read an existing page in app/(dashboard)/ for the current pattern
//    -> Read lib/permissions/withAuth.ts for the auth guard API
//    On auth failure: redirect to sign-in
//    On role failure: redirect to unauthorized

// 2. TRANSLATE — get server-side translations
//    Use getTranslations("namespace") from next-intl/server
//    -> Read messages/en.json for the namespace and key conventions

// 3. FETCH DATA — query via Drizzle ORM
//    Always filter by schoolId for multi-tenant isolation
//    -> Read lib/services/ for existing service query functions

// 4. RENDER — server component renders the layout
//    Pass data as props to client components for interactivity
//    Handle the empty state with a translated message
//    Use page-specific client components in a _components/ subdirectory
```

## Loading Skeleton Shape (loading.tsx)

```
// Skeleton should mirror the actual page layout structure
// so the transition from loading to loaded feels smooth

// -> Read an existing loading.tsx in app/ for current skeleton classes
// -> Read DaisyUI v5 docs for skeleton component classes

// Match the page structure:
// - Header area with title placeholder + action button placeholder
// - Data display area (table rows, card grid, etc.) with placeholder elements
// - Use Array.from({ length: N }) to generate repeated skeleton rows
```

## Client Component Shape (_components/record-table.tsx)

```
"use client"

// -> Read lib/contexts/NotificationContext.tsx for notification hook
// -> Read messages/en.json for translation key structure

// 1. TRANSLATE — use useTranslations("namespace") from next-intl (client)

// 2. NOTIFICATIONS — destructure methods from the notification hook

// 3. HANDLERS — async functions that call server actions
//    Check result.success, show feedback via notification hook
//    Use translation keys for success messages
//    -> Read app/actions/ for actual action import paths

// 4. RENDER — interactive table/list/cards
//    All column headers and labels use translation keys
//    Use DaisyUI component classes for consistent UI
//    -> Read DaisyUI v5 docs for current table, badge, button classes
//    -> Read existing components in components/ for project conventions
//    Icons come from lucide-react
```

## Why this pattern matters

- **Server/client split**: Server components handle auth + data fetching (secure, fast). Client components handle interactivity only. Data flows down as props — never expose `db` to client components.
- **Translation keys everywhere**: No hardcoded English text. Every user-facing string goes through `t()` so the app works in both English (UK) and Portuguese (PT).
- **School-scoped from the top**: The `schoolId` from URL params is authorized at the page level and passed down to all components and actions.
- **Loading skeletons match layout**: The skeleton shape mirrors the actual page structure so the transition feels smooth.
