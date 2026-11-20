# Repository Guidelines

GymBuddy is a mobile-first workout tracker that shows a lifter the last load used per exercise. It is built on the 10x Astro Starter: Astro 7 SSR, React 19 islands, Tailwind 4, Supabase auth/Postgres, and it is deployed to Cloudflare Workers. Product scope is in @context/foundation/prd.md and stack decisions are in @context/foundation/tech-stack.md.

## Hard Rules

- Enable RLS on every new table, with separate policies per operation and role. A user's data must never be readable by another user (PRD guardrail).
- Name migrations `supabase/migrations/YYYYMMDDHHmmss_short_description.sql`.
- Read secrets only through `astro:env/server` (schema in @astro.config.mjs). A new required env var also needs an entry in `configStatuses` in @src/lib/config-status.ts, which drives the missing-config banner.
- `createClient()` in @src/lib/supabase.ts returns `null` when env is missing. Handle that branch the way @src/pages/api/auth/signin.ts does.
- A set confirmed offline must not be lost. The starter has no offline queue. Set logging must write to a client-side queue (IndexedDB or a service worker) before calling the API, and remove the entry only after a 2xx response (@context/foundation/tech-stack.md).
- Deploy target is Cloudflare Workers (@wrangler.jsonc), not Pages. Never use `Astro.locals.runtime` (removed in adapter v14). Read bindings via `cloudflare:workers`. Do not rename `name` in `wrangler.jsonc`, because that creates a new Worker and URL.
- Pushing to `master` deploys to production (`deploy` job in @.github/workflows/ci.yml). Production secrets live in Workers Secrets (`wrangler secret put`), not in the build.
- Do not edit `context/archive/`. Archived changes are immutable.

## Project Structure

- `src/pages/` holds routes. `src/pages/api/` holds endpoints.
- `src/components/` holds Astro components for static markup and React only for interactivity. shadcn/ui lives in `src/components/ui/`.
- `src/lib/` holds services and helpers. `src/types.ts` holds shared DTOs.
- `src/middleware.ts` sets `locals.user`. Add protected paths to `PROTECTED_ROUTES`.
- `context/` holds the PRD, stack, and change docs.

## Commands

- `npm run dev`: dev server on the workerd runtime (port 4321).
- `npm run lint` and `npx astro check`: the same gates CI runs.
- `npm run smoke`: auth-flow check against a running server (`BASE_URL`).
- Build, local Supabase, and deploy steps: @README.md. All scripts: @package.json.

## Coding Style

- Import from `src` via `@/*`. Merge Tailwind classes with `cn()` from `@/lib/utils`, never string concatenation.
- Don't use `"use client"` or other Next.js directives. Put React hooks in `src/components/hooks/`.
- The husky pre-commit hook runs lint-staged (`eslint --fix`, `prettier --write`). Do not bypass it.

## Testing

There is no unit or E2E framework yet. `scripts/smoke.mjs` only guards the starter's auth flow. Add a test runner as part of the first feature that needs one.

## Commits & PRs

The history is too short to show a convention. Write imperative summaries, and target `main` on `github.com/p4we3/GymBuddyx10xDevs`.

More starter conventions: @CLAUDE.md.scaffold.
