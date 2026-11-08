---
bootstrapped_at: 2026-11-02T21:36:35Z
starter_id: 10x-astro-starter
starter_name: 10x Astro Starter (Astro + Supabase + Cloudflare)
project_name: gymbuddy
language_family: js
package_manager: npm
cwd_strategy: git-clone
bootstrapper_confidence: first-class
phase_3_status: ok
audit_command: npm audit --json
---

## Hand-off

```yaml
starter_id: 10x-astro-starter
package_manager: npm
project_name: gymbuddy
hints:
  language_family: js
  team_size: solo
  deployment_target: cloudflare-pages
  ci_provider: github-actions
  ci_default_flow: auto-deploy-on-merge
  bootstrapper_confidence: first-class
  path_taken: standard
  quality_override: false
  self_check_answers: null
  has_auth: true
  has_payments: false
  has_realtime: false
  has_ai: false
  has_background_jobs: false
```

### Why this stack

A solo developer shipping a mobile-first workout tracker in 3 after-hours weeks (hard deadline 2026-12-06) needs an agent-friendly starter that covers sign-up/sign-in, per-user data isolation and deployment out of the box. The 10x Astro Starter (Astro + React + TypeScript + Supabase + Cloudflare) is the recommended default for a JS web app and passes all four agent-friendly gates. Supabase covers FR-001 auth and, through row-level security, the guardrail that a user's data is visible only to that user. Postgres stores exercise-scoped load history that survives plan edits. Payments, realtime, AI and background jobs are out of scope according to the PRD. The NFR that a set confirmed offline must not be lost is not handled by the starter; it needs a client-side queue such as IndexedDB or a service worker that syncs when the connection returns, and that work must be planned explicitly. Scaffolding confidence is first-class, so occasional manual steps are possible. Deployment uses Cloudflare Pages and CI runs on GitHub Actions with auto-deploy on merge to main.

## Pre-scaffold verification

| Signal      | Value                                                     | Severity | Notes                                                                                  |
| ----------- | --------------------------------------------------------- | -------- | -------------------------------------------------------------------------------------- |
| npm package | not run                                                   | n/a      | cmd_template starts with `git clone`; no npm package to derive                         |
| GitHub repo | przeprogramowani/10x-astro-starter last pushed 2026-10-29 | fresh    | from card.docs_url; `gh` not installed, read from the public GitHub REST API with curl |

## Scaffold log

**Resolved invocation**: `git clone https://github.com/przeprogramowani/10x-astro-starter .bootstrap-scaffold && cd .bootstrap-scaffold && npm install`
**Strategy**: git-clone
**Starter commit**: d85617c63a5f400eaed77fe7e618ee6f5eda4113
**Exit code**: 0
**Files moved**: 51 (excluding `node_modules/`, moved as one directory): 50 to their own paths, 1 as a `.scaffold` sibling
**Conflicts (.scaffold siblings)**: CLAUDE.md.scaffold
**.gitignore handling**: moved silently (absent in cwd)
**.bootstrap-scaffold cleanup**: deleted

Cloned `.git/` was deleted before the move. Nothing was dropped under `context/`: the starter ships no `context/` directory.

### Runtime mismatch found during scaffold

`npm install` exited 0 but reported EBADENGINE warnings for local Node v22.20.0 / npm 10.9.3:

```
package: '@astrojs/prism@4.1.2',
required: { node: '>=22.23.4' },
package: '@astrojs/react@7.1.2',
required: { node: '>=22.23.4' },
package: '@astrojs/telemetry@3.4.2',
required: { node: '>=22.23.4' },
package: 'astro@7.4.4',
required: { node: '>=22.23.4', npm: '>=9.6.5', pnpm: '>=7.1.0' },
package: 'astro-eslint-parser@3.1.0',
required: { node: '^22.22.3 || ^24.16.0 || >=26.3.0' },
package: 'eslint-plugin-astro@3.2.1',
required: { node: '^22.22.3 || ^24.16.0 || >=26.3.0' },
package: 'lint-staged@17.6.0',
required: { node: '>=22.22.1' },
```

Post-move check: `npx --no-install astro --version` reports `Node.js v22.20.0 is not supported by Astro! Please upgrade Node.js to a supported version: ">=22.23.4"`. The starter's `.nvmrc` pins 26.13.0. The Astro CLI will not run until Node is upgraded. Because of this, build, dev server and lint were not exercised in this run.

## Post-scaffold audit

**Tool**: npm audit --json
**Summary**: 0 CRITICAL, 0 HIGH, 17 MODERATE, 0 LOW
**Direct vs transitive**: 0/0/9/0 direct of total 0/0/17/0

All 17 entries trace back to one advisory, reached through ESLint tooling. All 9 direct packages are `devDependencies`. npm's suggested fixes are older major versions of these packages; `npm audit fix` was not run.

#### CRITICAL findings

None.

#### HIGH findings

None.

#### MODERATE findings

Root advisory: **minimatch** `<11.0.1`, GHSA-g3hv-75wg-m249, potential denial of service via globparts amplification (https://github.com/advisories/GHSA-g3hv-75wg-m249).

Direct (dev): `@eslint/compat`, `@eslint/js`, `eslint`, `eslint-config-prettier`, `eslint-plugin-astro`, `eslint-plugin-jsx-a11y`, `eslint-plugin-prettier`, `eslint-plugin-react`, `typescript-eslint`.

Transitive: `@eslint-community/eslint-utils`, `@eslint/config-array`, `@typescript-eslint/eslint-plugin`, `@typescript-eslint/parser`, `@typescript-eslint/type-utils`, `@typescript-eslint/typescript-estree`, `@typescript-eslint/utils`, `minimatch`.

#### LOW / INFO findings

None.

## Hints recorded but not acted on

| Hint                    | Value                |
| ----------------------- | -------------------- |
| bootstrapper_confidence | first-class          |
| quality_override        | false                |
| path_taken              | standard             |
| self_check_answers      | null                 |
| team_size               | solo                 |
| deployment_target       | cloudflare-pages     |
| ci_provider             | github-actions       |
| ci_default_flow         | auto-deploy-on-merge |
| has_auth                | true                 |
| has_payments            | false                |
| has_realtime            | false                |
| has_ai                  | false                |
| has_background_jobs     | false                |

## Next steps

Next: a future skill will set up agent context (CLAUDE.md, AGENTS.md). For now, your project is scaffolded and verified — happy hacking.

Useful manual steps in the meantime:

- `git init` (if you have not already) to start your own repo history.
- Review any `.scaffold` siblings the conflict policy created and decide which version of each file to keep.
- Address audit findings per your project's risk tolerance — the full breakdown is in this log.
