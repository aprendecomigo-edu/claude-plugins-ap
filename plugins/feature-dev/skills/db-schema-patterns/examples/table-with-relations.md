# Table with Relations — Structural Shape

This example shows the **structural pattern** for defining tables. Do not copy verbatim — read the actual schema files for current conventions.

## Before creating any schema, read:
- `lib/db/schema/` — existing table definitions for naming/typing conventions
- `lib/schemas/` — existing Zod schemas that mirror DB tables
- `drizzle.config.ts` — how Drizzle is configured

## Schema Definition Shape

```typescript
// -> Read lib/db/schema/ for actual imports and column types used in this project
import { pgTable, uuid, varchar, timestamp, index, uniqueIndex } from "drizzle-orm/pg-core";
import { relations } from "drizzle-orm";

// Table definition
// - school_id enforces multi-tenant isolation (every query filters by this)
// - UUID primary keys with defaultRandom()
// - Timestamps for audit trail
// - Indexes on FK columns and common query patterns
export const records = pgTable(
  "records",
  {
    id: uuid("id").primaryKey().defaultRandom(),

    // MULTI-TENANCY: school_id is required for all tenant-scoped tables
    // onDelete: "cascade" means deleting a school removes its records
    schoolId: uuid("school_id")
      .notNull()
      .references(() => schools.id, { onDelete: "cascade" }),

    // Foreign keys to related tables
    // onDelete: "restrict" prevents deletion if records reference this
    relatedId: uuid("related_id")
      .notNull()
      .references(() => relatedTable.id, { onDelete: "restrict" }),

    // Business fields — types and constraints depend on domain
    name: varchar("name", { length: 255 }).notNull(),

    // Timestamps — standard across all tables
    createdAt: timestamp("created_at").defaultNow().notNull(),
    updatedAt: timestamp("updated_at").defaultNow().notNull(),
  },
  // Index function — add indexes for query performance
  (table) => [
    // Always index school_id for multi-tenant queries
    index("records_school_id_idx").on(table.schoolId),
    // Index foreign keys used in JOINs
    index("records_related_id_idx").on(table.relatedId),
    // Unique compound index for business uniqueness rules
    uniqueIndex("records_school_name_idx").on(table.schoolId, table.name),
  ]
);

// Relations — defined separately from the table
// Enable type-safe eager loading: db.query.records.findMany({ with: { school: true } })
export const recordsRelations = relations(records, ({ one, many }) => ({
  school: one(schools, {
    fields: [records.schoolId],
    references: [schools.id],
  }),
  related: one(relatedTable, {
    fields: [records.relatedId],
    references: [relatedTable.id],
  }),
  // one-to-many: this table has many child records
  children: many(childTable),
}));
```

## Companion Zod Schema Shape

```typescript
// -> Read lib/schemas/ for actual schema conventions and shared helpers
// Schemas live in lib/schemas/ and are shared between client forms and server actions

export const createRecordSchema = z.object({
  name: z.string().min(1, "Name is required").max(255),
  relatedId: z.string().uuid("Invalid ID"),
  // ... other fields matching the table columns
});

// Update schema is typically partial — only changed fields are required
export const updateRecordSchema = createRecordSchema.partial();
```

## Query Pattern Shape

```typescript
// -> Read lib/db/index.ts for the db client
// ALWAYS filter by schoolId for tenant-scoped data

// Eager load with relations
const record = await db.query.records.findFirst({
  where: and(
    eq(records.schoolId, schoolId),  // Multi-tenant isolation
    eq(records.id, recordId),
  ),
  with: {
    related: true,        // Load the related record
    children: true,       // Load child records
  },
});
```

## Why this pattern matters

- **school_id on every tenant table**: Enforces data isolation at the schema level. A query that forgets `school_id` is a data leak.
- **Separate relations from tables**: Drizzle's design keeps the table DDL clean and relations are only used for type-safe eager loading (not for FK constraints — those are in the table definition).
- **Zod schemas mirror DB tables**: Having validation schemas that match the table structure ensures client-side forms and server-side actions validate the same constraints.
