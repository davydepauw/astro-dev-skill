# Tailwind CSS v4 in Astro

## Setup (Astro 5)

**DO NOT use `@astrojs/tailwind`** — it is deprecated and only supports Tailwind v3.

```ts
// astro.config.ts
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
  vite: {
    plugins: [tailwindcss()],
  },
})
```

Install: `npm install tailwindcss @tailwindcss/vite`

## CSS Entry Point

```css
/* src/styles/global.css */
@import "tailwindcss";
```

Import in your layout:
```astro
---
// src/layouts/Layout.astro
import '../styles/global.css'
---
```

## No Config File

Tailwind v4 does NOT use `tailwind.config.js`. All configuration is in CSS.

```css
/* WRONG (v3 style) */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* CORRECT (v4) */
@import "tailwindcss";
```

## Theme Customization

```css
@import "tailwindcss";

/* Inline theme — adds to default theme */
@theme inline {
  --color-primary: oklch(0.6 0.2 250);
  --color-secondary: oklch(0.7 0.15 200);
  --font-sans: 'Geist', system-ui, sans-serif;
  --font-mono: 'Geist Mono', monospace;
}
```

### Using CSS Custom Properties

```css
@import "tailwindcss";

@theme inline {
  /* Map CSS variables to Tailwind tokens */
  --color-primary: var(--primary);
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --color-muted: var(--muted);
  --color-border: var(--border);
}

/* Then define the actual values per theme */
:root {
  --primary: oklch(0.6 0.2 250);
  --background: #ffffff;
  --foreground: #0a0a0a;
}

[data-theme='dark'] {
  --primary: oklch(0.7 0.2 250);
  --background: #1c1c1c;
  --foreground: #fafafa;
}
```

Usage: `<div class="bg-background text-foreground border-border">`

## Key v3 → v4 Changes

| v3 | v4 |
|----|-----|
| `tailwind.config.js` | CSS `@theme` directive |
| `@tailwind base/components/utilities` | `@import "tailwindcss"` |
| `theme.extend.colors` in JS | `@theme inline { --color-*: ... }` |
| `theme.extend.fontFamily` in JS | `@theme inline { --font-*: ... }` |
| `content: ['./src/**/*.{astro,tsx}']` | Auto-detected (no config needed) |
| `@apply` in components | Still works, but prefer utility classes |
| `darkMode: 'class'` | Auto-detected via `prefers-color-scheme` or `[data-theme]` |

## Utility Class Composition (with clsx + tailwind-merge)

```ts
// src/lib/utils.ts
import { clsx, type ClassValue } from 'clsx'
import { twMerge } from 'tailwind-merge'

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```

Usage in components:
```astro
---
interface Props { class?: string }
const { class: className } = Astro.props
import { cn } from '@/lib/utils'
---
<div class={cn('rounded-lg border p-4', className)}>
  <slot />
</div>
```

## Dark Mode

Tailwind v4 respects `prefers-color-scheme` by default. For manual toggle:

```css
/* Use data attribute instead of class */
@custom-variant dark (&:where([data-theme="dark"], [data-theme="dark"] *));
```

Then toggle via `document.documentElement.dataset.theme = 'dark'`.

## Common Gotchas

1. **Don't install both** `@astrojs/tailwind` and `@tailwindcss/vite` — they conflict
2. **No `content` array needed** — v4 auto-detects template files
3. **`@apply` in `.astro` files** works but be aware of specificity in scoped styles
4. **Arbitrary values** still work: `bg-[#1a1a1a]`, `text-[14px]`
5. **Container queries** are built-in: `@container`, `@lg:flex`
6. **`theme()` function in CSS** is replaced by direct CSS variable references: `var(--color-primary)`
