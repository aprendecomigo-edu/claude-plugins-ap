# Page & Component Patterns

Third-person description: Guides the developer through creating Next.js App Router pages, DaisyUI v5 components, next-intl internationalization, form handling with Zod validation, and notification patterns for a multi-tenant educational SaaS. Activates when the developer needs to "add a new page", "create a form", "add a UI component", "build a dashboard", or "create a list view".

---

**IMPORTANT**: Before creating any page or component, read the actual source files:
- `app/` — existing page structure (route groups, layouts, loading/error boundaries)
- `components/ui/` — existing reusable components
- `lib/contexts/NotificationContext.tsx` — notification hook API
- `messages/en.json` and `messages/pt.json` — existing translation key structure

## App Router Page Structure

Pages follow the Next.js App Router conventions with locale-based routing:

```
app/[locale]/(authenticated)/[schoolId]/feature-name/
├── page.tsx          # Main page component (server component by default)
├── loading.tsx       # Loading skeleton
├── error.tsx         # Error boundary
├── layout.tsx        # Optional layout wrapper
└── _components/      # Page-specific client components
```

Read existing pages in `app/` to confirm the actual route group structure and naming conventions.

### Server vs Client Components

- **Server components** (default) — data fetching, authorization checks, initial rendering
- **Client components** (`"use client"`) — interactivity, forms, state, event handlers
- Keep client components small and focused. Fetch data in server components, pass as props.

## DaisyUI v5 Components

Use DaisyUI v5 component classes for consistent UI:

- **Layout**: `card`, `drawer`, `navbar`, `footer`
- **Data display**: `table`, `stat`, `badge`, `avatar`
- **Forms**: `input`, `select`, `textarea`, `checkbox`, `toggle`
- **Actions**: `btn`, `btn-primary`, `btn-ghost`, `btn-error`
- **Feedback**: `alert`, `toast`, `loading`, `skeleton`
- **Navigation**: `tabs`, `breadcrumbs`, `pagination`

Combine with Tailwind utility classes. Do not write custom CSS — compose with existing classes. Read `app/globals.css` for custom utility classes like `glass-container`, `bg-gradient-primary`.

## Internationalization with next-intl

All user-facing text must use translation keys. Read `messages/en.json` for the existing key structure.

### In Server Components

```typescript
import { getTranslations } from "next-intl/server";

export default async function Page() {
  const t = await getTranslations("namespace");
  return <h1>{t("title")}</h1>;
}
```

### In Client Components

```typescript
"use client";
import { useTranslations } from "next-intl";

export function MyComponent() {
  const t = useTranslations("namespace");
  return <label>{t("form.fieldName")}</label>;
}
```

### Translation Files

Add keys to both language files in `messages/`:
- `messages/en.json` — English (UK)
- `messages/pt.json` — Portuguese (PT)

## Form Handling

Forms combine server actions with client-side validation:

1. **Define Zod schema** in `lib/schemas/` — shared between client and server
2. **Client-side validation** — validate on submit before calling the action
3. **Server action** — validates again server-side (never trust client)
4. **Feedback** — use the notification context to show success/error messages

## Notification Pattern

Read `lib/contexts/NotificationContext.tsx` for the actual hook API. The pattern:

```typescript
// -> Read lib/contexts/NotificationContext.tsx for the actual import and method names
const { showSuccess, showError, showWarning, showInfo } = useNotification();

// After a server action completes:
if (result.success) {
  showSuccess(t("messages.createSuccess"));
} else {
  showError(result.error);
}
```

## Responsive Design

- Use Tailwind responsive prefixes: `sm:`, `md:`, `lg:`, `xl:`
- Mobile-first approach — default styles for mobile, add breakpoints for larger screens
- Tables: consider card layout on mobile, table on desktop
- Navigation: drawer on mobile, sidebar on desktop

## Loading & Error States

Every page should handle loading and error states:

- `loading.tsx` — skeleton UI matching the page layout (use DaisyUI `skeleton` class)
- `error.tsx` — user-friendly error message with retry option
- Use `loading loading-spinner` for inline loading indicators

## References

- [i18n-patterns.md](references/i18n-patterns.md) — translation key conventions and locale patterns
- [form-patterns.md](references/form-patterns.md) — form structure and validation principles

## Examples

- [crud-page.md](examples/crud-page.md) — structural shape of a CRUD page with server/client component separation
