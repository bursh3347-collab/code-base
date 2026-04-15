# OAuth Social Login (Google + GitHub)

> Production-ready OAuth 2.0 social login with NextAuth.js (Auth.js v5), supporting Google and GitHub providers with database session persistence via Drizzle adapter.

**Source**: [NextAuth.js / Auth.js](https://github.com/nextauthjs/next-auth) (25k⭐, ISC) + [SaaS-Boilerplate by ixartz](https://github.com/ixartz/SaaS-Boilerplate) (4k⭐, MIT)

**Why this is good**:
- Auth.js v5 is the de facto standard for Next.js authentication — 25k⭐, maintained by Vercel-adjacent team
- Handles OAuth complexity (PKCE, state, nonce, token refresh) automatically
- Database adapter stores sessions server-side (more secure than JWT-only)
- ixartz pattern adds Stripe customer creation on first sign-up — critical for SaaS

**What we changed**:
- Extracted from ixartz's full SaaS boilerplate into standalone auth module
- Replaced Prisma adapter with Drizzle adapter (lighter, faster)
- Added role-based access control callback
- Added Stripe customer auto-creation on sign-up

---

## Auth Configuration

```typescript
// lib/auth/auth.config.ts
import { type NextAuthConfig } from 'next-auth';
import Google from 'next-auth/providers/google';
import GitHub from 'next-auth/providers/github';

export default {
  providers: [
    Google({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    }),
    GitHub({
      clientId: process.env.GITHUB_CLIENT_ID!,
      clientSecret: process.env.GITHUB_CLIENT_SECRET!,
    }),
  ],
} satisfies NextAuthConfig;
```

## Main Auth Setup

```typescript
// lib/auth/index.ts
import NextAuth from 'next-auth';
import { DrizzleAdapter } from '@auth/drizzle-adapter';
import { db } from '@/lib/db';
import authConfig from './auth.config';
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!);

export const {
  handlers: { GET, POST },
  auth,
  signIn,
  signOut,
} = NextAuth({
  adapter: DrizzleAdapter(db),
  session: { strategy: 'database' },
  pages: {
    signIn: '/login',
    error: '/auth-error',
  },
  callbacks: {
    async session({ session, user }) {
      // Inject user ID and role into session
      session.user.id = user.id;
      session.user.role = (user as any).role ?? 'user';
      session.user.stripeCustomerId = (user as any).stripeCustomerId;
      return session;
    },
  },
  events: {
    async createUser({ user }) {
      // Auto-create Stripe customer on first sign-up
      if (user.email) {
        const customer = await stripe.customers.create({
          email: user.email,
          name: user.name ?? undefined,
          metadata: { userId: user.id! },
        });

        // Update user record with Stripe customer ID
        await db
          .update(users)
          .set({ stripeCustomerId: customer.id })
          .where(eq(users.id, user.id!));
      }
    },
  },
  ...authConfig,
});
```

## Drizzle Schema for Auth

```typescript
// lib/db/schema/auth.ts
import {
  pgTable,
  text,
  timestamp,
  primaryKey,
  integer,
} from 'drizzle-orm/pg-core';
import type { AdapterAccountType } from 'next-auth/adapters';

export const users = pgTable('users', {
  id: text('id')
    .primaryKey()
    .$defaultFn(() => crypto.randomUUID()),
  name: text('name'),
  email: text('email').unique().notNull(),
  emailVerified: timestamp('email_verified', { mode: 'date' }),
  image: text('image'),
  role: text('role', { enum: ['user', 'admin'] })
    .default('user')
    .notNull(),
  stripeCustomerId: text('stripe_customer_id'),
  createdAt: timestamp('created_at').defaultNow().notNull(),
});

export const accounts = pgTable(
  'accounts',
  {
    userId: text('user_id')
      .notNull()
      .references(() => users.id, { onDelete: 'cascade' }),
    type: text('type').$type<AdapterAccountType>().notNull(),
    provider: text('provider').notNull(),
    providerAccountId: text('provider_account_id').notNull(),
    refresh_token: text('refresh_token'),
    access_token: text('access_token'),
    expires_at: integer('expires_at'),
    token_type: text('token_type'),
    scope: text('scope'),
    id_token: text('id_token'),
    session_state: text('session_state'),
  },
  (account) => [primaryKey({ columns: [account.provider, account.providerAccountId] })]
);

export const sessions = pgTable('sessions', {
  sessionToken: text('session_token').primaryKey(),
  userId: text('user_id')
    .notNull()
    .references(() => users.id, { onDelete: 'cascade' }),
  expires: timestamp('expires', { mode: 'date' }).notNull(),
});
```

## API Route Handler

```typescript
// app/api/auth/[...nextauth]/route.ts
import { GET, POST } from '@/lib/auth';
export { GET, POST };
```

## Middleware

```typescript
// middleware.ts
import { auth } from '@/lib/auth';

export default auth((req) => {
  const isLoggedIn = !!req.auth;
  const isOnDashboard = req.nextUrl.pathname.startsWith('/dashboard');

  if (isOnDashboard && !isLoggedIn) {
    return Response.redirect(new URL('/login', req.nextUrl));
  }
});

export const config = {
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico).*)'],
};
```

## Sign-In Component

```typescript
// components/auth/social-login-buttons.tsx
'use client';

import { signIn } from 'next-auth/react';

export function SocialLoginButtons() {
  return (
    <div className="flex flex-col gap-3">
      <button
        onClick={() => signIn('google', { callbackUrl: '/dashboard' })}
        className="flex items-center justify-center gap-2 rounded-lg border px-4 py-2.5 hover:bg-gray-50"
      >
        <GoogleIcon className="h-5 w-5" />
        Continue with Google
      </button>
      <button
        onClick={() => signIn('github', { callbackUrl: '/dashboard' })}
        className="flex items-center justify-center gap-2 rounded-lg border px-4 py-2.5 hover:bg-gray-50"
      >
        <GitHubIcon className="h-5 w-5" />
        Continue with GitHub
      </button>
    </div>
  );
}
```

---

## Usage

```typescript
// Server Component — get session
import { auth } from '@/lib/auth';

export default async function Dashboard() {
  const session = await auth();
  if (!session) redirect('/login');
  return <p>Welcome {session.user.email}</p>;
}
```

## Dependencies

```bash
npm install next-auth@beta @auth/drizzle-adapter stripe
```

## Environment Variables

```env
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
GITHUB_CLIENT_ID=
GITHUB_CLIENT_SECRET=
AUTH_SECRET=  # openssl rand -base64 32
STRIPE_SECRET_KEY=
```

## Caveats

- Auth.js v5 is still in beta (`next-auth@beta`). API may change before stable release.
- Database sessions are more secure but slightly slower than JWT. For API-heavy apps, consider JWT strategy.
- Stripe customer creation in `createUser` event means the user record is created first, then updated. If Stripe API fails, the user exists without a Stripe ID — add retry logic for production.
- Google OAuth requires configuring authorized redirect URIs: `https://yourdomain.com/api/auth/callback/google`
