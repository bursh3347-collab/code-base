# OpenAI Streaming with Vercel AI SDK

> Production streaming responses: streamText API route + useChat React hook + error handling + token counting.

**Source**: [Vercel AI SDK](https://github.com/vercel/ai) (12k⭐, Apache-2.0)
**Stack**: TypeScript + Next.js 14 + OpenAI + Vercel AI SDK

---

## API Route (Server)

```typescript
// app/api/chat/route.ts
import { openai } from "@ai-sdk/openai";
import { streamText, convertToCoreMessages } from "ai";

// Allow streaming responses up to 30 seconds
export const maxDuration = 30;

export async function POST(req: Request) {
  try {
    const { messages, system } = await req.json();

    const result = streamText({
      model: openai("gpt-4o-mini"),
      system: system || "You are a helpful assistant.",
      messages: convertToCoreMessages(messages),
      maxTokens: 2048,
      temperature: 0.7,
      // Optional: tool calling
      // tools: { ... },
      onFinish({ text, usage }) {
        // Log token usage for cost tracking
        console.log("Completion:", {
          promptTokens: usage.promptTokens,
          completionTokens: usage.completionTokens,
          totalTokens: usage.totalTokens,
        });
        // Save to DB if needed
      },
    });

    return result.toDataStreamResponse();
  } catch (error) {
    if (error instanceof Error && error.message.includes("rate_limit")) {
      return new Response("Rate limit exceeded. Please try again later.", {
        status: 429,
      });
    }
    return new Response("An error occurred while processing your request.", {
      status: 500,
    });
  }
}
```

---

## React Hook (Client)

```tsx
// components/chat.tsx
"use client";

import { useChat } from "ai/react";
import { useRef, useEffect } from "react";

export function Chat() {
  const {
    messages,
    input,
    handleInputChange,
    handleSubmit,
    isLoading,
    error,
    reload,
    stop,
  } = useChat({
    api: "/api/chat",
    // Optional: initial system message
    body: {
      system: "You are a helpful AI assistant for our SaaS product.",
    },
    onError(error) {
      console.error("Chat error:", error);
    },
  });

  const scrollRef = useRef<HTMLDivElement>(null);

  // Auto-scroll to bottom
  useEffect(() => {
    scrollRef.current?.scrollIntoView({ behavior: "smooth" });
  }, [messages]);

  return (
    <div className="flex h-[600px] flex-col">
      {/* Messages */}
      <div className="flex-1 overflow-y-auto p-4 space-y-4">
        {messages.map((m) => (
          <div
            key={m.id}
            className={`flex ${m.role === "user" ? "justify-end" : "justify-start"}`}
          >
            <div
              className={`max-w-[80%] rounded-lg px-4 py-2 ${
                m.role === "user"
                  ? "bg-primary text-primary-foreground"
                  : "bg-muted"
              }`}
            >
              <p className="whitespace-pre-wrap">{m.content}</p>
            </div>
          </div>
        ))}
        <div ref={scrollRef} />
      </div>

      {/* Error state */}
      {error && (
        <div className="mx-4 mb-2 rounded-md bg-destructive/10 p-3 text-sm text-destructive">
          Something went wrong.
          <button onClick={() => reload()} className="ml-2 underline">
            Try again
          </button>
        </div>
      )}

      {/* Input */}
      <form onSubmit={handleSubmit} className="flex gap-2 border-t p-4">
        <input
          value={input}
          onChange={handleInputChange}
          placeholder="Type a message..."
          className="flex-1 rounded-md border px-3 py-2"
          disabled={isLoading}
        />
        {isLoading ? (
          <button type="button" onClick={stop} className="rounded-md bg-destructive px-4 py-2 text-white">
            Stop
          </button>
        ) : (
          <button type="submit" className="rounded-md bg-primary px-4 py-2 text-white">
            Send
          </button>
        )}
      </form>
    </div>
  );
}
```

---

## Non-Streaming (Simple Completion)

```typescript
// lib/ai.ts
import { generateText } from "ai";
import { openai } from "@ai-sdk/openai";

export async function summarize(text: string): Promise<string> {
  const { text: summary } = await generateText({
    model: openai("gpt-4o-mini"),
    prompt: `Summarize the following text in 2-3 sentences:\n\n${text}`,
    maxTokens: 256,
  });
  return summary;
}
```

---

**Dependencies**:

```bash
npm install ai @ai-sdk/openai
```

**Why Vercel AI SDK**: Unified API across providers (swap OpenAI for Anthropic in one line). Built-in streaming, tool calling, and React hooks. Used in production by Vercel, v0, and thousands of apps.
