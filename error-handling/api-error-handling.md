# API Error Handling

> Unified error response format + try/catch wrapper for Next.js API routes.

**Source**: Community best practices + [Bulletproof React](https://github.com/alan2207/bulletproof-react) (29k⭐, MIT)
**Stack**: TypeScript + Next.js 14 App Router + Zod

---

## Error Response Format

```typescript
// lib/api-error.ts
export class ApiError extends Error {
  constructor(
    message: string,
    public statusCode: number = 500,
    public code?: string
  ) {
    super(message);
    this.name = "ApiError";
  }

  static badRequest(message: string) {
    return new ApiError(message, 400, "BAD_REQUEST");
  }

  static unauthorized(message = "Unauthorized") {
    return new ApiError(message, 401, "UNAUTHORIZED");
  }

  static forbidden(message = "Forbidden") {
    return new ApiError(message, 403, "FORBIDDEN");
  }

  static notFound(message = "Not found") {
    return new ApiError(message, 404, "NOT_FOUND");
  }

  static rateLimit(message = "Too many requests") {
    return new ApiError(message, 429, "RATE_LIMIT");
  }

  static internal(message = "Internal server error") {
    return new ApiError(message, 500, "INTERNAL_ERROR");
  }

  toJSON() {
    return {
      error: {
        message: this.message,
        code: this.code,
        statusCode: this.statusCode,
      },
    };
  }
}
```

---

## API Route Wrapper

```typescript
// lib/api-handler.ts
import { NextResponse } from "next/server";
import { ZodError } from "zod";
import { ApiError } from "./api-error";

type ApiHandler = (req: Request, context?: any) => Promise<Response>;

export function withErrorHandling(handler: ApiHandler): ApiHandler {
  return async (req: Request, context?: any) => {
    try {
      return await handler(req, context);
    } catch (error) {
      console.error(`[API Error] ${req.method} ${req.url}:`, error);

      // Known API errors
      if (error instanceof ApiError) {
        return NextResponse.json(error.toJSON(), {
          status: error.statusCode,
        });
      }

      // Zod validation errors
      if (error instanceof ZodError) {
        return NextResponse.json(
          {
            error: {
              message: "Validation error",
              code: "VALIDATION_ERROR",
              details: error.errors.map((e) => ({
                field: e.path.join("."),
                message: e.message,
              })),
            },
          },
          { status: 400 }
        );
      }

      // Unknown errors
      return NextResponse.json(
        {
          error: {
            message:
              process.env.NODE_ENV === "development"
                ? (error as Error).message
                : "Internal server error",
            code: "INTERNAL_ERROR",
          },
        },
        { status: 500 }
      );
    }
  };
}
```

---

## Usage in API Routes

```typescript
// app/api/projects/route.ts
import { NextResponse } from "next/server";
import { z } from "zod";
import { withErrorHandling } from "@/lib/api-handler";
import { ApiError } from "@/lib/api-error";
import { getUser } from "@/lib/auth";

const createProjectSchema = z.object({
  name: z.string().min(1).max(100),
  description: z.string().optional(),
});

export const POST = withErrorHandling(async (req: Request) => {
  // Auth check
  const user = await getUser();
  if (!user) throw ApiError.unauthorized();

  // Validate body
  const body = await req.json();
  const data = createProjectSchema.parse(body); // throws ZodError if invalid

  // Business logic
  const project = await db.insert(projects).values({
    ...data,
    userId: user.id,
  }).returning();

  return NextResponse.json(project[0], { status: 201 });
});

export const GET = withErrorHandling(async () => {
  const user = await getUser();
  if (!user) throw ApiError.unauthorized();

  const userProjects = await db.query.projects.findMany({
    where: eq(projects.userId, user.id),
  });

  return NextResponse.json(userProjects);
});
```

---

## Client-Side Error Handling

```typescript
// lib/api-client.ts
import { ApiError } from "./api-error";

interface ApiResponse<T> {
  data?: T;
  error?: {
    message: string;
    code: string;
    details?: Array<{ field: string; message: string }>;
  };
}

export async function apiClient<T>(
  url: string,
  options?: RequestInit
): Promise<T> {
  const res = await fetch(url, {
    headers: { "Content-Type": "application/json", ...options?.headers },
    ...options,
  });

  const json: ApiResponse<T> = await res.json();

  if (!res.ok) {
    throw new ApiError(
      json.error?.message || "Request failed",
      res.status,
      json.error?.code
    );
  }

  return json as T;
}

// Usage:
// const projects = await apiClient<Project[]>("/api/projects");
// const project = await apiClient<Project>("/api/projects", {
//   method: "POST",
//   body: JSON.stringify({ name: "My Project" }),
// });
```

---

**Why this pattern**: Consistent error format across all API routes. Zod errors auto-formatted with field-level details. Development mode shows real error messages, production hides them. Single wrapper = DRY.
