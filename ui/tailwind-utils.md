# Tailwind CSS Utility Patterns

> Essential Tailwind patterns: cn() implementation, responsive layouts, dark mode toggle, and common utility classes.

**Source**: [Tailwind CSS](https://github.com/tailwindlabs/tailwindcss) (94k⭐, MIT)
**Stack**: TypeScript + Tailwind CSS 3.4+

---

## cn() Implementation (clsx + tailwind-merge)

The single most important utility in any Tailwind project.

```typescript
// lib/utils.ts
import { type ClassValue, clsx } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

**Why both?**
- `clsx` — conditional classes: `clsx("base", isActive && "active", { "hidden": !show })`
- `twMerge` — resolves conflicts: `twMerge("px-4 py-2", "px-6")` → `"px-6 py-2"`
- Together = conditional + conflict-free class merging

```bash
npm install clsx tailwind-merge
```

---

## Responsive Layout Patterns

### Container with Max Width

```tsx
function PageContainer({ children }: { children: React.ReactNode }) {
  return (
    <div className="mx-auto w-full max-w-7xl px-4 sm:px-6 lg:px-8">
      {children}
    </div>
  );
}
```

### Responsive Grid

```tsx
// 1 col mobile → 2 col tablet → 3 col desktop
function CardGrid({ children }: { children: React.ReactNode }) {
  return (
    <div className="grid grid-cols-1 gap-6 sm:grid-cols-2 lg:grid-cols-3">
      {children}
    </div>
  );
}
```

### Sidebar Layout

```tsx
function SidebarLayout({ sidebar, content }: { sidebar: React.ReactNode; content: React.ReactNode }) {
  return (
    <div className="flex min-h-screen">
      {/* Sidebar: hidden on mobile, fixed width on desktop */}
      <aside className="hidden w-64 shrink-0 border-r bg-muted/40 lg:block">
        {sidebar}
      </aside>
      {/* Main content: full width, scrollable */}
      <main className="flex-1 overflow-y-auto p-6">
        {content}
      </main>
    </div>
  );
}
```

### Stack (Vertical Spacing)

```tsx
// Replaces manual margin-top on every child
function Stack({ children, gap = 4 }: { children: React.ReactNode; gap?: number }) {
  return (
    <div className={cn("flex flex-col", `gap-${gap}`)}>
      {children}
    </div>
  );
}
```

---

## Dark Mode Toggle

```tsx
"use client";

import { useTheme } from "next-themes";
import { Moon, Sun } from "lucide-react";
import { Button } from "@/components/ui/button";

export function ThemeToggle() {
  const { theme, setTheme } = useTheme();

  return (
    <Button
      variant="ghost"
      size="icon"
      onClick={() => setTheme(theme === "dark" ? "light" : "dark")}
    >
      <Sun className="h-5 w-5 rotate-0 scale-100 transition-all dark:-rotate-90 dark:scale-0" />
      <Moon className="absolute h-5 w-5 rotate-90 scale-0 transition-all dark:rotate-0 dark:scale-100" />
      <span className="sr-only">Toggle theme</span>
    </Button>
  );
}
```

---

## Common Utility Patterns

```tsx
// Truncate text with ellipsis
<p className="truncate">Very long text that gets cut off...</p>

// Line clamp (multi-line truncate)
<p className="line-clamp-3">Multi-line text that truncates after 3 lines...</p>

// Aspect ratio container
<div className="aspect-video w-full overflow-hidden rounded-lg">
  <img src={src} alt={alt} className="h-full w-full object-cover" />
</div>

// Scroll area with hidden scrollbar
<div className="scrollbar-hide max-h-96 overflow-y-auto">
  {/* scrollable content */}
</div>

// Glass morphism card
<div className="rounded-xl border border-white/20 bg-white/10 p-6 shadow-lg backdrop-blur-md">
  {/* content */}
</div>

// Gradient text
<h1 className="bg-gradient-to-r from-blue-600 to-purple-600 bg-clip-text text-transparent">
  Gradient Heading
</h1>

// Skeleton loading
<div className="h-4 w-3/4 animate-pulse rounded bg-muted" />
```

---

**Note**: Always use `cn()` when accepting `className` props to allow consumers to override styles safely.
