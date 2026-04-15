# ⚠️ Error Handling Patterns

> Extracted error handling architectures, boundary patterns, and recovery strategies from high-star projects.

## 📦 What's Inside

| Pattern | Type | Best For |
|---------|------|----------|
| Error Boundaries | React | Graceful UI failure recovery |
| API Error Responses | REST/tRPC | Consistent error formats |
| Global Error Handler | Middleware | Centralized error processing |
| Retry + Circuit Breaker | Resilience | External API calls |

## 🎯 Key Principles

1. **Never expose internals** — User-friendly messages, detailed logs
2. **Consistent format** — Same error shape across all APIs
3. **Graceful degradation** — App should work even when parts fail
4. **Retry with backoff** — For transient failures (network, rate limits)

## 📁 Directory Structure

```
error-handling/
├── README.md              ← You are here
├── react-boundaries/      ← React Error Boundary patterns
├── api-errors/            ← API error response patterns
├── global-handler/        ← Global error handler middleware
└── resilience/            ← Retry, circuit breaker, timeout
```

---

*Status: 🟡 Scaffolded — Patterns will be populated as projects are analyzed.*
