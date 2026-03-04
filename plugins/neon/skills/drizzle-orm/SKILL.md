---
name: drizzle-orm
description: >
  Type-safe SQL toolkit for TypeScript. Covers schema declaration with pgTable, column types, enums, indexes and constraints, relational queries, migrations with drizzle-kit (generate, migrate, push, pull, custom migrations), Neon database drivers (HTTP, WebSocket, postgres.js, node-postgres), Zod schema generation, and query patterns. Use for any Drizzle ORM questions including schema design, query building, migration workflows, and configuration.
---

# Drizzle ORM

Drizzle is a headless TypeScript ORM with a SQL-first philosophy. Zero dependencies, 31KB, serverless-ready. Two query interfaces: SQL-like (`db.select().from()`) and relational (`db.query.table.findMany()`).

Docs: https://orm.drizzle.team/docs/overview

## Schema Declaration

Define tables with `pgTable` from `drizzle-orm/pg-core`. Export all models so drizzle-kit can process them.

```typescript
import { pgTable, integer, varchar, timestamp, boolean, text, uuid, jsonb, pgEnum } from "drizzle-orm/pg-core";

// Enums
export const statusEnum = pgEnum("status", ["active", "inactive", "suspended"]);

// Tables
export const users = pgTable("users", {
  id: integer().primaryKey().generatedAlwaysAsIdentity(),
  name: varchar({ length: 255 }).notNull(),
  email: varchar({ length: 255 }).notNull().unique(),
  status: statusEnum().default("active").notNull(),
  metadata: jsonb().$type<{ preferences: Record<string, unknown> }>(),
  createdAt: timestamp({ withTimezone: true }).defaultNow().notNull(),
  updatedAt: timestamp({ withTimezone: true }).defaultNow().notNull().$onUpdate(() => new Date()),
});
```

### Column Naming

Use `casing: 'snake_case'` to auto-map camelCase to snake_case:

```typescript
const db = drizzle({ connection: process.env.DATABASE_URL, casing: "snake_case" });
```

### Reusable Column Patterns

```typescript
const timestamps = {
  createdAt: timestamp({ withTimezone: true }).defaultNow().notNull(),
  updatedAt: timestamp({ withTimezone: true }).defaultNow().notNull().$onUpdate(() => new Date()),
};

export const students = pgTable("students", {
  id: integer().primaryKey().generatedAlwaysAsIdentity(),
  schoolId: integer("school_id").notNull(),
  ...timestamps,
});
```

### Indexes, Constraints, and Foreign Keys

Define in the third argument to `pgTable`:

```typescript
import { pgTable, integer, varchar, text, index, uniqueIndex, unique, check, primaryKey, foreignKey, AnyPgColumn } from "drizzle-orm/pg-core";
import { sql } from "drizzle-orm";

export const posts = pgTable("posts", {
  id: integer().primaryKey().generatedAlwaysAsIdentity(),
  slug: varchar({ length: 256 }).notNull(),
  authorId: integer("author_id").references(() => users.id),
  categoryId: integer("category_id"),
}, (table) => [
  uniqueIndex("posts_slug_idx").on(table.slug),
  index("posts_author_idx").on(table.authorId),
  check("slug_not_empty", sql`${table.slug} <> ''`),
]);

// Composite primary keys (junction tables)
export const booksToAuthors = pgTable("books_to_authors", {
  authorId: integer("author_id").notNull(),
  bookId: integer("book_id").notNull(),
}, (table) => [
  primaryKey({ columns: [table.bookId, table.authorId] }),
]);

// Composite unique constraints
export const enrollment = pgTable("enrollment", {
  studentId: integer("student_id").notNull(),
  courseId: integer("course_id").notNull(),
}, (table) => [
  unique().on(table.studentId, table.courseId),
]);

// Self-referencing foreign keys
export const categories = pgTable("categories", {
  id: integer().primaryKey().generatedAlwaysAsIdentity(),
  parentId: integer("parent_id").references((): AnyPgColumn => categories.id),
});
```

### Schema Organization

Multi-file approach — configure with folder path or glob:

```
lib/db/schema/
├── users.ts
├── students.ts
├── schools.ts
└── index.ts        # re-exports all tables and relations
```

## Relations

Define separately from tables. Enable `db.query` relational API but don't create database foreign keys — use `.references()` for that.

```typescript
import { relations } from "drizzle-orm";

export const usersRelations = relations(users, ({ many }) => ({
  posts: many(posts),
}));

export const postsRelations = relations(posts, ({ one, many }) => ({
  author: one(users, { fields: [posts.authorId], references: [users.id] }),
  comments: many(comments),
}));
```

## Query Patterns

### SQL-Like API

```typescript
import { eq, and, gt, like, inArray } from "drizzle-orm";

const result = await db.select().from(users).where(eq(users.id, 1));

const data = await db.select()
  .from(posts)
  .innerJoin(users, eq(posts.authorId, users.id))
  .where(eq(users.schoolId, schoolId));

await db.insert(users).values({ name: "Ana", email: "ana@example.com" });
await db.update(users).set({ name: "Updated" }).where(eq(users.id, 1));
await db.delete(users).where(eq(users.id, 1));
```

### Relational Query API

Single SQL query under the hood — ideal for serverless where roundtrips are expensive:

```typescript
const usersWithPosts = await db.query.users.findMany({
  with: { posts: { with: { comments: true } } },
  where: (users, { eq }) => eq(users.schoolId, schoolId),
  columns: { id: true, name: true },
  orderBy: (users, { asc }) => [asc(users.name)],
  limit: 10,
});

const user = await db.query.users.findFirst({
  where: (users, { eq }) => eq(users.email, email),
  extras: { fullName: sql`concat(${users.firstName}, ' ', ${users.lastName})`.as("full_name") },
});
```

### Transactions and Prepared Statements

```typescript
await db.transaction(async (tx) => {
  const [user] = await tx.insert(users).values({ name: "Ana" }).returning();
  await tx.insert(profiles).values({ userId: user.id });
});

import { placeholder } from "drizzle-orm";
const prepared = db.query.users.findMany({
  where: (users, { eq }) => eq(users.schoolId, placeholder("schoolId")),
}).prepare("users_by_school");
const result = await prepared.execute({ schoolId: 42 });
```

## Neon Connection Drivers

Install: `npm i drizzle-orm @neondatabase/serverless`

### Neon HTTP (serverless, single queries — recommended)

Best for Next.js server components, API routes, edge functions:

```typescript
import { drizzle } from "drizzle-orm/neon-http";
import * as schema from "./schema";
export const db = drizzle(process.env.DATABASE_URL!, { schema, casing: "snake_case" });
```

Use the **pooled** connection string (`-pooler` endpoint) for serverless.

### Neon WebSocket (interactive transactions)

For multi-statement transactions and session support:

```typescript
import { drizzle } from "drizzle-orm/neon-serverless";
export const db = drizzle(process.env.DATABASE_URL!, { schema, casing: "snake_case" });
```

In Node.js, WebSocket isn't available globally — install `ws` and `bufferutil`:

```typescript
import { Pool, neonConfig } from "@neondatabase/serverless";
import ws from "ws";
neonConfig.webSocketConstructor = ws;
const pool = new Pool({ connectionString: process.env.DATABASE_URL });
const db = drizzle({ client: pool });
```

### postgres.js (full Node.js)

```typescript
import { drizzle } from "drizzle-orm/postgres-js";
import postgres from "postgres";
const queryClient = postgres(process.env.DATABASE_URL!);
const db = drizzle({ client: queryClient });
```

### node-postgres (full Node.js with pg)

```typescript
import { drizzle } from "drizzle-orm/node-postgres";
import { Pool } from "pg";
const pool = new Pool({ connectionString: process.env.DATABASE_URL });
const db = drizzle({ client: pool });
```

| Driver | Best For | Package |
|--------|----------|---------|
| `neon-http` | Serverless, edge, single queries | `@neondatabase/serverless` |
| `neon-serverless` | Serverless with transactions | `@neondatabase/serverless` |
| `postgres-js` | Traditional Node.js | `postgres` |
| `node-postgres` | Node.js with Pool | `pg` |

## Drizzle Kit (Migrations)

### Configuration

```typescript
// drizzle.config.ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  dialect: "postgresql",
  schema: "./lib/db/schema",
  out: "./drizzle",                          // Migration output folder
  dbCredentials: {
    url: process.env.DATABASE_URL!,          // Use direct (non-pooled) URL
  },
  casing: "snake_case",
  verbose: true,                             // Print SQL during generate/pull
  strict: true,                              // Confirm before push operations
  entities: {
    roles: { provider: "neon" },             // Exclude Neon system roles
  },
});
```

Additional config options: `tablesFilter` (glob filter for tables), `schemaFilter` (Postgres schemas to manage, default `["public"]`), `extensionsFilters` (e.g. `["postgis"]`), `breakpoints` (statement breakpoints in SQL files), `migrations.table` (custom migration table name).

Multi-environment: `npx drizzle-kit push --config=drizzle-prod.config.ts`

### Commands

| Command | Purpose |
|---------|---------|
| `drizzle-kit generate` | Generate SQL migration files from schema diff |
| `drizzle-kit migrate` | Apply pending migrations sequentially |
| `drizzle-kit push` | Push schema directly — dev only, no files |
| `drizzle-kit pull` | Introspect database into Drizzle schema |
| `drizzle-kit studio` | Launch web UI for browsing data |
| `drizzle-kit check` | Validate migrations for race conditions |
| `drizzle-kit up` | Upgrade migration snapshots to newer format |

### Migration Workflows

**Production** (generate + migrate): Creates reviewable SQL files with snapshots. Migration file structure:

```
drizzle/
├── 0000_initial/
│   ├── migration.sql      # DDL statements
│   └── snapshot.json      # Schema state snapshot
├── 0001_add_notifications/
│   ├── migration.sql
│   └── snapshot.json
└── meta/
    └── _journal.json      # Migration history
```

`generate` compares current schema against last snapshot, prompts for column renames vs drop+add. `migrate` checks the `__drizzle_migrations` table, runs unapplied files in order.

**Development** (push): Pushes schema directly without migration files. Faster iteration.

**Database-first** (pull): Introspects existing database and generates Drizzle schema files.

### Custom Migrations

For data migrations, seeding, or unsupported DDL:

```bash
npx drizzle-kit generate --custom --name=seed-users
```

Creates an empty SQL file to fill with custom statements. Runs via `drizzle-kit migrate` like any other migration.

Use the **direct (non-pooled)** connection string for drizzle-kit — PgBouncer's transaction mode doesn't support DDL.

## Zod Schema Generation

As of `drizzle-orm@0.38.0`, Zod integration is **first-class** in Drizzle ORM itself. The `drizzle-zod` package is deprecated.

```typescript
import { createInsertSchema, createSelectSchema, createUpdateSchema } from "drizzle-orm/zod";

const userSelectSchema = createSelectSchema(users);
const userInsertSchema = createInsertSchema(users);
const userUpdateSchema = createUpdateSchema(users);  // All fields optional

// With refinements (callback extends, Zod schema replaces)
const createStudentSchema = createInsertSchema(students, {
  name: (schema) => schema.min(1).max(255),    // Extends generated schema
  email: z.string().email(),                     // Replaces generated schema
});

// Factory with type coercion
import { createSchemaFactory } from "drizzle-orm/zod";
const { createInsertSchema } = createSchemaFactory({ coerce: { date: true } });
```

Generated schemas auto-exclude `generatedAlwaysAsIdentity` columns from insert/update. Supports views, enums, and custom `$type` columns.
