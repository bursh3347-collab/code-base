# Migration Patterns: Prisma vs Drizzle

> Side-by-side comparison of the two most popular TypeScript ORMs for migration workflows. Helps you choose and switch between them.

**Source**: [Prisma](https://github.com/prisma/prisma) (42k⭐, Apache-2.0) + [Drizzle ORM](https://github.com/drizzle-team/drizzle-orm) (30k⭐, Apache-2.0)

**Why this is good**:
- Both are excellent ORMs but with fundamentally different philosophies
- This comparison is based on real production migrations, not toy examples
- Includes the "gotchas" each ORM doesn't mention in their docs

**What we changed**:
- Created practical comparison based on actual SaaS schema migrations
- Added decision framework for choosing between them
- Included migration from Prisma → Drizzle guide (common direction)

---

## Philosophy Comparison

| Aspect | Prisma | Drizzle |
|--------|--------|---------|
| **Query style** | Custom DSL (`findMany`, `include`) | SQL-like (`select().from().where()`) |
| **Schema** | `.prisma` file (custom format) | TypeScript (`.ts` files) |
| **Migration** | Auto-generated + manual edit | Auto-generated SQL |
| **Runtime** | Query engine binary (~15MB) | Zero runtime overhead |
| **Edge** | ⚠️ Needs Accelerate proxy | ✅ Native support |
| **Type safety** | Generated types | Inferred from schema |
| **Learning curve** | Lower (abstracts SQL) | Higher (you write SQL-ish) |
| **Bundle size** | ~2MB+ (engine) | ~50KB |

## Schema Comparison

### Prisma Schema

```prisma
// prisma/schema.prisma
model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  role      Role     @default(USER)
  posts     Post[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model Post {
  id        String   @id @default(cuid())
  title     String
  content   String?
  published Boolean  @default(false)
  author    User     @relation(fields: [authorId], references: [id])
  authorId  String
  createdAt DateTime @default(now())
}

enum Role {
  USER
  ADMIN
}
```

### Drizzle Schema

```typescript
// lib/db/schema.ts
import { pgTable, text, boolean, timestamp, pgEnum } from 'drizzle-orm/pg-core';
import { relations } from 'drizzle-orm';

export const roleEnum = pgEnum('role', ['user', 'admin']);

export const users = pgTable('users', {
  id: text('id').primaryKey().$defaultFn(() => crypto.randomUUID()),
  email: text('email').unique().notNull(),
  name: text('name'),
  role: roleEnum('role').default('user').notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
});

export const posts = pgTable('posts', {
  id: text('id').primaryKey().$defaultFn(() => crypto.randomUUID()),
  title: text('title').notNull(),
  content: text('content'),
  published: boolean('published').default(false).notNull(),
  authorId: text('author_id').notNull().references(() => users.id),
  createdAt: timestamp('created_at').defaultNow().notNull(),
});

export const usersRelations = relations(users, ({ many }) => ({
  posts: many(posts),
}));

export const postsRelations = relations(posts, ({ one }) => ({
  author: one(users, { fields: [posts.authorId], references: [users.id] }),
}));
```

## Query Comparison

```typescript
// Find user with posts — Prisma
const user = await prisma.user.findUnique({
  where: { id: userId },
  include: { posts: { where: { published: true } } },
});

// Find user with posts — Drizzle
const user = await db.query.users.findFirst({
  where: eq(users.id, userId),
  with: { posts: { where: eq(posts.published, true) } },
});

// Or SQL-style:
const result = await db
  .select()
  .from(users)
  .leftJoin(posts, and(eq(posts.authorId, users.id), eq(posts.published, true)))
  .where(eq(users.id, userId));
```

## Migration Workflow

### Prisma

```bash
# Edit schema.prisma, then:
npx prisma migrate dev --name add_posts_table  # Dev: creates + applies
npx prisma migrate deploy                       # Prod: applies pending
npx prisma db push                               # Dev: sync without migration file
npx prisma generate                              # Regenerate client types
```

### Drizzle

```bash
# Edit schema.ts, then:
npx drizzle-kit generate    # Generate SQL migration file
npx drizzle-kit migrate     # Apply migrations
npx drizzle-kit push        # Dev: sync without migration file
npx drizzle-kit studio      # Visual DB browser
```

## Migration from Prisma to Drizzle

Common direction (for Edge compatibility or bundle size):

```bash
# Step 1: Install Drizzle
npm install drizzle-orm postgres
npm install -D drizzle-kit

# Step 2: Generate Drizzle schema from existing DB
npx drizzle-kit introspect
# This creates schema.ts from your existing database

# Step 3: Verify generated schema matches Prisma schema
# Step 4: Replace Prisma client usage with Drizzle queries
# Step 5: Remove Prisma
npm uninstall prisma @prisma/client
```

---

## Decision Framework

```
┌─ Need Edge Runtime (Vercel Edge, Cloudflare)?
│  YES → Drizzle (native Edge support)
│  NO  ↓
│
├─ Team prefers SQL?
│  YES → Drizzle (SQL-like syntax)
│  NO  ↓
│
├─ Want lowest learning curve?
│  YES → Prisma (more abstracted)
│  NO  ↓
│
├─ Care about bundle size (serverless cold starts)?
│  YES → Drizzle (~50KB vs Prisma ~2MB)
│  NO  → Either works, pick what you like
```

## ⭐ Recommendation

**For new SaaS projects on Vercel: use Drizzle.**

- Edge-native, tiny bundle, SQL-like (you learn actual SQL patterns)
- Vercel's own VP chose Drizzle for next-saas-starter
- Prisma is great but adds complexity for serverless (engine binary, connection pooling via Accelerate)

## Caveats

- Prisma's auto-generated migration names are more descriptive. Drizzle generates numbered SQL files — name them yourself for clarity.
- Drizzle `introspect` may not perfectly map all Prisma features (e.g., `@updatedAt`). Manual review is needed.
- Prisma's `include` for nested relations is more intuitive than Drizzle's `with` (but Drizzle's `leftJoin` is more flexible).
- Both ORMs: never run `push` in production. Always use migration files with version control.
