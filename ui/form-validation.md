# Form Validation (Zod + react-hook-form)

> Type-safe form validation: Zod schema definition, react-hook-form with zodResolver, and error display patterns.

**Source**: [Bulletproof React](https://github.com/alan2207/bulletproof-react) (29k⭐, MIT)
**Stack**: TypeScript + Zod + react-hook-form

---

## Zod Schema Definitions

```typescript
// lib/validations/index.ts
import { z } from "zod";

// Reusable field validators
export const emailField = z.string().email("Please enter a valid email");
export const passwordField = z
  .string()
  .min(8, "Password must be at least 8 characters")
  .regex(/[A-Z]/, "Password must contain an uppercase letter")
  .regex(/[0-9]/, "Password must contain a number");

// Auth schemas
export const signUpSchema = z
  .object({
    name: z.string().min(2, "Name must be at least 2 characters"),
    email: emailField,
    password: passwordField,
    confirmPassword: z.string(),
  })
  .refine((data) => data.password === data.confirmPassword, {
    message: "Passwords don't match",
    path: ["confirmPassword"],
  });

export const contactSchema = z.object({
  name: z.string().min(1, "Name is required"),
  email: emailField,
  subject: z.string().min(1, "Subject is required"),
  message: z.string().min(10, "Message must be at least 10 characters").max(1000),
});

// Product schema with transforms
export const productSchema = z.object({
  name: z.string().min(1).max(100),
  price: z.coerce.number().positive("Price must be positive"),
  description: z.string().optional(),
  category: z.enum(["saas", "template", "component", "other"]),
  tags: z.array(z.string()).max(5, "Maximum 5 tags"),
  isPublished: z.boolean().default(false),
});

// Infer types from schemas
export type SignUpInput = z.infer<typeof signUpSchema>;
export type ContactInput = z.infer<typeof contactSchema>;
export type ProductInput = z.infer<typeof productSchema>;
```

---

## react-hook-form with zodResolver

```tsx
"use client";

import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { signUpSchema, type SignUpInput } from "@/lib/validations";

export function SignUpForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
    setError,
    reset,
  } = useForm<SignUpInput>({
    resolver: zodResolver(signUpSchema),
    defaultValues: {
      name: "",
      email: "",
      password: "",
      confirmPassword: "",
    },
  });

  async function onSubmit(data: SignUpInput) {
    try {
      const res = await fetch("/api/auth/signup", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(data),
      });

      if (!res.ok) {
        const error = await res.json();
        // Set server-side errors
        if (error.field) {
          setError(error.field as keyof SignUpInput, { message: error.message });
        } else {
          setError("root", { message: error.message || "Something went wrong" });
        }
        return;
      }

      reset();
      // redirect or show success
    } catch {
      setError("root", { message: "Network error. Please try again." });
    }
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
      <div>
        <label htmlFor="name" className="text-sm font-medium">Name</label>
        <input
          id="name"
          {...register("name")}
          className="mt-1 w-full rounded-md border px-3 py-2"
        />
        {errors.name && (
          <p className="mt-1 text-sm text-red-500">{errors.name.message}</p>
        )}
      </div>

      <div>
        <label htmlFor="email" className="text-sm font-medium">Email</label>
        <input
          id="email"
          type="email"
          {...register("email")}
          className="mt-1 w-full rounded-md border px-3 py-2"
        />
        {errors.email && (
          <p className="mt-1 text-sm text-red-500">{errors.email.message}</p>
        )}
      </div>

      <div>
        <label htmlFor="password" className="text-sm font-medium">Password</label>
        <input
          id="password"
          type="password"
          {...register("password")}
          className="mt-1 w-full rounded-md border px-3 py-2"
        />
        {errors.password && (
          <p className="mt-1 text-sm text-red-500">{errors.password.message}</p>
        )}
      </div>

      <div>
        <label htmlFor="confirmPassword" className="text-sm font-medium">Confirm Password</label>
        <input
          id="confirmPassword"
          type="password"
          {...register("confirmPassword")}
          className="mt-1 w-full rounded-md border px-3 py-2"
        />
        {errors.confirmPassword && (
          <p className="mt-1 text-sm text-red-500">{errors.confirmPassword.message}</p>
        )}
      </div>

      {errors.root && (
        <div className="rounded-md bg-red-50 p-3 text-sm text-red-500">
          {errors.root.message}
        </div>
      )}

      <button
        type="submit"
        disabled={isSubmitting}
        className="w-full rounded-md bg-primary px-4 py-2 text-white disabled:opacity-50"
      >
        {isSubmitting ? "Creating account..." : "Sign Up"}
      </button>
    </form>
  );
}
```

---

## Server-Side Validation (API Route)

```typescript
// app/api/auth/signup/route.ts
import { NextResponse } from "next/server";
import { signUpSchema } from "@/lib/validations";

export async function POST(req: Request) {
  try {
    const body = await req.json();
    const data = signUpSchema.parse(body); // throws ZodError if invalid

    // ... create user
    return NextResponse.json({ success: true });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { message: error.errors[0].message, field: error.errors[0].path[0] },
        { status: 400 }
      );
    }
    return NextResponse.json(
      { message: "Internal server error" },
      { status: 500 }
    );
  }
}
```

---

**Dependencies**:

```bash
npm install zod react-hook-form @hookform/resolvers
```

**Key insight**: Define schemas once, use everywhere — client validation, server validation, and TypeScript types all from the same Zod schema.
