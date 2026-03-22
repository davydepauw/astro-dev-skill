# Astro 5 Breaking Changes & Patterns

## Content Collections — Complete Rewrite

The biggest change in Astro 5. See `content-collections-v3.md` for full details.

**Key difference**: Collections now require an explicit `loader` (glob, file, or custom).

## Config File

- **Preferred**: `astro.config.ts` (TypeScript, with full type inference)
- **Also works**: `astro.config.mjs`
- **Deprecated**: `astro.config.js` (ambiguous module format)

## Removed APIs

| Removed | Replacement |
|---------|-------------|
| `Astro.glob()` | `getCollection()` from `astro:content` |
| `Astro.fetchContent()` | `getCollection()` from `astro:content` |
| `@astrojs/tailwind` integration | `@tailwindcss/vite` as Vite plugin |
| `Content` from frontmatter layout | Explicit layout wrapping in pages |
| `getEntryBySlug()` | `getEntry()` with full ID |

## Rendering Content Entries

```ts
// Astro 5 pattern
import { render } from 'astro:content'

const post = await getEntry('blog', id)
const { Content, headings, remarkPluginFrontmatter } = await render(post)
```

`render()` is now a standalone function imported from `astro:content`, not a method on the entry.

## Static Paths

```ts
// src/pages/blog/[...id].astro
export async function getStaticPaths() {
  const posts = await getCollection('blog')
  return posts.map((post) => ({
    params: { id: post.id },
    props: { post },
  }))
}
```

Note: `post.id` in Astro 5 is the full path relative to the collection base (e.g., `my-post` or `series/part-1`).

## View Transitions

```astro
---
import { ViewTransitions } from 'astro:transitions'
---
<head>
  <ViewTransitions />
</head>
```

- Use `transition:persist` on elements that should survive navigation (e.g., audio players, headers)
- Use `transition:name="unique-name"` for matched animations
- Inline scripts re-run on each navigation unless wrapped in `transition:persist`

## Image Handling

```astro
---
import { Image } from 'astro:assets'
import heroImage from '../assets/hero.png'
---
<Image src={heroImage} alt="Hero" width={800} />
```

- Local images are optimized at build time
- Remote images need `width` and `height` explicitly
- In content collections, use `image()` schema helper for validation

## Middleware

```ts
// src/middleware.ts
import { defineMiddleware } from 'astro:middleware'

export const onRequest = defineMiddleware(async (context, next) => {
  // runs before every route
  const response = await next()
  return response
})
```

## Server Endpoints (API Routes)

```ts
// src/pages/api/data.ts
import type { APIRoute } from 'astro'

export const GET: APIRoute = async ({ request }) => {
  return new Response(JSON.stringify({ ok: true }), {
    headers: { 'Content-Type': 'application/json' },
  })
}

export const POST: APIRoute = async ({ request }) => {
  const body = await request.json()
  return new Response(JSON.stringify({ received: body }))
}
```

For static output, only `GET` endpoints work (pre-rendered at build time).
For `POST`/`PUT`/`DELETE`, need `output: 'server'` or `output: 'hybrid'`.

## Output Modes

| Mode | Behavior |
|------|----------|
| `'static'` (default) | All pages pre-rendered at build time |
| `'server'` | All pages server-rendered by default |
| `'hybrid'` | Static by default, opt-in to server with `export const prerender = false` |

## TypeScript

Astro 5 uses `strictest` tsconfig preset by default:
```json
{
  "extends": "astro/tsconfigs/strictest"
}
```

Path aliases work via `tsconfig.json`:
```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"]
    }
  }
}
```
