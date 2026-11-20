---
project: gymbuddy
researched_at: 2026-11-18
recommended_platform: Cloudflare Workers (static assets)
runner_up: Vercel
context_type: mvp
tech_stack:
  language: TypeScript
  framework: Astro 7 (SSR) + React 19, Supabase (external)
  runtime: Cloudflare workerd (@astrojs/cloudflare 14, wrangler 4)
---

## Recommendation

**Deploy on Cloudflare Workers (with static assets) — not Cloudflare Pages.**

The repo is already configured for Workers (`wrangler.jsonc`, `@astrojs/cloudflare` 14, `astro:env` secrets), so there is zero adapter migration before the 2026-12-06 deadline. Workers is the only shortlisted platform passing all five agent-friendly criteria at $0/month for this traffic (with a $5/month fallback), matching the interview answers: cost minimization first, prior Cloudflare familiarity, single region (Poland), no persistent connections, external Supabase. Vercel tied on criteria but needs an adapter swap, a manual EU region, and its free tier is non-commercial only.

> Correction to `tech-stack.md`: it lists `deployment_target: cloudflare-pages`. The Astro Cloudflare adapter v13+ no longer supports Pages; the target is Workers. Update the hint when tech-stack is next revised.

## Platform Comparison

Interview: Q1 persistent connections = No · Q2 = minimize cost · Q3 = familiar with Cloudflare and Railway · Q4 = single region · Q5 = external providers OK. No platform removed by hard filters.

| Platform           | CLI-first | Managed/Serverless | Agent-readable docs | Stable deploy API | MCP / Integration | Total | Est. cost (10k–100k req/mo)           |
| ------------------ | --------- | ------------------ | ------------------- | ----------------- | ----------------- | ----- | ------------------------------------- |
| Cloudflare Workers | Pass      | Pass               | Pass                | Pass              | Pass              | 5     | $0 (free) / $5 (paid, if CPU cap hit) |
| Vercel             | Pass      | Pass               | Pass                | Pass              | Pass              | 5     | $0 Hobby (non-commercial) / $20 Pro   |
| Render             | Partial   | Pass               | Pass                | Pass              | Pass              | 4.5   | $0 (≈1 min cold start) / $7           |
| Netlify            | Partial   | Pass               | Pass                | Pass              | Partial           | 4     | ≈$9 (credits); EU region Pro-only     |
| Railway            | Partial   | Pass               | Pass                | Partial           | Partial           | 3.5   | ≈$5                                   |
| Fly.io             | Partial   | Partial            | Pass                | Pass              | Partial           | 3.5   | ≈$1–5, no free tier                   |

Status checks as of 2026-11-18.

- **Cloudflare Workers** — `wrangler deploy` / `rollback` / `tail` cover the full loop from the terminal; `llms.txt` per product; MCP servers at `mcp.cloudflare.com` and `docs.mcp.cloudflare.com` (no GA/beta label stated). Free plan: 100k req/day but **10 ms CPU per request**; paid $5/mo with 10M requests. Static assets free and unmetered. Version (preview) URLs exist (status not stated) but do not appear in `wrangler tail`/Workers Logs.
- **Vercel** — `vercel deploy --prod`, `vercel rollback`, `vercel logs` (query-based; `--follow` deprecated). `llms.txt` + `.md` docs. Official MCP `mcp.vercel.com` works with Claude Code. Hobby is non-commercial only, keeps 1 h of logs, rolls back only one deploy. Functions default to `iad1`; `fra1` must be set and Hobby allows one region. Needs `@astrojs/vercel` (Astro 7 compatible).
- **Render** — CLI deploy and log tail; rollback only via dashboard/API (Partial). MCP server GA since 2025-08-21. Frankfurt GA. Free web service spins down after 15 min with ~1 min cold start — unacceptable at the gym; $7/mo removes it. Hobby bandwidth cut to 5 GB (2026-08-01). Needs `@astrojs/node`.
- **Netlify** — no CLI rollback (Partial); MCP Claude Code support for the stateless version "coming soon" (Partial). Credit pricing: free plan 300 credits hard cap, 15 credits per production deploy. Function region is US-East on Free/Personal; EU only on Pro. Blobs still beta; Netlify Server early access.
- **Railway** — rollback dashboard-only (Partial); `railway.json/.toml` config-as-code stops being read **2026-12-01**, successor IaC labelled both GA and experimental (Partial); MCP status unstated (Partial). Always-on ≈$5/mo. Needs `@astrojs/node`.
- **Fly.io** — rollback is manual image redeploy (Partial); container + Dockerfile management (Partial); `fly mcp server` experimental (Partial). No free tier for new orgs; Warsaw region deprecated (use fra/ams); ~2 s cold start from stop.

### Shortlisted Platforms

#### 1. Cloudflare Workers (Recommended)

Five Passes, lowest cost at this scale, already wired into the repo (adapter, `wrangler.jsonc`, `astro:env`, `.dev.vars`), and the developer has prior Cloudflare experience. `astro dev`/`astro preview` run on real workerd through the Cloudflare Vite plugin, so local behaviour matches production.

#### 2. Vercel

Equal on criteria and with the strongest Claude Code MCP story, but requires replacing the adapter, pinning `fra1`, and the free Hobby plan forbids commercial use (Pro is $20/mo) — worse on the cost priority and on time-to-deploy.

#### 3. Render

GA MCP server, Frankfurt region, plain Node runtime with no CPU cap. Loses on cost-for-UX ($7/mo to avoid a 1-minute cold start), no CLI rollback, and an adapter migration.

## Anti-Bias Cross-Check: Cloudflare Workers

### Devil's Advocate — Weaknesses

1. **10 ms CPU cap on the free plan.** React 19 SSR of a long set list plus Supabase session refresh in middleware can intermittently exceed it, producing error 1102 in production only.
2. **Edge-to-database latency.** The Worker runs at the PoP nearest the user; Supabase sits in one region. Pages that issue several sequential Supabase queries multiply the round-trip — the "< 1 s set confirmation" NFR depends on queries per request.
3. **workerd compatibility.** CommonJS-only or full-Node dependencies can fail on deploy despite working in plain Node; `nodejs_compat` covers only part of the Node API.
4. **Stale patterns from agents.** Adapter v14 removed `Astro.locals.runtime`; most tutorials and agent training data still use it, so generated code will be wrong unless redirected to `astro:env` / `cloudflare:workers`.
5. **Preview deployments are public and log-less.** Version URLs live on public `workers.dev`, their logs don't reach `wrangler tail`, and without a separate Supabase project they hit production data.

### Pre-Mortem — How This Could Fail

The team shipped GymBuddy on the Workers free plan because "a few users cost nothing." Locally everything worked; in production the workout screen failed now and then. Observability was enabled, but nobody read it — the cause was the 10 ms CPU cap during list rendering and token refresh. Next, a preview build pointed at the same Supabase project as production and wrote test sets into real history. Then a broken migration prompted a rollback: `wrangler rollback` restored the code but not the schema, and the old code crashed on new columns. Finally the browser cached an old service worker holding the offline queue; sets logged without signal at the gym were posted to a removed endpoint and silently dropped — violating the PRD's core NFR. None of this was the platform's fault; each failure came from assuming "serverless = zero operations."

### Unknown Unknowns

- `tech-stack.md` says Cloudflare **Pages**, but adapter v13+ doesn't support Pages; following Pages Git-integration docs is a dead end.
- The target environment is chosen at **build time** (`CLOUDFLARE_ENV`), not at deploy time — one bundle cannot be promoted from preview to production; CI builds once per environment.
- `wrangler rollback` reverts code only; secrets, bindings, Supabase schema and data stay as they are.
- Supabase Auth redirect URLs must allowlist the `workers.dev` preview hosts, or sign-in breaks on previews.
- The offline-queue service worker needs `Cache-Control: no-cache` (via `public/_headers`), or users stay pinned to an old queue implementation.
- Default branch is `master` (renamed from `main` on 2026-11-18); `.github/workflows/ci.yml` already triggers on `master`.

## Operational Story

- **Preview deploys**: CI on a PR runs `wrangler versions upload --preview-alias pr-<n>` → `pr-<n>-<worker>.<subdomain>.workers.dev`. Public by default; point previews at a separate Supabase project (or local data) and add the host to Supabase Auth redirect URLs. Fork PRs get no secrets, so no previews from forks.
- **Secrets**: production runtime secrets live in Workers Secrets (`wrangler secret put SUPABASE_URL` / `SUPABASE_KEY`); locally in gitignored `.dev.vars`; CI deploy credentials (`CLOUDFLARE_API_TOKEN`, `CLOUDFLARE_ACCOUNT_ID`) in GitHub Secrets. Rotate by re-running `wrangler secret put` (takes effect on next request; no redeploy needed) and updating GitHub Secrets.
- **Rollback**: `npx wrangler deployments list` → `npx wrangler rollback <version-id>`; takes seconds. Code only — Supabase migrations must be written backward-compatible (expand → migrate → contract) because the DB does not roll back.
- **Approval**: a human approves production deploys (merge to `master`), secret rotation, Supabase migrations against production, and any data deletion. An agent may run builds, upload preview versions, read logs, and propose (not execute) a rollback.
- **Logs**: `npx wrangler tail --format pretty` (live, production only), Workers Logs in the dashboard (observability already enabled in `wrangler.jsonc`), CI logs via `gh run view --log`. Cloudflare MCP (`mcp.cloudflare.com/mcp`) for structured read access.

## Risk Register

| Risk                                                        | Source                              | Likelihood | Impact | Mitigation                                                                                                                           |
| ----------------------------------------------------------- | ----------------------------------- | ---------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------ |
| Free-plan 10 ms CPU cap exceeded → intermittent 1102 errors | Devil's advocate                    | M          | H      | Check CPU time per request in Workers Logs after first deploy; switch to Workers Paid ($5/mo) on any exceedance; keep SSR pages lean |
| Sequential Supabase queries push set confirmation past 1 s  | Devil's advocate                    | M          | M      | One query (or RPC) per request on the workout flow; confirm sets optimistically on the client                                        |
| Dependency incompatible with workerd                        | Devil's advocate                    | L          | M      | Always verify with `npm run build && npm run preview` (runs on workerd) before merge; prefer fetch-based libraries                   |
| Agent generates `Astro.locals.runtime` / Pages-era code     | Devil's advocate / Unknown unknowns | H          | L      | Add a rule to `AGENTS.md`: env via `astro:env/server` or `cloudflare:workers`; target is Workers, not Pages                          |
| Preview version writes to production data                   | Pre-mortem                          | M          | H      | Separate Supabase project (or local Supabase) for previews; never give preview builds production secrets                             |
| Rollback leaves incompatible DB schema                      | Pre-mortem                          | M          | H      | Backward-compatible migrations only; a human runs production migrations separately from code deploys                                 |
| Stale service worker drops offline-queued sets              | Pre-mortem                          | M          | H      | `public/_headers` with `Cache-Control: no-cache` for the SW; version the queue payload; server endpoint accepts old payload versions |
| Preview logs invisible in `wrangler tail`                   | Research finding                    | H          | L      | Debug previews locally with `npm run preview`; rely on CI smoke test                                                                 |
| MCP servers / Version URLs have no stated GA status         | Research finding                    | L          | L      | Treat as convenience; CLI (`wrangler`) remains the primary path                                                                      |
| Worker named `10x-astro-starter`                            | Research finding                    | H          | L      | Rename `name` in `wrangler.jsonc` to `gymbuddy` before the first deploy (renaming later creates a new Worker and URL)                |

## Getting Started

Verified against the repo's pinned versions: `wrangler` 4.156.0, `@astrojs/cloudflare` 14.4.3, `astro` 7.4.4. No `wrangler dev` — `astro dev` already runs on workerd.

1. Set `"name": "gymbuddy"` in `wrangler.jsonc`.
2. Authenticate and set production secrets: `npx wrangler login`, then `npx wrangler secret put SUPABASE_URL` and `npx wrangler secret put SUPABASE_KEY` (they are read through `astro:env/server`).
3. First manual deploy: `npm run build && npx wrangler deploy`; check with `npx wrangler tail --format pretty` while opening the `*.workers.dev` URL.
4. Add a deploy job to GitHub Actions on push to `master` using `CLOUDFLARE_API_TOKEN` and `CLOUDFLARE_ACCOUNT_ID` secrets, running build then `npx wrangler deploy`; PR builds use `npx wrangler versions upload --preview-alias pr-<n>`.
5. In Supabase Auth, add the production `workers.dev` URL (and preview hosts) to allowed redirect URLs.

## Out of Scope

The following were not evaluated in this research:

- Docker image configuration
- CI/CD pipeline setup
- Production-scale architecture (multi-region, HA, DR)
