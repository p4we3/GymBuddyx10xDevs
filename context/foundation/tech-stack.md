---
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
---

## Why this stack

A solo developer shipping a mobile-first workout tracker in 3 after-hours weeks (hard deadline 2026-12-06) needs an agent-friendly starter that covers sign-up/sign-in, per-user data isolation and deployment out of the box. The 10x Astro Starter (Astro + React + TypeScript + Supabase + Cloudflare) is the recommended default for a JS web app and passes all four agent-friendly gates. Supabase covers FR-001 auth and, through row-level security, the guardrail that a user's data is visible only to that user. Postgres stores exercise-scoped load history that survives plan edits. Payments, realtime, AI and background jobs are out of scope according to the PRD. The NFR that a set confirmed offline must not be lost is not handled by the starter; it needs a client-side queue such as IndexedDB or a service worker that syncs when the connection returns, and that work must be planned explicitly. Scaffolding confidence is first-class, so occasional manual steps are possible. Deployment uses Cloudflare Pages and CI runs on GitHub Actions with auto-deploy on merge to main.
