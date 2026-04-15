# Supabase Best Practices

> Row-Level Security, Realtime subscriptions, and Storage patterns for Next.js. The complete Supabase playbook for SaaS.

**Source**: [Supabase](https://github.com/supabase/supabase) (78k⭐, Apache-2.0) + [Supabase documentation](https://supabase.com/docs)

**Why this is good**:
- Supabase = Postgres + Auth + Realtime + Storage + Edge Functions in one platform
- RLS (Row-Level Security) is the gold standard for multi-tenant data isolation
- Realtime subscriptions are built on Postgres logical replication — no extra infrastructure
- Free tier is generous (500MB DB, 1GB storage, 50K monthly active users)

**What we changed**:
- Compiled scattered documentation into one executable playbook
- Added multi-tenant RLS patterns (user-level + team-level)
- Added typed Realtime subscription helper
- Added Storage upload with automatic image optimization

---

## Row-Level Security (RLS)

### Enable RLS on All Tables

```sql
-- ALWAYS enable RLS. Without it, any authenticated user can read/write all rows.
alter table public.projects enable row level security;
alter table public.documents enable row level security;
```

### User-Level Isolation

```sql
-- Users can only see their own data
create policy "Users read own projects"
  on public.projects for select
  using (auth.uid() = user_id);

create policy "Users insert own projects"
  on public.projects for insert
  with check (auth.uid() = user_id);

create policy "Users update own projects"
  on public.projects for update
  using (auth.uid() = user_id);

create policy "Users delete own projects"
  on public.projects for delete
  using (auth.uid() = user_id);
```

### Team-Level Isolation (Multi-Tenant)

```sql
-- Users can see data belonging to their team
create policy "Team members read team projects"
  on public.projects for select
  using (
    team_id in (
      select team_id from public.team_members
      where user_id = auth.uid()
    )
  );

-- Only team admins can delete
create policy "Team admins delete projects"
  on public.projects for delete
  using (
    team_id in (
      select team_id from public.team_members
      where user_id = auth.uid() and role = 'admin'
    )
  );
```

### Service Role Bypass (Server-Side Only)

```typescript
// lib/supabase/admin.ts — NEVER expose on client
import { createClient } from '@supabase/supabase-js';

export const supabaseAdmin = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!, // Bypasses RLS
  { auth: { persistSession: false } }
);
```

## Realtime Subscriptions

```typescript
// hooks/use-realtime.ts
'use client';

import { useEffect, useState } from 'react';
import { createClient } from '@/lib/supabase/client';
import type { RealtimePostgresChangesPayload } from '@supabase/supabase-js';

type ChangeEvent = 'INSERT' | 'UPDATE' | 'DELETE';

export function useRealtime<T extends Record<string, unknown>>(
  table: string,
  filter?: string, // e.g., 'team_id=eq.xxx'
  events: ChangeEvent[] = ['INSERT', 'UPDATE', 'DELETE']
) {
  const [changes, setChanges] = useState<
    RealtimePostgresChangesPayload<T>[]
  >([]);

  useEffect(() => {
    const supabase = createClient();

    const channel = supabase
      .channel(`realtime:${table}`)
      .on<T>(
        'postgres_changes',
        {
          event: events.length === 1 ? events[0] : '*',
          schema: 'public',
          table,
          ...(filter && { filter }),
        },
        (payload) => {
          setChanges((prev) => [...prev.slice(-99), payload]);
        }
      )
      .subscribe();

    return () => {
      supabase.removeChannel(channel);
    };
  }, [table, filter]);

  return changes;
}
```

```typescript
// Usage in component
function ProjectList({ teamId }: { teamId: string }) {
  const changes = useRealtime('projects', `team_id=eq.${teamId}`);

  useEffect(() => {
    if (changes.length > 0) {
      const latest = changes[changes.length - 1];
      if (latest.eventType === 'INSERT') {
        toast.success('New project created!');
        // Refresh data
      }
    }
  }, [changes]);
}
```

## Storage Patterns

```typescript
// lib/supabase/storage.ts
import { createClient } from '@/lib/supabase/client';

const BUCKET = 'avatars'; // Create bucket in Supabase dashboard

export async function uploadAvatar(
  userId: string,
  file: File
): Promise<{ url: string } | { error: string }> {
  const supabase = createClient();

  // Validate file
  if (file.size > 5 * 1024 * 1024) {
    return { error: 'File must be under 5MB' };
  }
  if (!['image/jpeg', 'image/png', 'image/webp'].includes(file.type)) {
    return { error: 'Only JPEG, PNG, and WebP are allowed' };
  }

  const ext = file.name.split('.').pop();
  const path = `${userId}/avatar.${ext}`;

  const { error } = await supabase.storage
    .from(BUCKET)
    .upload(path, file, {
      cacheControl: '3600',
      upsert: true, // Overwrite existing avatar
    });

  if (error) return { error: error.message };

  // Get public URL with automatic transformation
  const { data: { publicUrl } } = supabase.storage
    .from(BUCKET)
    .getPublicUrl(path, {
      transform: {
        width: 200,
        height: 200,
        resize: 'cover',
        format: 'webp',
        quality: 80,
      },
    });

  return { url: publicUrl };
}

export async function deleteAvatar(userId: string) {
  const supabase = createClient();
  await supabase.storage.from(BUCKET).remove([`${userId}/avatar.webp`]);
}
```

### Storage RLS Policy

```sql
-- Users can upload to their own folder
create policy "Users upload own avatar"
  on storage.objects for insert
  with check (
    bucket_id = 'avatars'
    and (storage.foldername(name))[1] = auth.uid()::text
  );

-- Anyone can read avatars (public bucket)
create policy "Public avatar access"
  on storage.objects for select
  using (bucket_id = 'avatars');
```

---

## Usage

```typescript
// Server Component with RLS
import { createServerSupabase } from '@/lib/supabase/server';

export default async function Projects() {
  const supabase = await createServerSupabase();
  // RLS automatically filters to current user's projects
  const { data: projects } = await supabase.from('projects').select('*');
  return <ProjectList projects={projects ?? []} />;
}
```

## Dependencies

```bash
npm install @supabase/supabase-js @supabase/ssr
```

## Caveats

- **RLS is OFF by default**. Forgetting to enable it = public data leak. Always enable RLS on every table.
- **Service role key** bypasses RLS. Never expose it to the client. Only use in server-side code.
- Realtime requires enabling replication for the table: `alter publication supabase_realtime add table projects;`
- Storage image transformations are a Pro feature on Supabase. On free tier, handle resizing client-side or use a CDN.
- Supabase free tier pauses after 1 week of inactivity. For always-on, upgrade to Pro ($25/mo).
