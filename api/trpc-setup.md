# tRPC End-to-End Type-Safe API

> Full-stack type safety from database to UI with zero code generation. Define your API once, get autocomplete and type checking everywhere.

**Source**: [tRPC](https://github.com/trpc/trpc) (40k⭐, MIT) + [create-t3-app](https://github.com/t3-oss/create-t3-app) (26k⭐, MIT)

**Why this is good**:
- Zero schema duplication — TypeScript types flow from server to client automatically
- No code generation step (unlike GraphQL)
- First-class Next.js App Router support with React Server Components
- Subscriptions, batching, and middleware built-in
- t3-app is the most popular Next.js + tRPC starter (26k⭐)

**What we changed**:
- Extracted tRPC setup from create-t3-app into standalone module
- Simplified for App Router (removed pages/ router setup)
- Added Zod input validation pattern
- Added auth middleware (integrates with JWT or NextAuth)

---

## Server: tRPC Router Setup

```typescript
// server/trpc.ts — Core tRPC initialization
import { initTRPC, TRPCError } from '@trpc/server';
import superjson from 'superjson';
import { ZodError } from 'zod';
import { getUser, type UserJWTPayload } from '@/lib/auth/get-user';

export interface TRPCContext {
  user: UserJWTPayload | null;
}

export async function createTRPCContext(): Promise<TRPCContext> {
  const user = await getUser();
  return { user };
}

const t = initTRPC.context<TRPCContext>().create({
  transformer: superjson,
  errorFormatter({ shape, error }) {
    return {
      ...shape,
      data: {
        ...shape.data,
        zodError:
          error.cause instanceof ZodError ? error.cause.flatten() : null,
      },
    };
  },
});

export const router = t.router;
export const publicProcedure = t.procedure;

// Auth middleware — reusable for all protected routes
const enforceAuth = t.middleware(async ({ ctx, next }) => {
  if (!ctx.user) {
    throw new TRPCError({ code: 'UNAUTHORIZED' });
  }
  return next({ ctx: { user: ctx.user } });
});

export const protectedProcedure = t.procedure.use(enforceAuth);
```

## Define Your API Routes

```typescript
// server/routers/user.ts
import { z } from 'zod';
import { router, protectedProcedure } from '../trpc';
import { db } from '@/lib/db';
import { users } from '@/lib/db/schema';
import { eq } from 'drizzle-orm';

export const userRouter = router({
  me: protectedProcedure.query(async ({ ctx }) => {
    const [user] = await db
      .select()
      .from(users)
      .where(eq(users.id, ctx.user.userId))
      .limit(1);
    return user;
  }),

  updateProfile: protectedProcedure
    .input(
      z.object({
        name: z.string().min(1).max(100),
      })
    )
    .mutation(async ({ ctx, input }) => {
      await db
        .update(users)
        .set({ name: input.name })
        .where(eq(users.id, ctx.user.userId));
      return { success: true };
    }),
});
```

```typescript
// server/routers/_app.ts — Root router
import { router } from '../trpc';
import { userRouter } from './user';

export const appRouter = router({
  user: userRouter,
});

export type AppRouter = typeof appRouter;
```

## Next.js API Route Handler

```typescript
// app/api/trpc/[trpc]/route.ts
import { fetchRequestHandler } from '@trpc/server/adapters/fetch';
import { appRouter } from '@/server/routers/_app';
import { createTRPCContext } from '@/server/trpc';

const handler = (req: Request) =>
  fetchRequestHandler({
    endpoint: '/api/trpc',
    req,
    router: appRouter,
    createContext: createTRPCContext,
  });

export { handler as GET, handler as POST };
```

## Client Setup (React Query)

```typescript
// lib/trpc/client.ts
import { createTRPCReact } from '@trpc/react-query';
import type { AppRouter } from '@/server/routers/_app';

export const trpc = createTRPCReact<AppRouter>();
```

```typescript
// lib/trpc/provider.tsx
'use client';

import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { httpBatchLink } from '@trpc/client';
import { useState } from 'react';
import superjson from 'superjson';
import { trpc } from './client';

export function TRPCProvider({ children }: { children: React.ReactNode }) {
  const [queryClient] = useState(() => new QueryClient());
  const [trpcClient] = useState(() =>
    trpc.createClient({
      links: [
        httpBatchLink({
          url: '/api/trpc',
          transformer: superjson,
        }),
      ],
    })
  );

  return (
    <trpc.Provider client={trpcClient} queryClient={queryClient}>
      <QueryClientProvider client={queryClient}>
        {children}
      </QueryClientProvider>
    </trpc.Provider>
  );
}
```

## Server-Side Caller (RSC)

```typescript
// lib/trpc/server.ts
import { appRouter } from '@/server/routers/_app';
import { createTRPCContext } from '@/server/trpc';

export async function serverTRPC() {
  const ctx = await createTRPCContext();
  return appRouter.createCaller(ctx);
}
```

---

## Usage

```typescript
// Client Component — full autocomplete
'use client';
import { trpc } from '@/lib/trpc/client';

export function Profile() {
  const { data: user } = trpc.user.me.useQuery();
  const updateProfile = trpc.user.updateProfile.useMutation();

  return (
    <div>
      <p>{user?.name}</p>
      <button onClick={() => updateProfile.mutate({ name: 'New Name' })}>
        Update
      </button>
    </div>
  );
}

// Server Component — direct call
import { serverTRPC } from '@/lib/trpc/server';

export default async function DashboardPage() {
  const trpc = await serverTRPC();
  const user = await trpc.user.me();
  return <h1>{user.name}</h1>;
}
```

## Dependencies

```bash
npm install @trpc/server @trpc/client @trpc/react-query @tanstack/react-query superjson zod
```

## Caveats

- tRPC is TypeScript-only. If you need to expose an API to non-TS consumers (mobile, third-party), use REST or GraphQL instead.
- `superjson` transformer is needed to serialize Dates, Maps, etc. Don't forget it on both client and server.
- Batching is enabled by default with `httpBatchLink`. Each page load may send 1 request with multiple procedures bundled.
- For very large apps, consider splitting routers into separate files and using `mergeRouters`.
