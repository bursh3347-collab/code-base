# Next.js Error Boundaries

> Global error handling: error.tsx for route segments, global-error.tsx for root layout, and not-found.tsx.

**Source**: [Next.js Documentation](https://nextjs.org/docs/app/building-your-application/routing/error-handling)
**Stack**: TypeScript + Next.js 14 App Router

---

## Route Error Boundary (error.tsx)

```tsx
// app/error.tsx
"use client"; // Error boundaries must be Client Components

import { useEffect } from "react";

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  useEffect(() => {
    // Log to error reporting service
    console.error("Route error:", error);
    // Sentry.captureException(error);
  }, [error]);

  return (
    <div className="flex min-h-[400px] flex-col items-center justify-center gap-4">
      <div className="text-center">
        <h2 className="text-2xl font-bold">Something went wrong</h2>
        <p className="mt-2 text-muted-foreground">
          {error.message || "An unexpected error occurred."}
        </p>
        {error.digest && (
          <p className="mt-1 text-xs text-muted-foreground">
            Error ID: {error.digest}
          </p>
        )}
      </div>
      <div className="flex gap-2">
        <button
          onClick={reset}
          className="rounded-md bg-primary px-4 py-2 text-sm text-primary-foreground hover:bg-primary/90"
        >
          Try again
        </button>
        <a
          href="/"
          className="rounded-md border px-4 py-2 text-sm hover:bg-accent"
        >
          Go home
        </a>
      </div>
    </div>
  );
}
```

---

## Global Error Boundary (global-error.tsx)

Catches errors in the root layout itself.

```tsx
// app/global-error.tsx
"use client";

export default function GlobalError({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <html>
      <body>
        <div style=
          display: "flex",
          minHeight: "100vh",
          flexDirection: "column",
          alignItems: "center",
          justifyContent: "center",
          fontFamily: "system-ui, sans-serif",
        >
          <h1 style= fontSize: "1.5rem", fontWeight: "bold" >
            Something went wrong
          </h1>
          <p style= marginTop: "0.5rem", color: "#666" >
            A critical error occurred. Please try again.
          </p>
          <button
            onClick={reset}
            style=
              marginTop: "1rem",
              padding: "0.5rem 1rem",
              borderRadius: "0.375rem",
              backgroundColor: "#000",
              color: "#fff",
              border: "none",
              cursor: "pointer",
            
          >
            Try again
          </button>
        </div>
      </body>
    </html>
  );
}
```

---

## Not Found Page

```tsx
// app/not-found.tsx
import Link from "next/link";

export default function NotFound() {
  return (
    <div className="flex min-h-[400px] flex-col items-center justify-center gap-4">
      <div className="text-center">
        <h1 className="text-6xl font-bold text-muted-foreground">404</h1>
        <h2 className="mt-2 text-xl font-semibold">Page not found</h2>
        <p className="mt-2 text-muted-foreground">
          The page you're looking for doesn't exist or has been moved.
        </p>
      </div>
      <Link
        href="/"
        className="rounded-md bg-primary px-4 py-2 text-sm text-primary-foreground hover:bg-primary/90"
      >
        Go home
      </Link>
    </div>
  );
}
```

---

## Dashboard-specific Error Boundary

```tsx
// app/(dashboard)/error.tsx
"use client";

export default function DashboardError({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <div className="rounded-lg border border-destructive/50 bg-destructive/10 p-6">
      <h3 className="font-semibold text-destructive">Dashboard Error</h3>
      <p className="mt-2 text-sm text-muted-foreground">{error.message}</p>
      <button
        onClick={reset}
        className="mt-4 rounded-md bg-primary px-3 py-1.5 text-sm text-primary-foreground"
      >
        Retry
      </button>
    </div>
  );
}
```

---

**Key points**:
- `error.tsx` catches errors in its route segment and below
- `global-error.tsx` catches errors in the root layout (must include `<html>` and `<body>`)
- Error boundaries are Client Components (must have `"use client"`)
- `error.digest` is a hash useful for server-side log correlation
- Nest error boundaries for granular error handling per section
