# Migration Structure

**IMPORTANT**: Read these files before creating migrations:
- `supabase/migrations/` — existing migration files to understand naming and structure
- `drizzle.config.ts` — Drizzle Kit configuration
- `lib/db/schema/` — schema files that are the source of truth for DDL

## Two-File Migration Structure

The project uses a two-file structure in `supabase/migrations/`:

### 1. Drizzle-generated DDL (tables, enums, indexes, FKs)

Auto-generated from `lib/db/schema/` files. Contains `CREATE TABLE`, `ALTER TABLE`, indexes, etc.

This is the standard Drizzle Kit output — never hand-edit these.

### 2. Hand-written custom SQL (functions, triggers, storage)

Contains PostgreSQL functions, triggers, storage bucket definitions, and RLS policies that Drizzle ORM cannot generate.

These are created manually using `npm run db:generate -- --custom`.

Read the existing migration files to see the actual naming convention and what goes in each file.

## Workflow

### For schema changes (tables, columns, indexes):

1. Modify the schema in `lib/db/schema/`
2. Run `npm run db:generate` to generate a new migration
3. Review the generated SQL in `supabase/migrations/`
4. Run `npm run db:migrate` to apply
5. Test with `supabase db reset` on local

### For custom SQL (functions, triggers, storage):

1. Run `npm run db:generate -- --custom` to create an empty migration file
2. Write the SQL manually (functions, triggers, etc.)
3. Run `npm run db:migrate` to apply

### For full local reset:

`supabase db reset` — drops and recreates the database, re-running all migrations.

## Principles

- **Schema files are the source of truth** for DDL. Edit `lib/db/schema/`, then generate.
- **Never edit Drizzle-generated migrations** after they've been applied.
- **Always review generated SQL** before applying — Drizzle may generate unexpected changes.
- **Keep migrations small and focused** — one logical change per migration.
- **Test on local first** — use `supabase db reset` to verify the full migration chain works.
- **Drizzle tracks its own migrations** in `drizzle.__drizzle_migrations` (separate from Supabase's `supabase_migrations.schema_migrations`).
