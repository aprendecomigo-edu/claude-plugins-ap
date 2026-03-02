# Internationalization Patterns

**IMPORTANT**: Read these source files before adding translations:
- `messages/en.json` — existing English translation keys (source of truth for structure)
- `messages/pt.json` — Portuguese translations (must stay in sync with en.json)
- An existing page using `useTranslations` or `getTranslations` — to see the pattern in practice

## Principles

### Translation key organization

Keys are organized by feature namespace with dot notation:

```json
{
  "featureName": {
    "title": "Feature Title",
    "form": {
      "fieldName": "Field Label",
      "submit": "Save",
      "cancel": "Cancel"
    },
    "table": {
      "columnName": "Column Header"
    },
    "messages": {
      "createSuccess": "Record created successfully",
      "deleteConfirm": "Are you sure you want to delete this?"
    },
    "empty": "No records found"
  }
}
```

Read `messages/en.json` to see the actual structure and naming conventions used in this project.

### Key naming rules

- Use dot notation for nesting: `featureName.form.fieldName`
- Group by UI context: `form`, `table`, `messages`, `errors`
- Use descriptive action names: `createSuccess`, `deleteConfirm`
- Keep keys in both `en.json` and `pt.json` in sync
- Never hardcode English text in components — always use `t("key")`

### Parameterized messages

When translation messages need dynamic values, use ICU message format:

```json
{
  "messages": {
    "recordCount": "Showing {count} records",
    "greeting": "Hello, {name}"
  }
}
```

```typescript
t("messages.recordCount", { count: records.length });
```

Read the next-intl docs (via context7) for advanced patterns like pluralization and rich text.

### Locale-specific formatting

Use next-intl formatters for dates, numbers, and currencies — do not format manually:

```typescript
const format = useFormatter();

// Dates — respects locale (en-GB vs pt-PT)
format.dateTime(date, { dateStyle: "medium" });

// Currency — always EUR for this project
format.number(amount, { style: "currency", currency: "EUR" });
```

## Supported Locales

- `en` — English (UK)
- `pt` — Portuguese (PT)

Default locale is configured in the project's i18n settings. Read the next-intl configuration file for details.

## Why this matters

- **i18n violations score 80+ in code review**: Hardcoded English text is a high-confidence issue because it breaks the Portuguese experience.
- **Both locales must stay in sync**: Adding a key to `en.json` without `pt.json` (or vice versa) causes missing translation warnings at runtime.
- **Locale-aware formatting**: Dates, numbers, and currencies look different in en-GB vs pt-PT. Using formatters handles this automatically.
