# Form Patterns

**IMPORTANT**: Read these source files before building forms:
- `lib/schemas/` — existing Zod schemas (shared with server actions)
- `lib/contexts/NotificationContext.tsx` — notification hook API
- `components/ui/` — existing form-related components
- An existing form in `app/` — to see the pattern in practice

## Principles

### Form structure

Forms follow a consistent 4-step pattern:

1. **Zod schema** — defines validation rules, lives in `lib/schemas/`, shared between client-side form validation and server-side action validation
2. **Client component** — renders the form with DaisyUI input classes
3. **Submit handler** — validates client-side, then calls the server action
4. **Feedback** — shows success/error via the notification context

### Client-side validation

Validate before calling the server action to give instant feedback. Use the same Zod schema that the server action uses:

```typescript
// -> Read lib/schemas/ for actual schema imports
const result = schema.safeParse(formData);
if (!result.success) {
  // Show first validation error to user
  showError(result.error.issues[0].message);
  return;
}
// Only call server action after client validation passes
const response = await serverAction(result.data);
```

### Server-side re-validation

The server action validates again independently — never trust client-side validation alone. This double-validation is intentional.

### Feedback pattern

After calling a server action, check the result and show feedback:

```typescript
// -> Read lib/contexts/NotificationContext.tsx for actual method names
const { showSuccess, showError } = useNotification();

const result = await someAction(data);
if (result.success) {
  showSuccess(t("messages.saveSuccess"));
} else {
  showError(result.error);
}
```

## DaisyUI Form Classes

Read the DaisyUI v5 docs (via context7) for current form component classes — including form controls, labels, inputs, selects, textareas, error text, and buttons. Read existing forms in `app/` and `components/` for project conventions.

### Confirmation dialogs

Use DaisyUI modal for destructive actions. Read DaisyUI v5 docs for the current modal pattern.

## Why this matters

- **Shared schemas**: Using the same Zod schema on client and server prevents validation drift — the rules are defined once and enforced everywhere.
- **Instant feedback**: Client-side validation gives users immediate error messages without a round-trip.
- **Double validation**: Server-side re-validation protects against malicious or manipulated requests.
