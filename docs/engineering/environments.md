# Entornos

**Estado:** definición inicial
**Fecha:** 2026-10-24

## Resumen

| Entorno    | Rama       | Vercel                      | Supabase        | Uso             |
| ---------- | ---------- | --------------------------- | --------------- | --------------- |
| Local      | cualquiera | `localhost:3000`            | local o staging | Desarrollo      |
| Dev        | `dev`      | Preview `dev.tudominio.com` | staging         | Integración     |
| QA         | `qa`       | Preview `qa.tudominio.com`  | staging         | QA manual y E2E |
| Producción | `main`     | `tudominio.com`             | producción      | Usuarios reales |

`dev` y `qa` comparten el mismo proyecto Supabase de staging durante el MVP.
Producción usa un proyecto separado.

## Local

### Opción A: Supabase local con Docker

```powershell
supabase start
supabase db reset
pnpm dev
```

Ventajas: aislamiento total, migraciones y RLS reproducibles.

### Opción B: Supabase remoto de staging

Si no hay Docker, usar un proyecto Supabase de desarrollo/staging y apuntar
`.env.local` a ese proyecto. No usar credenciales de producción.

## Proyectos Supabase

| Proyecto          | Uso        | Credenciales                            |
| ----------------- | ---------- | --------------------------------------- |
| `linkbio-staging` | dev + qa   | Vercel Preview / GitHub `staging`       |
| `linkbio-prod`    | producción | Vercel Production / GitHub `production` |

Cada proyecto tiene su propia URL, publishable key y secret key. Las buckets de
Storage y las políticas RLS se crean por migración.

## Vercel

- **Production Branch:** `main`.
- `dev` y `qa` son Preview deployments con alias estables.
- Los previews deben estar protegidos con Vercel Authentication o contraseña.
- Los feature branches generan previews efímeros.

### Variables por entorno

Vercel tiene tres entornos integrados: Production, Preview y Development.
**Todas las ramas de preview comparten las variables de Preview.** Como `dev` y
`qa` comparten staging, esto no es un problema en el MVP.

Si más adelante `qa` necesita un Supabase distinto:

1. Vercel Custom Environments (plan Pro), o
2. proyectos Vercel separados, o
3. deploy por GitHub Actions con secretos por entorno.

## Variables de entorno

```dotenv
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=
SUPABASE_SECRET_KEY=
NEXT_PUBLIC_SITE_URL=http://localhost:3000
```

Reglas:

- `NEXT_PUBLIC_*` puede llegar al navegador.
- `SUPABASE_SECRET_KEY` es solo servidor; nunca en componentes cliente.
- `.env.local` no se commitea.
- `.env.example` es el único archivo de entorno versionado.
- Las variables de producción solo existen en Vercel Production y GitHub
  Environment `production`.

## Auth redirect URLs

Configurar en Supabase Auth:

```text
http://localhost:3000/**
https://dev.tudominio.com/**
https://qa.tudominio.com/**
https://tudominio.com/**
```

Para Google OAuth, agregar el callback de Supabase en Google Cloud:

```text
https://<project-ref>.supabase.co/auth/v1/callback
```

## Migraciones

### Local

```powershell
supabase migration new add_profiles
supabase db reset
pnpm db:types
supabase test db
```

### Staging

Se aplican con el workflow `Migrations` o desde un entorno autorizado. Nunca
con credenciales de producción.

### Producción

1. Probar la migración en staging.
2. Hacer backup si el cambio es destructivo.
3. Ejecutar `Migrations` con environment `production`.
4. Confirmar el input de confirmación.
5. Mergear `qa` a `main` y verificar el deploy.

## Expand-contract

Para cambios destructivos:

1. **Expand:** agregar columnas/tablas nuevas sin romper el código actual.
2. **Migrate:** desplegar código que escribe en ambos formatos si hace falta.
3. **Contract:** eliminar lo viejo en una migración posterior.

Así el orden entre migración y deploy no es crítico.

## Secretos

- No se leen `.env` desde el agente sin aprobación.
- No se pegan claves en el chat.
- No se guardan claves en `opencode.jsonc`.
- GitHub Environments `staging` y `production` tienen secretos separados.
- Producción requiere confirmación manual en el workflow de migraciones.

### Secrets de GitHub para migraciones

Crear un secret `SUPABASE_DB_URL` en cada GitHub Environment:

| Environment  | Secret                                    |
| ------------ | ----------------------------------------- |
| `staging`    | `SUPABASE_DB_URL` del proyecto staging    |
| `production` | `SUPABASE_DB_URL` del proyecto producción |

Formato:

```text
postgresql://postgres.<project-ref>:<password>@<host>:5432/postgres?sslmode=require
```

- Usar conexión directa (puerto 5432), no el pooler, para migraciones.
- URL-encodear la contraseña si tiene caracteres especiales.
- El workflow `Migrations` pide escribir el nombre del entorno como confirmación.
- El environment `production` puede configurarse con required reviewers en GitHub.

## Dominios

| Dominio             | Destino                      |
| ------------------- | ---------------------------- |
| `tudominio.com`     | Vercel Production            |
| `www.tudominio.com` | redirect a apex o Production |
| `dev.tudominio.com` | rama `dev`                   |
| `qa.tudominio.com`  | rama `qa`                    |

## Pagos (Fase 2)

Variables adicionales, solo cuando exista código de pagos:

```dotenv
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=
PAYPAL_CLIENT_ID=
PAYPAL_CLIENT_SECRET=
PAYPAL_WEBHOOK_ID=
```

En staging se usan claves de test/sandbox. En producción, claves live.
Nunca mezclar claves de test con la base de producción.
