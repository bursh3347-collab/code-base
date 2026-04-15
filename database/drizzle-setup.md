# Drizzle ORM Quick Start

> Type-safe SQL with zero runtime overhead. Schema definition, client setup, queries, and migrations — all in TypeScript.

**Source**: [Drizzle ORM](https://github.com/drizzle-team/drizzle-orm) (30k⭐, Apache-2.0) + [next-saas-starter](https://github.com/leerob/next-saas-starter) (7k⭐, MIT)

**Why this is good**:
- SQL-like syntax — if you know SQL, you know Drizzle (unlike Prisma's custom query language)
- Zero runtime overhead — compiles to plain SQL, no heavy query engine
- Schema defined in TypeScript = single source of truth for types and migrations
- Edge-compatible with `postgres.js` or Neon serverless driver
- Lee Robinson (Vercel VP) chose Drizzle over Prisma for next-saas-starter

**What we changed**:
- Extracted Drizzle setup from next-saas-starter into standalone module
- Added multi-table schema example (users, teams, subscriptions — SaaS pattern)
- Added connection pooling pattern for serverless (Neon / Supabase)
- Added soft delete and audit timestamp patterns

---

## Schema Definition

```typescript
// lib/db/schema.ts
import {
  pgTable,
  text,
  timestamp,
  integer,
  boolean,
  pgEnum,
} from 'drizzle-orm/pg-core';
import { relations } from 'drizzle-orm';

// Enums
export const roleEnum = pgEnum('role', ['user', 'admin']);
export const planEnum = pgEnum('plan', ['free', 'pro', 'team']);

// Users
export const users = pgTable('users', {
  id: text('id')
    .primaryKey()
    .$defaultFn(() => crypto.randomUUID()),
  name: text('name'),
  email: text('email').unique().notNull(),
  passwordHash: text('password_hash'),
  role: roleEnum('role').default('user').notNull(),
  stripeCustomerId: text('stripe_customer_id'),
  image: text('image'),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
  deletedAt: timestamp('deleted_at'), // Soft delete
});

// Teams
export const teams = pgTable('teams', {
  id: text('id')
    .primaryKey()
    .$defaultFn(() => crypto.randomUUID()),
  name: text('name').notNull(),
  plan: planEnum('plan').default('free').notNull(),
  stripeSubscriptionId: text('stripe_subscription_id'),
  createdAt: timestamp('created_at').defaultNow().notNull(),
});

// Team Members (many-to-many)
export const teamMembers = pgTable('team_members', {
  userId: text('user_id')
    .notNull()
    .references(() => users.id, { onDelete: 'cascade' }),
  teamId: text('team_id')
    .notNull()
    .references(() => teams.id, { onDelete: 'cascade' }),
  role: roleEnum('role').default('user').notNull(),
  joinedAt: timestamp('joined_at').defaultNow().notNull(),
});

// Relations
export const usersRelations = relations(users, ({ many }) => ({
  teamMemberships: many(teamMembers),
}));

export const teamsRelations = relations(teams, ({ many }) => ({
  members: many(teamMembers),
}));

export const teamMembersRelations = relations(teamMembers, ({ one }) => ({
  user: one(users, { fields: [teamMembers.userId], references: [users.id] }),
  team: one(teams, { fields: [teamMembers.teamId], references: [teams.id] }),
}));
```

## Database Client

```typescript
// lib/db/index.ts
import { drizzle } from 'drizzle-orm/postgres-js';
import postgres from 'postgres';
import * as schema from './schema';

const connectionString = process.env.DATABASE_URL!;

// Connection pooling for serverless
const client = postgres(connectionString, {
  max: 10,             // max connections
  idle_timeout: 20,    // close idle connections after 20s
  connect_timeout: 10, // connection timeout
});

export const db = drizzle(client, { schema });

// Type helper for transactions
export type Database = typeof db;
```

## Common Query Patterns

```typescript
// lib/db/queries.ts
import { eq, and, isNull, desc, sql } from 'drizzle-orm';
import { db } from '.';
import { users, teams, teamMembers } from './schema';

// Find user by email (exclude soft-deleted)
export async function findUserByEmail(email: string) {
  const [user] = await db
    .select()
    .from(users)
    .where(and(eq(users.email, email), isNull(users.deletedAt)))
    .limit(1);
  return user ?? null;
}

// Get user's teams with member count
export async function getUserTeams(userId: string) {
  return db
    .select({
      team: teams,
      memberCount: sql<number>`count(${teamMembers.userId})::int`,
    })
    .from(teams)
    .innerJoin(teamMembers, eq(teams.id, teamMembers.teamId))
    .where(eq(teamMembers.userId, userId))
    .groupBy(teams.id)
    .orderBy(desc(teams.createdAt));
}

// Soft delete user
export async function softDeleteUser(userId: string) {
  return db
    .update(users)
    .set({ deletedAt: new Date() })
    .where(eq(users.id, userId));
}

// Transaction example
export async function createTeamWithOwner(teamName: string, userId: string) {
  return db.transaction(async (tx) => {
    const [team] = await tx
      .insert(teams)
      .values({ name: teamName })
      .returning();

    await tx.insert(teamMembers).values({
      teamId: team.id,
      userId,
      role: 'admin',
    });

    return team;
  });
}
```

## Drizzle Config

```typescript
// drizzle.config.ts
import { defineConfig } from 'drizzle-kit';

export default defineConfig({
  schema: './lib/db/schema.ts',
  out: './drizzle',
  dialect: 'postgresql',
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
});
```

## Migration Commands

```bash
# Generate migration from schema changes
npx drizzle-kit generate

# Apply migrations
npx drizzle-kit migrate

# Open Drizzle Studio (visual DB browser)
npx drizzle-kit studio

# Push schema directly (dev only, no migration files)
npx drizzle-kit push
```

---

## Usage

```typescript
import { db } from '@/lib/db';
import { users } from '@/lib/db/schema';
import { eq } from 'drizzle-orm';

// Insert
const [newUser] = await db
  .insert(users)
  .values({ email: 'user@example.com', name: 'John' })
  .returning();

// Select
const allUsers = await db.select().from(users);

// Update
await db.update(users).set({ name: 'Jane' }).where(eq(users.id, 'xxx'));

// Delete
await db.delete(users).where(eq(users.id, 'xxx'));
```

## Dependencies

```bash
npm install drizzle-orm postgres
npm install -D drizzle-kit
```

## Environment Variables

```env
DATABASE_URL=postgresql://user:password@host:5432/dbname
```

## Caveats

- `postgres.js` is the recommended driver for Drizzle + serverless. `pg` (node-postgres) requires connection pooling (PgBouncer) on serverless platforms.
- For Supabase, use the connection pooler URL (`port 6543`) not the direct connection (`port 5432`) in serverless environments.
- Drizzle Studio runs locally and connects to your DB directly — never expose it in production.
- `$defaultFn` runs in JavaScript, not PostgreSQL. For DB-level defaults, use `.default(sql`gen_random_uuid()`)` instead.
- Soft delete requires adding `isNull(deletedAt)` to every query. Consider creating helper functions or views.
