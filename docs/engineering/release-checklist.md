# Release checklist

**Estado:** vigente desde Fase 0

## Antes de abrir un PR

- [ ] La rama sale de `dev` y tiene nombre `feat/`, `fix/`, `db/`, `chore/`,
      `docs/`, `refactor/` o `test/`.
- [ ] `pnpm verify` pasa localmente.
- [ ] Si hay migración: `supabase db reset`, `pnpm db:types` y `supabase test db`.
- [ ] Si hay UI: revisada a 360 px y 390 px.
- [ ] Si hay imágenes: MIME, tamaño y dimensiones validados.
- [ ] Si hay RLS: probado acceso cruzado entre usuarios.
- [ ] No hay secretos ni `.env` en el diff.
- [ ] El PR tiene contexto, cómo probar y riesgos.

## Feature PR → dev

- [ ] CI verde: `verify`, `db`, `secret-scan`, `branch-policy`.
- [ ] Revisión del subagente `reviewer`.
- [ ] Si hay migración, revisión de `db-reviewer`.
- [ ] Sin conversaciones abiertas.
- [ ] Merge con **Squash**.
- [ ] Borrar la rama.

## Promoción dev → qa

- [ ] `dev` está estable y con CI verde.
- [ ] PR `dev` → `qa` con merge commit.
- [ ] Vercel desplegó el alias de QA.
- [ ] Smoke test: registro, login, crear link, subir imagen, publicar, abrir
      página pública.
- [ ] Revisión manual en iOS Safari y Android Chrome.
- [ ] Lighthouse mobile >= 90 en la página pública.
- [ ] Sin bugs P1/P2 abiertos.
- [ ] Migración probada en staging.
- [ ] Datos de QA sembrados o reseteados.

## Producción qa → main

- [ ] QA aprobado.
- [ ] Variables de entorno de producción verificadas.
- [ ] Si hay migración obligatoria, aplicada a producción antes del merge.
- [ ] Backup de producción si el cambio es destructivo.
- [ ] PR `qa` → `main` con merge commit.
- [ ] Vercel Production desplegó correctamente.
- [ ] Smoke test en producción.
- [ ] Tag `vX.Y.Z`.
- [ ] Revisar logs de Vercel y Supabase.
- [ ] Back-merge si hubo hotfix.

## Hotfix

- [ ] Rama `hotfix/*` desde `main`.
- [ ] Cambio mínimo y test que reproduce el bug.
- [ ] PR directo a `main`.
- [ ] CI verde y revisión.
- [ ] Deploy y smoke test.
- [ ] Back-merge `main` → `dev`.
- [ ] Back-merge `dev` → `qa`.
- [ ] Tag de patch.

## Rollback

### Aplicación

- Usar “Instant Rollback” de Vercel al deployment anterior.
- Verificar que la migración aplicada siga siendo compatible.

### Base de datos

- Preferir forward-fix; no revertir migraciones a ciegas.
- Si es imprescindible, restaurar backup o PITR.
- Documentar el incidente y la causa.

## Post-release

- [ ] Monitorear errores 5xx y latencia.
- [ ] Revisar logs de Auth y Storage.
- [ ] Confirmar que los links públicos cargan y que las imágenes se ven.
- [ ] Actualizar CHANGELOG o notas de release si existe.
- [ ] Cerrar el milestone.
