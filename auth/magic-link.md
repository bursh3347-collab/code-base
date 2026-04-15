# Magic Link Passwordless Login

> Zero-password authentication using Supabase Auth magic links. User enters email, receives a one-time link, clicks it, and is authenticated. No passwords to forget, no credentials to leak.

**Source**: [Supabase Auth](https://github.com/supabase/auth) (MIT) + [Supabase JS SDK](https://github.com/supabase/supabase-js) (3.5k⭐, MIT)

**Why this is good**:
- Supabase Auth handles the entire magic link flow: token generation, email delivery, verification, session management
- No need to manage SMTP servers — Supabase includes email delivery (or BYO SMTP)
- Built-in rate limiting, PKCE flow, and token expiry
- SSR-compatible via `@supabase/ssr` package for Next.js App Router

**What we changed**:
- Extracted magic link pattern from Supabase docs into complete copy-paste module
- Added server-side session handling for App Router (not just client-side)
- Added auth callback route with PKCE exchange
- Added protected route middleware pattern

---

## Supabase Client Setup

```typescript
// lib/supabase/client.ts — Browser client
import { createBrowserClient } from '@supabase/ssr';

export function createClient() {
  return createBrowserClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  );
}
```

```typescript
// lib/supabase/server.ts — Server client (App Router)
import { createServerClient } from '@supabase/ssr';
import { cookies } from 'next/headers';

export async function createServerSupabase() {
  const cookieStore = await cookies();

  return createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll() {
          return cookieStore.getAll();
        },
        setAll(cookiesToSet) {
          cookiesToSet.forEach(({ name, value, options }) =>
            cookieStore.set(name, value, options)
          );
        },
      },
    }
  );
}
```

## Magic Link Sign-In Form

```typescript
// components/auth/magic-link-form.tsx
'use client';

import { useState } from 'react';
import { createClient } from '@/lib/supabase/client';

export function MagicLinkForm() {
  const [email, setEmail] = useState('');
  const [sent, setSent] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const [loading, setLoading] = useState(false);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setLoading(true);
    setError(null);

    const supabase = createClient();
    const { error } = await supabase.auth.signInWithOtp({
      email,
      options: {
        emailRedirectTo: `${window.location.origin}/auth/callback`,
      },
    });

    if (error) {
      setError(error.message);
    } else {
      setSent(true);
    }
    setLoading(false);
  };

  if (sent) {
    return (
      <div className="text-center">
        <h2 className="text-lg font-semibold">Check your email</h2>
        <p className="mt-2 text-gray-600">
          We sent a magic link to <strong>{email}</strong>
        </p>
      </div>
    );
  }

  return (
    <form onSubmit={handleSubmit} className="flex flex-col gap-4">
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="you@example.com"
        required
        className="rounded-lg border px-4 py-2.5"
      />
      {error && <p className="text-sm text-red-500">{error}</p>}
      <button
        type="submit"
        disabled={loading}
        className="rounded-lg bg-black px-4 py-2.5 text-white hover:bg-gray-800 disabled:opacity-50"
      >
        {loading ? 'Sending...' : 'Send Magic Link'}
      </button>
    </form>
  );
}
```

## Auth Callback Route (PKCE Exchange)

```typescript
// app/auth/callback/route.ts
import { NextResponse } from 'next/server';
import { createServerSupabase } from '@/lib/supabase/server';

export async function GET(request: Request) {
  const { searchParams, origin } = new URL(request.url);
  const code = searchParams.get('code');
  const next = searchParams.get('next') ?? '/dashboard';

  if (code) {
    const supabase = await createServerSupabase();
    const { error } = await supabase.auth.exchangeCodeForSession(code);

    if (!error) {
      return NextResponse.redirect(`${origin}${next}`);
    }
  }

  // Auth error — redirect to error page
  return NextResponse.redirect(`${origin}/auth-error`);
}
```

## Middleware (Refresh Session)

```typescript
// middleware.ts
import { createServerClient } from '@supabase/ssr';
import { NextResponse, type NextRequest } from 'next/server';

export async function middleware(request: NextRequest) {
  let supabaseResponse = NextResponse.next({ request });

  const supabase = createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll() {
          return request.cookies.getAll();
        },
        setAll(cookiesToSet) {
          cookiesToSet.forEach(({ name, value, options }) => {
            request.cookies.set(name, value);
            supabaseResponse.cookies.set(name, value, options);
          });
        },
      },
    }
  );

  // Refresh session if expired
  const { data: { user } } = await supabase.auth.getUser();

  // Protect dashboard routes
  if (!user && request.nextUrl.pathname.startsWith('/dashboard')) {
    const url = request.nextUrl.clone();
    url.pathname = '/login';
    return NextResponse.redirect(url);
  }

  return supabaseResponse;
}

export const config = {
  matcher: ['/((?!_next/static|_next/image|favicon.ico|.*\\.(?:svg|png|jpg|jpeg|gif|webp)$).*)'],
};
```

## Server-Side: Get Current User

```typescript
// lib/auth/get-user.ts
import { createServerSupabase } from '@/lib/supabase/server';
import { redirect } from 'next/navigation';

export async function getUser() {
  const supabase = await createServerSupabase();
  const { data: { user } } = await supabase.auth.getUser();
  return user;
}

export async function requireUser() {
  const user = await getUser();
  if (!user) redirect('/login');
  return user;
}
```

---

## Usage

```typescript
// In any Server Component:
import { requireUser } from '@/lib/auth/get-user';

export default async function DashboardPage() {
  const user = await requireUser();
  return <h1>Welcome, {user.email}</h1>;
}
```

## Dependencies

```bash
npm install @supabase/supabase-js @supabase/ssr
```

## Environment Variables

```env
NEXT_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJ...
```

## Caveats

- Magic links expire in 1 hour by default (configurable in Supabase dashboard).
- Supabase free tier allows 4 emails/hour per user. For production, configure a custom SMTP provider (Resend, Postmark, etc.).
- The PKCE flow (`exchangeCodeForSession`) is critical for security — never use the deprecated implicit flow.
- `getUser()` makes a network call to Supabase on every invocation. For high-frequency checks, consider caching the session.
- Magic link emails may land in spam — configure a custom domain sender for better deliverability.
