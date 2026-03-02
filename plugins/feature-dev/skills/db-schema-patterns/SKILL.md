# Database Schema Patterns

Third-person description: Guides the developer through creating Drizzle ORM schemas, multi-tenant table design, migration files, relations, and indexes for a Next.js + Supabase project. Activates when the developer needs to "add a database table", "create a schema", "add a migration", "define a new model", or "modify the database".

---

**IMPORTANT**: Before creating any schema or migration, read the actual source files:
- `lib/db/schema/` — existing table definitions (the source of truth for DDL)
- `lib/schemas/` — existing Zod validation schemas
- `drizzle.config.ts` — migration configuration
- `supabase/migrations/` — existing migration files to understand the two-file structure

## Drizzle Schema Design

Schemas are defined in `lib/db/schema/` using Drizzle ORM's TypeScript DSL:

1. **Define the table** with columns, types, defaults, and constraints
2. **Add relations** to connect tables via foreign keys
3. **Add indexes** for frequently queried columns
4. **Export** the table and relations from the schema barrel file
5. **Generate migration** with `npm run db:generate`

Read an existing schema file in `lib/db/schema/` to see the actual patterns (column types, naming conventions, how relations are defined).

## Multi-Tenant Isolation

Every table that stores tenant-specific data must include a `school_id` column:

- Foreign key referencing the schools table with appropriate `onDelete` behavior
- Index on `school_id` for query performance
- Compound index on `(school_id, <primary-lookup-column>)` for common queries
- Authorization enforced in TypeScript via `lib/permissions/` (NOT via RLS)

Not all tables are school-scoped. Read `lib/db/schema/` to identify which tables have `school_id` and which are global (e.g., `profiles`).

## Table Structure Conventions

Read existing tables in `lib/db/schema/` to confirm these conventions:

- Primary keys: UUID with `defaultRandom()`
- Timestamps: `created_at` and `updated_at` columns with defaults
- Foreign keys: explicit `references()` with `onDelete` behavior
- Enums: `pgEnum` for status fields and type discriminators
- Naming: snake_case for database columns, camelCase for TypeScript properties
- Use `pgTable` from Drizzle

## Student Account Types

When working with student data, account for three account types:

- **STUDENT_WITH_ACCOUNT** — has auth.users account + has guardian(s)
- **ADULT_STUDENT** — has auth.users account + no guardians (self-managed, 18+)
- **STUDENT_NO_ACCOUNT** — no auth.users account (user_id IS NULL) + has guardian(s)

These types affect permissions, data visibility, and query patterns. Read `lib/db/schema/` for the students table definition and the database trigger enforcing adult student rules.

## Relations

Define relations separately from tables using `relations()`:

- One-to-many: parent table has `many`, child table has `one`
- Many-to-many: use a junction table with two `one` relations
- Relations enable type-safe eager loading with `db.query.*.findMany({ with: { ... } })`

## Two-File Migration Structure

The project uses a specific two-file migration structure in `supabase/migrations/`:

1. **Drizzle-generated DDL** — tables, enums, indexes, foreign keys (auto-generated from `lib/db/schema/`)
2. **Hand-written custom SQL** — PostgreSQL functions, triggers, storage buckets, RLS policies that Drizzle cannot generate

Read `supabase/migrations/` to see the actual file naming and structure. Use `npm run db:generate` for DDL changes, and `npm run db:generate -- --custom` for hand-written SQL.

## References

- [migration-structure.md](references/migration-structure.md) — migration workflow and two-file structure
- [multi-tenant-patterns.md](references/multi-tenant-patterns.md) — school-scoped data isolation principles

## Examples

- [table-with-relations.md](examples/table-with-relations.md) — structural shape of a table definition with relations and indexes
