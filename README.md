# astro-dev-skill

Agent skill for **Astro 5** web development. Prevents outdated Astro 3/4 patterns that LLM-based coding agents default to.

## What it covers

- **Astro 5 breaking changes** — removed APIs, new patterns, migration guide
- **Content Collections v3** — glob loader, schema functions, `render()` standalone
- **Tailwind CSS v4** — `@tailwindcss/vite`, CSS `@theme`, no config file
- **Documentation strategy** — MCP server integration, LLM-optimized doc endpoints, live verification

## Install

```bash
# Install for current project
npx skills add gigio1023/astro-dev-skill

# Install globally
npx skills add gigio1023/astro-dev-skill -g
```

Works with Claude Code, Codex CLI, Cursor, and [40+ coding agents](https://github.com/vercel-labs/skills).

## Structure

```
skills/astro-dev/
├── SKILL.md                          # Main: top 5 gotchas, doc strategy, workflow
└── references/
    ├── astro5-breaking-changes.md    # Astro 5 removed APIs & new patterns
    ├── content-collections-v3.md     # Content Collections new API
    ├── tailwind-v4.md                # Tailwind v4 migration
    └── doc-endpoints.md              # LLM-optimized documentation URLs
```

## License

MIT
