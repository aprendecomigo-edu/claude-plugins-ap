# Error Handling Patterns

**IMPORTANT**: Read `lib/utils/errors.ts` for current helper signatures before implementing.

## Principles

### Standardized response shape

All server actions return a consistent shape:
- Success: `{ success: true, message: string, ...optionalData }`
- Error: `{ success: false, error: string }`

This lets client components handle any action result uniformly without knowing the specific action.

### Response helpers

Read `lib/utils/errors.ts` for the current helper functions. The module provides:

- **Success responses** — wrap a message with optional data fields
- **Success with warnings** — success response that also carries a warnings array
- **Error responses** — take the actual error (for logging/Sentry) plus a user-friendly fallback message
- **Zod error formatting** — converts validation issues into a readable string
- **Generic error formatting** — extracts a message from any error type

### Validation errors

Handle Zod validation failures gracefully. Two approaches:

1. **Throwing** — use `.parse()` and catch `ZodError` separately from other errors, formatting it with the Zod error helper
2. **Non-throwing** — use `.safeParse()`, check `result.success`, and format errors from `result.error`

Read `lib/utils/errors.ts` for the Zod error formatting function. Read existing server actions in `app/actions/` to see which approach the project prefers.

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
