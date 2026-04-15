# REST API Design Patterns

> Unified response format, Zod request validation, cursor-based pagination, and error handling for Next.js App Router API routes.

**Source**: Patterns extracted from [NestJS](https://github.com/nestjs/nest) (70k⭐, MIT), [Hono](https://github.com/honojs/hono) (30k⭐, MIT), and [next-saas-starter](https://github.com/leerob/next-saas-starter) by Lee Robinson

**Why this is good**:
- Consistent API shape makes frontend integration predictable — always `{ data, error, meta }`
- Zod validation catches bad input at the boundary before it hits your database
- Cursor pagination is more performant and consistent than offset pagination for real-time data
- Type-safe error handling prevents leaking internal errors to clients

**What we changed**:
- Adapted NestJS interceptor pattern into Next.js App Router middleware-style helpers
- Added Zod integration for request body + query param validation
- Converted offset pagination to cursor-based with Drizzle ORM
- All patterns are standalone functions, no class decorators needed

---

## Unified API Response

```typescript
// lib/api/response.ts
import { NextResponse } from 'next/server';

interface ApiSuccess<T> {
  data: T;
  error: null;
  meta?: Record<string, unknown>;
}

interface ApiError {
  data: null;
  error: { code: string; message: string; details?: unknown };
  meta?: never;
}

type ApiResponse<T> = ApiSuccess<T> | ApiError;

export function ok<T>(data: T, meta?: Record<string, unknown>) {
  return NextResponse.json<ApiResponse<T>>(
    { data, error: null, ...(meta && { meta }) },
    { status: 200 }
  );
}

export function created<T>(data: T) {
  return NextResponse.json<ApiResponse<T>>(
    { data, error: null },
    { status: 201 }
  );
}

export function err(
  code: string,
  message: string,
  status: number = 400,
  details?: unknown
) {
  return NextResponse.json<ApiError>(
    { data: null, error: { code, message, ...(details && { details }) } },
    { status }
  );
}

// Common error shortcuts
export const unauthorized = (msg = 'Unauthorized') =>
  err('UNAUTHORIZED', msg, 401);

export const forbidden = (msg = 'Forbidden') =>
  err('FORBIDDEN', msg, 403);

export const notFound = (resource = 'Resource') =>
  err('NOT_FOUND', `${resource} not found`, 404);

export const validationError = (details: unknown) =>
  err('VALIDATION_ERROR', 'Invalid request', 422, details);
```

## Zod Request Validation

```typescript
// lib/api/validate.ts
import { z, ZodSchema } from 'zod';
import { validationError } from './response';

export async function validateBody<T extends ZodSchema>(
  request: Request,
  schema: T
): Promise<{ data: z.infer<T> } | { error: Response }> {
  try {
    const body = await request.json();
    const data = schema.parse(body);
    return { data };
  } catch (e) {
    if (e instanceof z.ZodError) {
      return { error: validationError(e.flatten()) };
    }
    return { error: validationError('Invalid JSON body') };
  }
}

export function validateQuery<T extends ZodSchema>(
  searchParams: URLSearchParams,
  schema: T
): { data: z.infer<T> } | { error: Response } {
  try {
    const raw = Object.fromEntries(searchParams);
    const data = schema.parse(raw);
    return { data };
  } catch (e) {
    if (e instanceof z.ZodError) {
      return { error: validationError(e.flatten()) };
    }
    return { error: validationError('Invalid query parameters') };
  }
}
```

## Cursor-Based Pagination

```typescript
// lib/api/pagination.ts
import { z } from 'zod';
import { gt, asc, desc, type SQL } from 'drizzle-orm';

export const paginationSchema = z.object({
  cursor: z.string().optional(),
  limit: z.coerce.number().min(1).max(100).default(20),
});

export type PaginationInput = z.infer<typeof paginationSchema>;

export interface PaginatedResult<T> {
  items: T[];
  nextCursor: string | null;
  hasMore: boolean;
}

// Usage with Drizzle:
export async function paginateQuery<T extends { id: string }>(
  db: any,
  table: any,
  opts: PaginationInput,
  where?: SQL
): Promise<PaginatedResult<T>> {
  const conditions = [];
  if (where) conditions.push(where);
  if (opts.cursor) conditions.push(gt(table.id, opts.cursor));

  const items = await db
    .select()
    .from(table)
    .where(conditions.length > 0 ? conditions.reduce((a, b) => a && b) : undefined)
    .orderBy(asc(table.id))
    .limit(opts.limit + 1); // Fetch one extra to check hasMore

  const hasMore = items.length > opts.limit;
  if (hasMore) items.pop();

  return {
    items,
    nextCursor: items.length > 0 ? items[items.length - 1].id : null,
    hasMore,
  };
}
```

## Complete API Route Example

```typescript
// app/api/projects/route.ts
import { z } from 'zod';
import { ok, created, unauthorized } from '@/lib/api/response';
import { validateBody, validateQuery } from '@/lib/api/validate';
import { paginationSchema, paginateQuery } from '@/lib/api/pagination';
import { requireUser } from '@/lib/auth/get-user';
import { db } from '@/lib/db';
import { projects } from '@/lib/db/schema';
import { eq } from 'drizzle-orm';

// GET /api/projects?cursor=xxx&limit=20
export async function GET(request: Request) {
  const user = await requireUser().catch(() => null);
  if (!user) return unauthorized();

  const { searchParams } = new URL(request.url);
  const parsed = validateQuery(searchParams, paginationSchema);
  if ('error' in parsed) return parsed.error;

  const result = await paginateQuery(
    db, projects, parsed.data,
    eq(projects.userId, user.userId)
  );

  return ok(result.items, {
    nextCursor: result.nextCursor,
    hasMore: result.hasMore,
  });
}

// POST /api/projects
const createProjectSchema = z.object({
  name: z.string().min(1).max(200),
  description: z.string().max(2000).optional(),
});

export async function POST(request: Request) {
  const user = await requireUser().catch(() => null);
  if (!user) return unauthorized();

  const parsed = await validateBody(request, createProjectSchema);
  if ('error' in parsed) return parsed.error;

  const [project] = await db
    .insert(projects)
    .values({
      ...parsed.data,
      userId: user.userId,
    })
    .returning();

  return created(project);
}
```

---

## Usage (Client-side)

```typescript
// Frontend — consistent API shape
const res = await fetch('/api/projects?limit=10');
const json = await res.json();

if (json.error) {
  console.error(json.error.code, json.error.message);
} else {
  console.log(json.data);      // Project[]
  console.log(json.meta);      // { nextCursor, hasMore }
}
```

## Dependencies

```bash
npm install zod drizzle-orm
```

## Caveats

- Cursor pagination requires a unique, sortable column (usually `id` or `createdAt`). UUIDs work but are not time-ordered — use `cuid2` or `ulid` for time-sortable IDs.
- `request.json()` throws on empty body. The `validateBody` helper catches this, but be aware when testing with curl.
- Zod `.coerce` is needed for query params since they're always strings from `URLSearchParams`.
- For file uploads, skip JSON validation — use `request.formData()` instead.
