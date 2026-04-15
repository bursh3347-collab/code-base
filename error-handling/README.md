# ⚠️ Error Handling Patterns

> Consistent error handling for both server and client in Next.js applications.

## Modules

| Module | What it does | Source |
|--------|-------------|--------|
| [global-error-boundary.md](./global-error-boundary.md) | Next.js error.tsx, global-error.tsx, not-found.tsx | Next.js Docs |
| [api-error-handling.md](./api-error-handling.md) | ApiError class, withErrorHandling wrapper, client-side handler | Bulletproof React 29k⭐ |

## Error Handling Strategy

| Layer | Pattern | Catches |
|-------|---------|--------|
| **API Routes** | `withErrorHandling()` wrapper | Validation errors, auth errors, business logic errors |
| **Route Segments** | `error.tsx` per route group | React rendering errors in that segment |
| **Root Layout** | `global-error.tsx` | Catastrophic errors in root layout |
| **Client Fetch** | `apiClient()` helper | Network errors, API error responses |

⭐ **Key principle**: Errors should be caught at the closest boundary and show user-friendly messages. Raw error details only in development mode.
