---
name: astro-dev
description: "Astro 5 web development with modern stack (Tailwind v4, Content Collections v3, MDX, React islands). Use when working in any Astro project, editing .astro/.mdx files, modifying astro.config.*, or asking about Astro patterns. Prevents common mistakes with outdated Astro 3/4 syntax that LLM-based coding agents default to."
---

# Astro Dev

## Documentation Strategy

Before writing Astro code from memory, check the official docs. Astro evolves fast — URLs, MCP setup, and APIs may change between versions. **Always verify against the live source before trusting hardcoded references in this skill.**

### Step 1: Check for MCP tool availability

Search your available tools for anything matching `astro` or `astro_docs`. If found, use it:
```
search_astro_docs({ query: "content collections" })
```

### Step 2: If no MCP tool, fetch the latest AI integration guide

Fetch the live page to get the current MCP server URL and setup instructions:
```
WebFetch("https://docs.astro.build/en/guides/build-with-ai/")
```
This page is maintained by the Astro team and contains the canonical MCP config. If the MCP URL or setup in `references/doc-endpoints.md` differs from this live page, **trust the live page**.

### Step 3: Use LLM-optimized doc endpoints for code reference

```
https://docs.astro.build/llms-full.txt          # Complete docs
https://docs.astro.build/_llms-txt/api-reference.txt  # API reference
```

See `references/doc-endpoints.md` for the full list of endpoints and which to use per task.

### Step 4: Fall back to this skill's reference files

Use the curated gotcha lists and migration guides in `references/` when web access is unavailable or for quick offline reference. These files are snapshots and may lag behind the latest Astro release.

---

## Quick Router — Read the right file for your task

| What you're doing | Read this file |
|---|---|
| **Astro 5 migration / new project** | `references/astro5-breaking-changes.md` |
| **Content collections** (schema, loader, querying) | `references/content-collections-v3.md` |
| **Tailwind CSS** (config, theming, classes) | `references/tailwind-v4.md` |
| **Finding documentation** (URLs, LLM endpoints) | `references/doc-endpoints.md` |

Load **only the module you need**. Never preload all.

---

## Top 5 Gotchas (always relevant)

**1. Content Collections require explicit `loader` (Astro 5):**
```ts
// WRONG (Astro 4 style - no loader)
const blog = defineCollection({ schema: z.object({...}) })

// CORRECT (Astro 5 - glob loader required)
import { glob } from 'astro/loaders'
const blog = defineCollection({
  loader: glob({ pattern: '**/*.{md,mdx}', base: './src/content/blog' }),
  schema: ({ image }) => z.object({...})
})
```
Schema is now a **function** receiving helpers like `image()`.

**2. Tailwind v4 has no config file:**
```css
/* WRONG (v3) */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* CORRECT (v4) */
@import "tailwindcss";
@theme inline {
  --color-primary: oklch(0.6 0.2 250);
}
```
Use `@tailwindcss/vite` plugin, NOT `@astrojs/tailwind` (deprecated).

**3. React components need `client:*` directives:**
```astro
<!-- WRONG: renders server-side only, no interactivity -->
<Counter />

<!-- CORRECT: hydrates on client -->
<Counter client:idle />
```
Options: `client:load` (immediate), `client:idle` (after idle), `client:visible` (in viewport).

**4. `Astro.glob()` is removed in Astro 5:**
```ts
// WRONG
const posts = await Astro.glob('./posts/*.md')

// CORRECT - use content collections
import { getCollection } from 'astro:content'
const posts = await getCollection('blog')
```

**5. Use `astro.config.ts` (not `.mjs`):**
Astro 5 fully supports TypeScript config. Prefer `.ts` for type safety.
```ts
import { defineConfig } from 'astro/config'
export default defineConfig({
  site: 'https://example.com',
  integrations: [/* ... */],
})
```

---

## Common Integration Stack (Astro 5)

```ts
// astro.config.ts
import { defineConfig } from 'astro/config'
import mdx from '@astrojs/mdx'
import react from '@astrojs/react'
import sitemap from '@astrojs/sitemap'

export default defineConfig({
  site: 'https://example.com',
  integrations: [mdx(), react(), sitemap()],
  // Tailwind configured via @tailwindcss/vite in vite.plugins, NOT as integration
  vite: {
    plugins: [tailwindcss()], // from @tailwindcss/vite
  },
})
```

---

## Environment Variables

```ts
// Client-side (exposed to browser): must have PUBLIC_ prefix
import.meta.env.PUBLIC_SITE_URL

// Server-side only (build time, SSR endpoints):
import.meta.env.SECRET_API_KEY

// In astro.config.ts, use process.env or astro:env module
```

---

## Workflow: Explore Before Modifying

1. **Check Astro version**: `package.json` → `"astro"` version determines API surface
2. **Check config format**: `.ts` vs `.mjs`, which integrations are installed
3. **Check content schema**: `src/content.config.ts` or `src/content/config.ts`
4. **Check Tailwind version**: v3 uses `tailwind.config.js`, v4 uses CSS `@theme`
5. **Then write code** using the correct API for the detected versions
