# ADR 0003: Pagos y donaciones (Fase 2)

**Estado:** Diferido — Fase 2
**Fecha:** 2026-10-24

## Contexto

El producto contempla donaciones, pero se decidió dejarlas fuera del MVP para
validar primero el núcleo: perfil, links, imágenes y tema.

Cuando se implementen, serán **donaciones solo a la plataforma**, no pagos a
cada creador. Esto evita un marketplace con onboarding, KYC, payouts, impuestos
y contracargos por terceros.

## Decisión

- **Modelo:** donación única a la plataforma.
- **Mercado:** México.
- **Moneda:** MXN.
- **Proveedor principal:** Stripe (tarjeta).
- **Google Pay:** vía Stripe Payment Element / Express Checkout, si la cuenta y
  el comprador cumplen los requisitos.
- **Proveedor alternativo:** PayPal.
- **Sin suscripciones** en la primera versión.
- **Sin payouts** a creadores.

## Arquitectura prevista

1. El usuario elige un monto en MXN.
2. El servidor crea la sesión o intención de pago con el monto validado.
3. El cliente completa el pago con Stripe o PayPal.
4. El proveedor envía un webhook firmado.
5. El servidor verifica la firma y registra la donación de forma idempotente.
6. Se muestra una página de gracias.

### Tabla `donations`

- `id` uuid.
- `provider` (`stripe` | `paypal`).
- `provider_order_id` único.
- `amount_cents` integer.
- `currency` (`MXN`).
- `status` (`pending` | `paid` | `failed` | `refunded`).
- `created_at`, `updated_at`.

## Seguridad

- El monto se calcula y valida en servidor; nunca se confía en el cliente.
- Webhooks con verificación de firma.
- Idempotencia por `provider_order_id`.
- No se almacenan datos de tarjeta.
- Las claves secretas solo existen en el servidor.
- Rate limiting en la creación de pagos.
- Logs sin datos personales innecesarios.

## Legal y fiscal

- Las donaciones son ingreso para la plataforma.
- Puede aplicar IVA/CFDI según el régimen fiscal.
- Hace falta aviso de privacidad y términos del servicio.
- Conviene consultar a un contador antes de lanzar la Fase 2.
- No se debe prometer deducibilidad fiscal sin validarlo.

## Fuera de alcance

- Donaciones a creadores.
- Suscripciones.
- Facturación automática.
- Reembolsos automáticos.
- Multi-moneda.

## Seguimiento

- Crear cuenta Stripe México.
- Validar Google Pay en sandbox.
- Definir flujo de reembolsos y soporte.
- Crear `40-payments.md` cuando exista código de pagos.
