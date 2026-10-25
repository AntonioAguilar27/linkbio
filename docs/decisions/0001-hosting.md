# ADR 0001: Hosting del frontend y backend

**Estado:** Aceptado
**Fecha:** 2026-10-24

## Contexto

El producto es una aplicación Next.js con Supabase como base de datos, auth y
storage. Necesitamos SSR/ISR para la página pública, metadata dinámica, OG
images y una experiencia mobile-first rápida.

## Decisión

- **Frontend:** Next.js 16 en Vercel.
- **Backend:** Supabase (Postgres, Auth, Storage, RLS).
- **Producción:** Vercel Production despliega desde `main`.
- **Preproducción:** `qa` y `dev` usan previews de Vercel.
- **Staging:** un proyecto Supabase compartido por `dev` y `qa`.
- **Producción:** un proyecto Supabase separado.

## Consecuencias

- Vercel simplifica SSR, ISR, previews y rollback.
- Supabase sigue siendo la fuente de verdad de datos y auth.
- Las variables de entorno se separan por entorno.
- La página pública debe ser cacheable y no debe refrescar cookies de sesión.
- El refresco de sesión de Supabase vive en `proxy.ts` y usa `getClaims()`.
- Las migraciones se aplican con un workflow y nunca desde una laptop a
  producción.

## Alternativas consideradas

### Export estático alojado en Supabase Storage

Descartado: pierde SSR/ISR, metadata dinámica y OG images; además obliga a
duplicar lógica de auth en el cliente.

### Cloudflare Workers/Pages

Viable, pero con más configuración para Next.js y Supabase SSR. No aporta una
ventaja clara para el MVP.

### VPS propio

Descartado por costo operativo: SSL, CI, escalado, logs y rollback quedan a
cargo del equipo.

## Seguimiento

- Configurar dominio y aliases de `dev` y `qa`.
- Definir proyecto Supabase de staging y producción.
- Documentar variables de entorno en `docs/engineering/environments.md`.
