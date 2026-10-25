---
name: Supabase Migration
description: Create and verify a Supabase migration with RLS, indexes, and generated types
---

Use this skill whenever a change requires a database migration.

## Workflow

1. Read `AGENTS.md` and `docs/engineering/environments.md`.
2. Create the migration: `supabase migration new <snake_case_name>`.
3. Write the SQL:
   - tables, columns, constraints, defaults
   - indexes for foreign keys and query paths
   - `alter table ... enable row level security`
   - policies for select, insert, update, and delete
   - triggers or functions with a safe `search_path`
   - storage buckets and policies if needed
4. Reset the local database: `supabase db reset`.
5. Regenerate types: `pnpm db:types`.
6. Add RLS tests under `supabase/tests/`.
7. Run `supabase test db` and `pnpm verify`.
8. Ask the `db-reviewer` subagent to review the migration.

## Rules

- Never edit an applied migration.
- Never run `supabase db push` against production from a local machine.
- Prefer expand-contract for destructive changes.
- Every new table gets RLS in the same migration.
- Every policy must be tested for cross-tenant access.
- Commit the migration and the regenerated types together.
