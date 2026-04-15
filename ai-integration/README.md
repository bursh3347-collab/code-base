# 🤖 AI Integration Modules

> Extracted AI integration patterns, LLM configurations, and RAG architectures from high-star projects.

## 📦 What's Inside

| Module | Source Project | Type | Best For |
|--------|---------------|------|----------|
| OpenAI SDK | *TBD* | Chat + Function Calling | General LLM integration |
| RAG Pipeline | *TBD* | Retrieval-Augmented Gen | Knowledge-based AI apps |
| Embedding | *TBD* | Vector embeddings | Semantic search, similarity |
| Streaming | *TBD* | SSE / WebSocket | Real-time AI responses |
| Vercel AI SDK | *TBD* | React + AI | Next.js AI apps |

## 🎯 Selection Guide

```
What kind of AI feature?
├── Chat interface → OpenAI SDK + Vercel AI SDK (streaming)
├── Knowledge Q&A → RAG Pipeline (embeddings + vector DB)
├── Search → Embedding + similarity search
└── Agent with tools → OpenAI Function Calling / Agents SDK
```

### ⭐ Recommended Default: Vercel AI SDK + OpenAI

**Why**: Best DX for Next.js, built-in streaming, multi-provider support, React hooks.

## 📁 Directory Structure

```
ai-integration/
├── README.md          ← You are here
├── openai/            ← OpenAI API patterns + function calling
├── rag/               ← RAG pipeline patterns
├── embedding/         ← Embedding generation + vector storage
├── streaming/         ← Streaming response patterns
└── agents/            ← Agent integration patterns
```

---

*Status: 🟡 Scaffolded — Patterns will be populated as projects are analyzed.*
