# 🎨 UI Components & Patterns

> Extracted best practices from 100k+ star UI projects for the TypeScript + Next.js + Tailwind stack.

## Modules

| Module | What it does | Source | Stars |
|--------|-------------|--------|-------|
| [shadcn-patterns.md](./shadcn-patterns.md) | cn() utility, Form integration, theme customization | [shadcn/ui](https://github.com/shadcn-ui/ui) | 112k |
| [tailwind-utils.md](./tailwind-utils.md) | cn() impl, responsive layouts, dark mode, utility patterns | [Tailwind CSS](https://github.com/tailwindlabs/tailwindcss) | 94k |
| [form-validation.md](./form-validation.md) | Zod schemas, react-hook-form zodResolver, error handling | [Bulletproof React](https://github.com/alan2207/bulletproof-react) | 29k |

## Recommended Selection

⭐ **If you're starting a new project**, use all three together:

1. **shadcn/ui** for components (copy-paste, you own them)
2. **Tailwind + cn()** for styling (utility-first, no CSS files)
3. **Zod + react-hook-form** for forms (type-safe, single source of truth)

This is the exact stack used by Vercel's own templates and most top Next.js projects.

## Quick Start

```bash
# Initialize shadcn/ui (sets up Tailwind + CSS variables)
npx shadcn-ui@latest init

# Add form components
npx shadcn-ui@latest add form input button

# Install validation
npm install zod react-hook-form @hookform/resolvers

# Install dark mode
npm install next-themes
```
