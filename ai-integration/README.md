# 🤖 AI Integration Patterns

> Production AI patterns for Next.js + OpenAI + Supabase.

## Modules

| Module | What it does | Source | Stars |
|--------|-------------|--------|-------|
| [openai-streaming.md](./openai-streaming.md) | Vercel AI SDK streaming, useChat hook, error handling | [Vercel AI SDK](https://github.com/vercel/ai) | 12k |
| [rag-basic.md](./rag-basic.md) | RAG pipeline: pgvector + OpenAI embeddings + similarity search | [LlamaIndex.TS](https://github.com/run-llama/LlamaIndexTS) + Supabase | 3.5k |
| [embedding-search.md](./embedding-search.md) | Vector search: embedding generation + hybrid search (semantic + full-text) | Supabase AI Docs + OpenAI Cookbook | — |

## Recommended Selection

⭐ **For a SaaS with AI chat**: Use `openai-streaming.md` — streaming responses with `useChat` hook.

⭐ **For a SaaS with knowledge base**: Use all three:
1. `rag-basic.md` for the pipeline (ingest → embed → store → search → generate)
2. `embedding-search.md` for advanced hybrid search
3. `openai-streaming.md` for the chat UI

## Quick Start

```bash
# Install AI dependencies
npm install ai @ai-sdk/openai openai @supabase/supabase-js

# Set env vars
OPENAI_API_KEY="sk-..."
NEXT_PUBLIC_SUPABASE_URL="https://xxx.supabase.co"
SUPABASE_SERVICE_ROLE_KEY="eyJ..."
```

## Cost Guide

| Operation | Model | Cost |
|-----------|-------|------|
| Embedding | text-embedding-3-small | $0.02 / 1M tokens |
| Chat | gpt-4o-mini | $0.15 / 1M input, $0.60 / 1M output |
| Chat | gpt-4o | $2.50 / 1M input, $10 / 1M output |

A typical SaaS with 1000 DAU and moderate AI usage: ~$50-100/month.
