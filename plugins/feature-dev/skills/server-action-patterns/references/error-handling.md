# Error Handling Patterns

**IMPORTANT**: Read `lib/utils/errors.ts` for current helper signatures before implementing.

## Principles

### Standardized response shape

All server actions return a consistent shape:
- Success: `{ success: true, message: string, ...optionalData }`
- Error: `{ success: false, error: string }`

This lets client components handle any action result uniformly without knowing the specific action.

### Response helpers

Read `lib/utils/errors.ts` for the actual function signatures. Key helpers:

- **`createSuccessResponse(message, data?)`** — wraps a success message with optional data fields
- **`createSuccessWithWarningsResponse(message, warnings, data?)`** — success with warnings array
- **`createErrorResponse(error, defaultMessage)`** — takes the actual error object (for logging/Sentry) plus a user-friendly fallback message
- **`formatZodErrors(zodError)`** — converts Zod validation issues into a readable string
- **`formatError(error, defaultMessage)`** — extracts a message from any error type

### Validation errors

Handle Zod validation failures gracefully:

```typescript
// Pattern: catch ZodError separately from other errors
try {
  const parsed = schema.parse(input);
  // ... rest of action
} catch (error) {
  if (error instanceof z.ZodError) {
    return { success: false, error: formatZodErrors(error) };
  }
  return createErrorResponse(error, "Operation failed");
}

// Alternative: use safeParse for non-throwing validation
const result = schema.safeParse(input);
if (!result.success) {
  return { success: false, error: formatZodErrors(result.error) };
}
```

### Never expose internals

- Do not return stack traces, SQL errors, or internal error objects to the client
- `createErrorResponse` handles this: it logs the real error server-side and returns only the user-friendly message
- In production, errors are automatically reported to Sentry via `createErrorResponse`
- Return generic messages for unexpected errors ("An unexpected error occurred")

### Error propagation

Services may throw errors that actions catch. The pattern:
1. Service throws a descriptive error (for logging)
2. Action catches it and returns a user-friendly message via `createErrorResponse`
3. Client component reads `result.success` and shows appropriate feedback
