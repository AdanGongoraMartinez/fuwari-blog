# Agents

Fuwari is a static blog built with Astro. Fork/cloned from [saicaca/fuwari](https://github.com/saicaca/fuwari).

## Package Manager

**pnpm only** — enforced via `preinstall` hook (`only-allow pnpm`). Do not use npm/yarn/bun.

## Developer Commands

| Command | Action |
|---------|--------|
| `pnpm dev` | Dev server at `localhost:4321` |
| `pnpm build` | `astro build && pagefind --site dist` — two steps chained |
| `pnpm check` | `astro check` (Astro type checking) |
| `pnpm format` | `biome format --write ./src` — **mutates files** |
| `pnpm lint` | `biome check --write ./src` — **mutates files** |
| `pnpm new-post ` | Creates post in `src/content/posts/` |

**Verification order** (from CONTRIBUTING.md): `pnpm check` then `pnpm format`.

## Biome Quirk

Biome ignores `.css` files, `public/**`, `dist/**`, and `node_modules`. **Do not manually edit ignored files** — they are processed by other tools.

**Special overrides** for Svelte/Astro/Vue files: unused variables, unused imports, and `useConst` rules are disabled. These files are exempt from those rules intentionally.

## Content

Posts live in `src/content/posts/`. Use `pnpm new-post` to create with frontmatter template.

Site config: `src/config.ts`. Nav: `navBarConfig`, profile: `profileConfig`.

## Build Artifacts

- `dist/` — static output (gitignored)
- `.astro/` — generated types and cache (gitignored)

## Dependencies

- **Node**: >= 20 required. CI tests on Node 22 and 23.
- **Astro 5** with Svelte, Tailwind CSS, swup (page transitions), Pagefind (search), and Biome.
- Custom Astro plugins in `src/plugins/` (rehype components, remark directives, expressive code config).