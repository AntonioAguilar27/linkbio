# linkbio

Nombre provisional. Aplicación de link-in-bio mobile-first para creadores.

> Evitar usar "Linktree" como nombre comercial: es una marca registrada.

## Estado

**Fase 0 completada:** reglas, decisiones, workflow de ramas y CI base.
Todavía no hay aplicación Next.js; el scaffold empieza en la Fase 1.

## Stack

| Capa            | Tecnología                              |
| --------------- | --------------------------------------- |
| Frontend        | Next.js 16, React 19, TypeScript strict |
| Estilos         | Tailwind CSS 4                          |
| Backend         | Supabase (Auth, Postgres, Storage, RLS) |
| Hosting         | Vercel                                  |
| Package manager | pnpm                                    |
| Tests           | Vitest, Testing Library, Playwright     |

## Requisitos

- Node.js 24
- pnpm 12+
- Git
- Supabase CLI
- Docker Desktop (opcional; si no, se usa un proyecto Supabase remoto de staging)
- GitHub CLI (recomendado para PRs)
- Vercel CLI (opcional; el deploy se hace por integración con GitHub)

## Setup inicial

```powershell
# pnpm
npm install -g pnpm

# Supabase CLI
npm install -g supabase

# Vercel CLI (opcional)
npm install -g vercel

# GitHub CLI: instalar desde https://cli.github.com/
# y luego autenticar:
gh auth login

# Identidad de Git para los commits
git config --global user.name "Tu Nombre"
git config --global user.email "tu@email.com"
```

Copiar variables de entorno:

```powershell
Copy-Item .env.example .env.local
```

Completar `.env.local` con las credenciales del proyecto Supabase de desarrollo.
Nunca commitear `.env.local`.

## Scripts

Se agregan en la Fase 1:

- `pnpm dev`
- `pnpm lint`
- `pnpm typecheck`
- `pnpm test`
- `pnpm test:e2e`
- `pnpm verify`
- `pnpm db:types`
- `pnpm db:reset`

## Estructura

```text
src/app                 rutas, layouts, metadata
src/features/<domain>   UI, actions, queries, schemas
src/components/ui       primitivas
src/lib/supabase        clientes browser/server
supabase/migrations     migraciones SQL
e2e                     tests Playwright
docs/decisions          ADRs
docs/engineering        workflow, entornos y releases
```

## Flujo de ramas

```text
feat/* ──PR──► dev ──PR──► qa ──PR──► main
                 ▲                       │
                 └──── back-merge ───────┘
hotfix/* ──PR──► main ──back-merge──► dev ──► qa
```

- `main`: producción; Vercel despliega automáticamente.
- `qa`: preproducción; compuerta de QA.
- `dev`: integración.
- Features salen de `dev` y se mergean con squash.
- Las promociones usan merge commit.
- No se hace push directo a `dev`, `qa` ni `main`.

Ver [docs/engineering/git-workflow.md](docs/engineering/git-workflow.md).

## Decisiones pendientes

- Nombre definitivo del proyecto.

## Auth

Email OTP de 6 dígitos + Google OAuth. Ver
[docs/decisions/0002-auth.md](docs/decisions/0002-auth.md).

## Documentación

- [Producto / MVP](docs/product/mvp.md)
- [Git workflow](docs/engineering/git-workflow.md)
- [Entornos](docs/engineering/environments.md)
- [Release checklist](docs/engineering/release-checklist.md)
- [ADR hosting](docs/decisions/0001-hosting.md)
- [ADR auth](docs/decisions/0002-auth.md)
- [ADR pagos fase 2](docs/decisions/0003-payments-phase2.md)
