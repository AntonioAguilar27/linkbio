---
description: Plan and implement a feature or fix
---

Plan and implement: $ARGUMENTS

1. Read `AGENTS.md` and the relevant ADRs.
2. Inspect the current code before proposing changes.
3. State the plan, files to touch, and risks.
4. Implement the smallest correct change.
5. If the database changes, create a migration with RLS, indexes, and regenerated types.
6. Add or update tests.
7. Run `/verify`.
8. Report the summary, commands run, and remaining risks.

Do not commit, push, deploy, or apply remote migrations.
