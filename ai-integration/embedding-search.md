# Vector Embedding Search

> Complete vector search implementation: embedding generation + pgvector storage + match_documents function + hybrid search.

**Source**: [Supabase AI Docs](https://supabase.com/docs/guides/ai) + [OpenAI Cookbook](https://cookbook.openai.com)
**Stack**: TypeScript + Supabase + OpenAI + pgvector

---

## Embedding Generation Service

```typescript
// lib/embedding-service.ts
import OpenAI from "openai";

const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

interface EmbeddingResult {
  embedding: number[];
  tokensUsed: number;
}

export async function embed(text: string): Promise<EmbeddingResult> {
  const cleaned = text.replace(/\n/g, " ").trim();

  if (!cleaned) throw new Error("Empty text cannot be embedded");

  const response = await openai.embeddings.create({
    model: "text-embedding-3-small",
    input: cleaned,
  });

  return {
    embedding: response.data[0].embedding,
    tokensUsed: response.usage.total_tokens,
  };
}

// Batch embed with rate limiting
export async function embedBatch(
  texts: string[],
  batchSize = 100
): Promise<EmbeddingResult[]> {
  const results: EmbeddingResult[] = [];

  for (let i = 0; i < texts.length; i += batchSize) {
    const batch = texts.slice(i, i + batchSize);
    const cleaned = batch.map((t) => t.replace(/\n/g, " ").trim());

    const response = await openai.embeddings.create({
      model: "text-embedding-3-small",
      input: cleaned,
    });

    results.push(
      ...response.data.map((d) => ({
        embedding: d.embedding,
        tokensUsed: Math.ceil(response.usage.total_tokens / batch.length),
      }))
    );

    // Rate limit: wait between batches
    if (i + batchSize < texts.length) {
      await new Promise((resolve) => setTimeout(resolve, 200));
    }
  }

  return results;
}
```

---

## pgvector Storage & Search Functions

```sql
-- Supabase SQL Editor: Full setup

create extension if not exists vector;

-- Documents with full-text search support
create table documents (
  id bigserial primary key,
  title text,
  content text not null,
  metadata jsonb default '{}'::jsonb,
  embedding vector(1536),
  -- Full-text search vector (for hybrid search)
  fts tsvector generated always as (to_tsvector('english', coalesce(title, '') || ' ' || content)) stored,
  created_at timestamptz default now(),
  updated_at timestamptz default now()
);

-- Indexes
create index documents_embedding_idx on documents using ivfflat (embedding vector_cosine_ops) with (lists = 100);
create index documents_fts_idx on documents using gin (fts);

-- Pure vector similarity search
create or replace function match_documents (
  query_embedding vector(1536),
  match_threshold float default 0.7,
  match_count int default 5,
  filter_metadata jsonb default '{}'::jsonb
)
returns table (
  id bigint,
  title text,
  content text,
  metadata jsonb,
  similarity float
)
language plpgsql
as $$
begin
  return query
  select
    d.id,
    d.title,
    d.content,
    d.metadata,
    1 - (d.embedding <=> query_embedding) as similarity
  from documents d
  where 1 - (d.embedding <=> query_embedding) > match_threshold
    and (filter_metadata = '{}'::jsonb or d.metadata @> filter_metadata)
  order by d.embedding <=> query_embedding
  limit match_count;
end;
$$;

-- Hybrid search (vector + full-text)
create or replace function hybrid_search (
  query_text text,
  query_embedding vector(1536),
  match_count int default 5,
  full_text_weight float default 0.3,
  semantic_weight float default 0.7
)
returns table (
  id bigint,
  title text,
  content text,
  metadata jsonb,
  score float
)
language sql
as $$
  with semantic as (
    select id, 1 - (embedding <=> query_embedding) as semantic_score
    from documents
    order by embedding <=> query_embedding
    limit match_count * 2
  ),
  full_text as (
    select id, ts_rank_cd(fts, websearch_to_tsquery('english', query_text)) as fts_score
    from documents
    where fts @@ websearch_to_tsquery('english', query_text)
    limit match_count * 2
  ),
  combined as (
    select
      coalesce(s.id, f.id) as id,
      coalesce(s.semantic_score, 0) * semantic_weight +
      coalesce(f.fts_score, 0) * full_text_weight as combined_score
    from semantic s
    full outer join full_text f on s.id = f.id
  )
  select
    d.id,
    d.title,
    d.content,
    d.metadata,
    c.combined_score as score
  from combined c
  join documents d on d.id = c.id
  order by c.combined_score desc
  limit match_count;
$$;
```

---

## TypeScript Search Client

```typescript
// lib/search.ts
import { createClient } from "@supabase/supabase-js";
import { embed } from "./embedding-service";

const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!
);

interface SearchResult {
  id: number;
  title: string;
  content: string;
  metadata: Record<string, unknown>;
  similarity?: number;
  score?: number;
}

// Pure semantic search
export async function semanticSearch(
  query: string,
  options?: { threshold?: number; limit?: number; filter?: Record<string, unknown> }
): Promise<SearchResult[]> {
  const { embedding } = await embed(query);

  const { data, error } = await supabase.rpc("match_documents", {
    query_embedding: embedding,
    match_threshold: options?.threshold ?? 0.7,
    match_count: options?.limit ?? 5,
    filter_metadata: options?.filter ?? {},
  });

  if (error) throw new Error(`Search failed: ${error.message}`);
  return data as SearchResult[];
}

// Hybrid search (semantic + full-text)
export async function hybridSearch(
  query: string,
  options?: { limit?: number }
): Promise<SearchResult[]> {
  const { embedding } = await embed(query);

  const { data, error } = await supabase.rpc("hybrid_search", {
    query_text: query,
    query_embedding: embedding,
    match_count: options?.limit ?? 5,
  });

  if (error) throw new Error(`Hybrid search failed: ${error.message}`);
  return data as SearchResult[];
}
```

---

## API Route

```typescript
// app/api/search/route.ts
import { NextResponse } from "next/server";
import { hybridSearch } from "@/lib/search";

export async function POST(req: Request) {
  const { query, limit } = await req.json();

  if (!query) {
    return NextResponse.json({ error: "Query required" }, { status: 400 });
  }

  const results = await hybridSearch(query, { limit: limit ?? 5 });
  return NextResponse.json({ results });
}
```

---

**Dependencies**:

```bash
npm install openai @supabase/supabase-js
```

**Why hybrid search**: Pure vector search misses exact keyword matches. Pure full-text search misses semantic meaning. Hybrid combines both with configurable weights (default: 70% semantic + 30% full-text).
