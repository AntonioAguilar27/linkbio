# linkbio (working name)

## Product

Multi-tenant link-in-bio app for creators. Spanish UI, Mexico, MXN.

MVP: auth, profile, links with images, banner/theme, public page.

Out of scope: donations, analytics, custom domains, paid plans.

Donations are platform-only and belong to Phase 2.

## Stack

Next.js 16 App Router, React 19, TypeScript strict, Tailwind CSS 4,
Supabase (Auth, Postgres, Storage, RLS), Vercel, pnpm.

## Commands

- `pnpm dev`
- `pnpm lint`
- `pnpm typecheck`
- `pnpm test`
- `pnpm test:e2e`
- `pnpm verify` (lint + typecheck + test)
- `pnpm db:types` after migrations
- `pnpm db:reset` local only

## Structure

- `src/app`: routes, layouts, metadata
- `src/features/<domain>`: UI, actions, queries, schemas
- `src/components/ui`: primitives
- `src/lib/supabase`: browser/server clients
- `supabase/migrations`: SQL migrations
- `e2e`: Playwright tests
- `docs/decisions`: ADRs
- `docs/engineering`: workflow and environment docs

## Architecture

- Server Components by default; `"use client"` only at leaves.
- Mutations via Server Actions or Route Handlers, validated with Zod.
- No database queries inside components.
- Public pages are cacheable; auth and dashboard routes are `no-store`.
- Supabase session refresh lives in `proxy.ts`; use `getClaims()`.
- Never trust client-provided amounts, ownership, or published state.

## Data and security

- RLS enabled on every table. Never expose secret/service-role keys.
- Public read only for published profiles and links; owners write their own rows.
- Storage paths scoped to `auth.uid()`; validate MIME and magic bytes.
- Link URLs: http/https only, max 2048, `rel="noopener noreferrer nofollow"`.
- Images: JPEG/PNG/WebP, max 2 MB, no SVG.
- No secrets in the repo. `.env.example` is the only env file committed.

## Mobile-first

- Base 360px; test 360, 390 and 768.
- Touch targets >= 44px; inputs >= 16px; safe-area insets; `100dvh`.
- No hover-only interactions; visible focus; respect reduced motion.
- Public page max-width 480px, single column.
- WCAG 2.2 AA, contrast 4.5:1.

## Git workflow

- Branch from `dev`: `feat/`, `fix/`, `db/`, `chore/`, `docs/`, `refactor/`, `test/`.
- Promotion: feature -> dev -> qa -> main. Hotfix starts from `main`.
- After a hotfix, back-merge `main -> dev -> qa`.
- Never push directly to `dev`, `qa`, or `main`; never force-push them.
- Squash feature PRs; use merge commits for `dev -> qa` and `qa -> main`.
- Conventional Commits. Run `pnpm verify` before opening a PR.
- Read `docs/engineering/git-workflow.md` for the full process.

## Definition of Done

`pnpm verify` passes; migration + regenerated types + RLS test;
checked at 360/390/768; no new warnings; ADR updated if a decision changed.

## Forbidden

- `git push`, production deploy, or `supabase db push` without approval.
- Editing applied migrations or generated database types by hand.
- Disabling RLS, ESLint, or TypeScript to make code compile.
- Adding dependencies without justification.
