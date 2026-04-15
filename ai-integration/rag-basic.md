# RAG with Supabase pgvector

> Retrieval-Augmented Generation: Supabase pgvector + OpenAI embeddings + similarity search SQL function.

**Source**: [LlamaIndex.TS](https://github.com/run-llama/LlamaIndexTS) (3.5k⭐, MIT) + [Supabase pgvector Guide](https://supabase.com/docs/guides/ai)
**Stack**: TypeScript + Supabase + OpenAI + pgvector

---

## 1. Database Setup (Supabase SQL Editor)

```sql
-- Enable pgvector extension
create extension if not exists vector;

-- Documents table
create table documents (
  id bigserial primary key,
  content text not null,
  metadata jsonb default '{}'::jsonb,
  embedding vector(1536), -- text-embedding-3-small dimension
  created_at timestamptz default now()
);

-- Index for fast similarity search
create index on documents using ivfflat (embedding vector_cosine_ops)
  with (lists = 100);

-- Similarity search function
create or replace function match_documents (
  query_embedding vector(1536),
  match_threshold float default 0.7,
  match_count int default 5
)
returns table (
  id bigint,
  content text,
  metadata jsonb,
  similarity float
)
language sql stable
as $$
  select
    id,
    content,
    metadata,
    1 - (documents.embedding <=> query_embedding) as similarity
  from documents
  where 1 - (documents.embedding <=> query_embedding) > match_threshold
  order by documents.embedding <=> query_embedding
  limit match_count;
$$;
```

---

## 2. Embedding Generation

```typescript
// lib/embeddings.ts
import OpenAI from "openai";

const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

export async function generateEmbedding(text: string): Promise<number[]> {
  const response = await openai.embeddings.create({
    model: "text-embedding-3-small",
    input: text.replace(/\n/g, " ").trim(),
  });
  return response.data[0].embedding;
}

// Batch embedding (up to 2048 inputs)
export async function generateEmbeddings(
  texts: string[]
): Promise<number[][]> {
  const response = await openai.embeddings.create({
    model: "text-embedding-3-small",
    input: texts.map((t) => t.replace(/\n/g, " ").trim()),
  });
  return response.data.map((d) => d.embedding);
}
```

---

## 3. Document Ingestion

```typescript
// lib/ingest.ts
import { createClient } from "@supabase/supabase-js";
import { generateEmbedding } from "./embeddings";

const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!
);

export async function ingestDocument(
  content: string,
  metadata: Record<string, unknown> = {}
) {
  const embedding = await generateEmbedding(content);

  const { error } = await supabase.from("documents").insert({
    content,
    metadata,
    embedding,
  });

  if (error) throw new Error(`Ingestion failed: ${error.message}`);
}

// Chunk long documents before ingestion
export function chunkText(text: string, chunkSize = 1000, overlap = 200): string[] {
  const chunks: string[] = [];
  let start = 0;

  while (start < text.length) {
    const end = Math.min(start + chunkSize, text.length);
    chunks.push(text.slice(start, end));
    start += chunkSize - overlap;
  }

  return chunks;
}
```

---

## 4. RAG Query (Search + Generate)

```typescript
// lib/rag.ts
import { createClient } from "@supabase/supabase-js";
import { generateEmbedding } from "./embeddings";
import { streamText } from "ai";
import { openai } from "@ai-sdk/openai";

const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!
);

export async function ragQuery(question: string) {
  // 1. Generate embedding for the question
  const queryEmbedding = await generateEmbedding(question);

  // 2. Search for relevant documents
  const { data: documents, error } = await supabase.rpc("match_documents", {
    query_embedding: queryEmbedding,
    match_threshold: 0.7,
    match_count: 5,
  });

  if (error) throw new Error(`Search failed: ${error.message}`);

  // 3. Build context from retrieved documents
  const context = documents
    .map((doc: { content: string; similarity: number }) =>
      `[Relevance: ${(doc.similarity * 100).toFixed(1)}%] ${doc.content}`
    )
    .join("\n\n---\n\n");

  // 4. Generate answer with context
  const result = streamText({
    model: openai("gpt-4o-mini"),
    system: `You are a helpful assistant. Answer questions based on the provided context.
If the context doesn't contain enough information, say so.
Always cite which parts of the context you used.`,
    prompt: `Context:\n${context}\n\nQuestion: ${question}`,
    maxTokens: 1024,
  });

  return result;
}
```

---

## 5. API Route

```typescript
// app/api/rag/route.ts
import { ragQuery } from "@/lib/rag";

export async function POST(req: Request) {
  const { question } = await req.json();

  if (!question) {
    return new Response("Question is required", { status: 400 });
  }

  const result = await ragQuery(question);
  return result.toDataStreamResponse();
}
```

---

**Dependencies**:

```bash
npm install openai @supabase/supabase-js ai @ai-sdk/openai
```

**Cost estimate**: text-embedding-3-small = $0.02/1M tokens. 1000 documents × 500 tokens each = $0.01 to embed. Queries cost ~$0.00002 each.

**Why Supabase pgvector**: No separate vector DB needed. Same Postgres that stores your app data. Free tier includes pgvector. Scales with Supabase.
