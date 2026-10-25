# Git workflow

**Estado:** vigente desde Fase 0
**Modo actual:** un solo desarrollador
**Repo:** GitHub + Vercel + Supabase

## Objetivo

Mantener `main` siempre desplegable, usar `dev` para integración y `qa` como
compuerta de preproducción. Las features viven poco y se integran con PRs
pequeños.

## Ramas

| Rama       | Propósito          | Nace de | Recibe PR de                                                                                            | Deploy                      |
| ---------- | ------------------ | ------- | ------------------------------------------------------------------------------------------------------- | --------------------------- |
| `main`     | Producción         | —       | `qa`, `hotfix/*`                                                                                        | Vercel Production           |
| `qa`       | Preproducción / QA | `dev`   | `dev`, `hotfix/*`, `main` (back-merge)                                                                  | Preview `qa.tudominio.com`  |
| `dev`      | Integración        | `main`  | `feat/*`, `fix/*`, `db/*`, `chore/*`, `docs/*`, `refactor/*`, `test/*`, `hotfix/*`, `main` (back-merge) | Preview `dev.tudominio.com` |
| `feat/*`   | Feature            | `dev`   | —                                                                                                       | Preview efímero             |
| `fix/*`    | Corrección         | `dev`   | —                                                                                                       | Preview efímero             |
| `db/*`     | Migración          | `dev`   | —                                                                                                       | Preview efímero             |
| `chore/*`  | Mantenimiento      | `dev`   | —                                                                                                       | Preview efímero             |
| `hotfix/*` | Urgencia           | `main`  | —                                                                                                       | Production al mergear       |

## Diagrama

```text
feat/* ──PR──► dev ──PR──► qa ──PR──► main
                 ▲                       │
                 └──── back-merge ───────┘
hotfix/* ──PR──► main ──back-merge──► dev ──► qa
```

## Creación inicial del repositorio

```powershell
git init -b main
git add .
git commit -m "chore: initialize phase 0"
git branch dev
git switch dev
git branch qa
git switch main
git push -u origin main dev qa
```

Después de crear las ramas en GitHub, configurar Rulesets sobre `main`, `qa` y
`dev`.

## Flujo diario

```powershell
git switch dev
git pull --ff-only
git switch -c feat/link-thumbnails

# trabajar

git add .
git commit -m "feat(links): add thumbnail upload"
git push -u origin feat/link-thumbnails
gh pr create --base dev --fill
```

Cuando el PR pasa CI y la revisión:

- Merge **Squash** a `dev`.
- Borrar la rama remota.

## Promoción a QA

Cuando `dev` tenga un bloque estable:

```powershell
gh pr create --base qa --head dev --title "chore: promote dev to qa"
```

- Merge con **Merge commit**.
- Vercel despliega el alias de QA.
- Ejecutar E2E y revisión manual en iOS Safari y Android Chrome.
- No mergear si hay bugs P1/P2 abiertos.

## Promoción a producción

```powershell
gh pr create --base main --head qa --title "chore: release v0.1.0"
```

- Merge con **Merge commit**.
- Vercel despliega producción automáticamente.
- Smoke test en producción.
- Crear tag:

```powershell
git switch main
git pull --ff-only
git tag v0.1.0
git push origin v0.1.0
```

## Estrategia de merge

| PR                  | Merge        | Motivo                          |
| ------------------- | ------------ | ------------------------------- |
| `feat/*` → `dev`    | Squash       | Un commit limpio por feature    |
| `fix/*` → `dev`     | Squash       | Historial legible               |
| `db/*` → `dev`      | Squash       | Migración + tipos en un bloque  |
| `dev` → `qa`        | Merge commit | Conserva commits y trazabilidad |
| `qa` → `main`       | Merge commit | La promoción queda registrada   |
| `hotfix/*` → `main` | Merge commit | Se conserva el fix urgente      |

No actives “require linear history” en `qa` y `main`: bloquearía las
promociones con merge commit.

## Reglas de PR

- Título con Conventional Commits.
- Descripción con contexto, cambios, cómo probar y riesgos.
- Capturas mobile a 360 y 390 px si hay UI.
- Un PR debe tener un solo propósito.
- Idealmente menos de 400 líneas de diff.
- CI verde antes de mergear.
- Revisión del subagente `reviewer`; si hay migraciones, también `db-reviewer`.
- No se mergea con conversaciones abiertas.

## Modo solo

GitHub no permite aprobar tu propio PR. Por eso:

- Required approvals: **0**.
- Required checks: `verify`, `db`, `secret-scan`, `branch-policy`.
- El PR sigue siendo obligatorio; no hay push directo.
- La revisión automática la hacen `reviewer` y `db-reviewer`.
- Si el proyecto crece a 2+ personas, subir a 1 approval y agregar CODEOWNERS.

## Hotfix

```powershell
git switch main
git pull --ff-only
git switch -c hotfix/login-redirect
# corregir
git commit -m "fix(auth): preserve redirect after login"
git push -u origin hotfix/login-redirect
gh pr create --base main --fill
```

Después del deploy:

```powershell
git switch dev
git merge --no-ff main
git push origin dev

git switch qa
git merge --no-ff dev
git push origin qa
```

Sin back-merge, el próximo release reincorpora el bug.

## Reglas de commits

Formato:

```text
type(scope): imperative summary
```

Tipos permitidos:

- `feat`
- `fix`
- `db`
- `chore`
- `docs`
- `refactor`
- `test`
- `perf`
- `ci`
- `build`

Ejemplos:

```text
feat(links): add thumbnail upload
fix(auth): handle expired otp
db(profiles): add username unique index
chore(deps): update supabase client
```

## Protecciones de GitHub

Rulesets sobre `main`, `qa` y `dev`:

- Require pull request before merging.
- Required approvals: 0.
- Require status checks: `verify`, `db`, `secret-scan`, `branch-policy`.
- Require branches to be up to date.
- Block force pushes.
- Block deletions.
- Require conversation resolution.
- No linear history requirement.

Configuración del repo:

- Permitir Squash and merge.
- Permitir Merge commit.
- Deshabilitar Rebase and merge.
- Auto-delete head branches.
- Crear tags de release.

## Branch policy Action

`.github/workflows/branch-policy.yml` falla si el origen no corresponde:

```text
main  <- qa | hotfix/*
qa    <- dev | hotfix/* | main
dev   <- feat/* | fix/* | db/* | chore/* | docs/* | refactor/* | test/* | hotfix/* | main
```

Las promociones `main -> dev` y `main -> qa` solo se usan para back-merge
después de un hotfix. No son una vía para saltarse `dev`.

Este check debe ser obligatorio en los Rulesets.

## Limpieza

- Borrar ramas feature después del merge.
- No mantener ramas de features abandonadas por semanas.
- Si una feature crece, dividirla en PRs pequeños.
- Actualizar `dev` con `git pull --ff-only` antes de crear una rama.

## Errores comunes

- **Mergear `dev` directo a `main`:** prohibido; siempre pasa por `qa`.
- **Squash en una promoción:** reescribe la historia y genera divergencias.
- **Olvidar el back-merge después de un hotfix:** el bug vuelve.
- **Migrar producción desde la laptop:** prohibido; usar `migrations.yml`.
- **Mergear sin CI verde:** bloqueado por Rulesets.
