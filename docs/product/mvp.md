# MVP — linkbio

**Estado:** definición inicial
**Fecha:** 2026-10-24
**Nombre:** provisional; pendiente de confirmar

## Problema

Los creadores necesitan una página móvil simple para reunir sus links, mostrar
su identidad y compartirla desde una sola URL. Las alternativas existentes
suelen ser pesadas, costosas o poco flexibles para personalizar la imagen y el
tema.

## Propuesta

Una aplicación multiusuario de link-in-bio, mobile-first, donde cada creador
puede registrarse, elegir un `username`, publicar sus links con imágenes y
personalizar su banner y tema. La página pública vive en `/[username]`.

## Público objetivo

Creadores de contenido, freelancers, pequeños negocios y personas que quieren
una página personal de links, principalmente en México.

## Roles

| Rol       | Puede                                                        |
| --------- | ------------------------------------------------------------ |
| Visitante | Ver páginas publicadas y abrir links                         |
| Creador   | Editar su perfil, links, imágenes y tema; publicar su página |
| Admin     | Suspender perfiles y revisar reportes (operación mínima)     |

## Flujo principal

1. El creador se registra o inicia sesión.
2. Elige su `username`.
3. Completa nombre, bio y avatar.
4. Agrega links, les pone título e imagen opcional.
5. Personaliza banner y tema.
6. Previsualiza y publica.
7. Comparte `/[username]`.
8. Un visitante abre la página y hace clic en un link.

## Alcance v1

### Incluido

- Auth con Supabase.
- Onboarding de `username`.
- Perfil: nombre, bio y avatar.
- Links: crear, editar, borrar, reordenar, activar/desactivar.
- Imagen opcional por link.
- Banner de fondo.
- Tema con presets y colores seguros.
- Página pública mobile-first.
- Metadata y Open Graph por perfil.
- Dashboard móvil con navegación inferior.
- Moderación mínima: reporte y suspensión.

### Fuera de alcance

- Donaciones.
- Analíticas de clics.
- Dominios personalizados.
- Planes de pago o suscripciones.
- Múltiples páginas por usuario.
- Equipos o colaboradores.
- i18n.
- Integraciones con redes sociales más allá del enlace.

## Límites y parámetros

| Campo               | Regla                                                                                |
| ------------------- | ------------------------------------------------------------------------------------ |
| `username`          | 3–20 caracteres, `^[a-z0-9_]+$`, sin `_` al inicio/fin, único, reservados bloqueados |
| Nombre visible      | 2–40 caracteres                                                                      |
| Bio                 | 160 caracteres                                                                       |
| Título de link      | 1–60 caracteres                                                                      |
| URL de link         | Solo `http/https`, máximo 2048 caracteres                                            |
| Links por perfil    | 50                                                                                   |
| Avatar              | 400×400, JPEG/PNG/WebP                                                               |
| Banner              | 1600×900, JPEG/PNG/WebP                                                              |
| Thumbnail           | 320×320, JPEG/PNG/WebP                                                               |
| Upload máximo       | 2 MB                                                                                 |
| Formatos prohibidos | SVG, GIF animado y cualquier archivo no raster                                       |
| Página pública      | Una columna, máximo 480 px                                                           |
| Touch target        | >= 44×44 px                                                                          |
| Inputs              | >= 16 px                                                                             |
| Idiomas             | Español                                                                              |
| Zona horaria        | `America/Mexico_City`                                                                |
| Moneda              | MXN                                                                                  |

## Modelo de datos v1

### `profiles`

- `id` uuid, PK, referencia a `auth.users`.
- `username` único.
- `display_name`, `bio`.
- `avatar_path`, `banner_path`.
- `theme` JSONB.
- `is_published`, `is_suspended`.
- `created_at`, `updated_at`.

### `links`

- `id` uuid, PK.
- `profile_id` FK.
- `title`, `url`.
- `thumbnail_path` nullable.
- `position` integer.
- `is_active` boolean.
- `created_at`, `updated_at`.

### `reports`

- `id` uuid, PK.
- `profile_id`, `link_id` nullable.
- `reason`, `status`.
- `created_at`.

### Storage

- `avatars/{user_id}/...`
- `banners/{user_id}/...`
- `link-thumbnails/{user_id}/...`

## Reglas de negocio

- El `username` se guarda en minúsculas y es único.
- Una página solo es visible públicamente si `is_published = true` y
  `is_suspended = false`.
- Solo el dueño puede modificar su perfil, links e imágenes.
- El orden de los links se guarda en `position`; el reordenamiento es atómico.
- Las URLs se validan en servidor; no se permite `javascript:`, `data:` ni
  esquemas distintos de `http/https`.
- Las imágenes se validan por MIME y magic bytes; se elimina EXIF.
- Los reportes no son visibles para el público.

## Métricas de éxito

- Porcentaje de usuarios que publican su página.
- Tiempo desde registro hasta primer link.
- Links promedio por perfil.
- Lighthouse mobile >= 90 en la página pública.
- Tasa de errores de subida de imágenes.
- Reportes por cada 1,000 páginas vistas (calidad/modernización).

## Fases

1. **Fase 0:** reglas, ADRs, workflow y CI.
2. **Fase 1:** scaffold, auth, perfil y onboarding.
3. **Fase 2:** links, imágenes, banner y tema.
4. **Fase 3:** página pública, OG y moderación mínima.
5. **Fase 4:** donaciones a la plataforma y analíticas.

## Decisiones pendientes

- Nombre definitivo.
- Definir dominio de producción.
- Definir cuenta de Supabase y Vercel.

## Riesgos

- Abuso de imágenes o links maliciosos; requiere validación y moderación.
- Colisiones de datos entre `dev` y `qa` mientras compartan staging.
- Crecimiento del alcance antes de validar el MVP.
- Dependencia de un solo proveedor de auth y base de datos.
