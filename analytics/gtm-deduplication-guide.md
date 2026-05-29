# Guía para coordinar la deduplicación de eventos de GTM con AFRUS

**Audiencia:** equipos técnicos de clientes (ad-ops, integradores) que configuran sus propias tags de Google Tag Manager y necesitan que esas tags no dupliquen eventos contra los que AFRUS ya envía por Meta Conversions API (CAPI) y Google Analytics 4 Measurement Protocol.

**Última actualización:** 2026-05-29

---

## Por qué esta guía existe

AFRUS dispara eventos a Meta y a GA4 directamente desde su backend (server-side) y desde el navegador del widget de donación/registro. Si tu landing tiene además un contenedor de GTM con tus propias tags de Meta Pixel o GA4 configuradas para disparar eventos de conversión (`Lead`, `Purchase`, etc.), Meta y GA4 contarán esos eventos **como adicionales** a los que AFRUS envía — inflando tus conversiones reportadas y degradando la calidad de atribución.

La solución estándar de Meta y GA4 es la **deduplicación por clave compartida**. AFRUS ya empuja todas las claves necesarias al `window.dataLayer` del navegador. Tu trabajo es configurar tus tags de GTM para leer esas claves y pasarlas a Meta/GA4 con el formato correcto.

Esta guía documenta exactamente qué claves están disponibles en `dataLayer`, qué evento las dispara, y cómo mapearlas en tu tag de GTM.

---

## Por qué hacer esta configuración (beneficio cuantificable)

Configurar tus tags de GTM para deduplicar contra los eventos AFRUS no es solo "buena práctica" — Meta documenta el beneficio en cifras concretas:

- **Hasta -54.9% en cost per result** para anunciantes que alcanzan 75% de cobertura de Event ID entre Pixel y Conversions API ([fuente oficial Meta](https://developers.facebook.com/documentation/ads-commerce/conversions-api/deduplicate-pixel-and-server-events))
- **Event Match Quality (EMQ) de Meta** mejora típicamente de ~4-5/10 a ~7-8/10 con deduplicación bien configurada
- **Cobertura de Event ID** sube de 0% a **75-100%** en los eventos donde Pixel y CAPI ambos disparan con claves correctas

Sin la configuración de deduplicación, tus campañas optimizan sobre conversiones dobles e infladas — gastas más por menos resultados reales. Con la configuración correcta (que esta guía te ayuda a hacer), Meta cuenta las conversiones una sola vez con señales combinadas server-side + browser-side, lo que mejora la atribución y la optimización automática de las campañas.

---

## Cómo Meta deduplica (resumen)

- **Clave primaria:** `(event_name, event_id)` deben coincidir entre el evento del navegador (Pixel) y el evento del servidor (CAPI), dentro de una ventana de 48 horas.
- Si coinciden → Meta los cuenta como **una sola conversión** con la información combinada de ambos canales.
- Si no coinciden → Meta los cuenta como **dos conversiones distintas**.
- Documentación oficial: https://developers.facebook.com/documentation/ads-commerce/conversions-api/deduplicate-pixel-and-server-events

## Cómo GA4 deduplica (resumen)

- **Solo eventos `purchase`** se deduplican automáticamente, y la clave es `transaction_id`.
- Otros eventos (`view_item`, `add_to_cart`, etc.) **no tienen mecanismo nativo de deduplicación** en GA4.
- Documentación oficial: https://support.google.com/analytics/answer/12313109

---

## Claves disponibles en `dataLayer` (por evento)

Cada vez que el widget AFRUS dispara un evento, hace un `dataLayer.push({...})` con un objeto que incluye los siguientes campos. Tu tag de GTM puede leerlos vía **Variables → Data Layer Variable** en GTM.

### Evento `start` (corresponde a ViewContent de Meta / view_item de GA4)

| Campo en `dataLayer` | Tipo | Para qué usarlo |
|----------------------|------|-----------------|
| `event` | string `'start'` | Identificar el evento en el Trigger de GTM |
| `goal` | string | Tipo de widget (`donation`, `register`, `signup`) |
| `pixel_id` | string | Pixel de Meta configurado en el widget (informativo) |
| `lead_id` | null en este punto | No disponible aún — el lead no existe |

> No se puede deduplicar este evento por `event_id` porque AFRUS genera un UUID por disparo en el lado servidor que no se expone al navegador. Para `view_item` (GA4) tampoco existe deduplicación nativa, así que no hay riesgo de doble conteo automático — Meta y GA4 tratan a cada evento independientemente.

### Evento `add_to_cart` (AddToCart / add_to_cart)

| Campo en `dataLayer` | Tipo | Uso |
|----------------------|------|-----|
| `event` | `'add_to_cart'` | Trigger |
| `ecommerce.items[0].item_id` | string (= ID del widget) | Identifica el widget |
| `ecommerce.items[0].price` | number | Monto seleccionado (si ya está fijado) |
| `ecommerce.currency` | string | Moneda |

### Evento `begin_checkout` (InitiateCheckout / begin_checkout)

Mismo conjunto de campos que `add_to_cart`, más:

| Campo en `dataLayer` | Tipo | Uso |
|----------------------|------|-----|
| `ecommerce.items[0].item_category` | string | Frecuencia (`once`, `monthly`) |

### Evento `add_payment_info` (AddPaymentInfo / add_payment_info)

Mismo conjunto que `begin_checkout`, más:

| Campo en `dataLayer` | Tipo | Uso |
|----------------------|------|-----|
| `ecommerce.payment_type` | string | Método de pago (`card`, `bank_transfer`, etc.) |

### Evento `generate_lead` (Lead / generate_lead) — **conversión clave**

| Campo en `dataLayer` | Tipo | Uso para deduplicación |
|----------------------|------|------------------------|
| `event` | `'generate_lead'` | Trigger |
| `ecommerce.lead_id` | string o number | **Úsalo como `eventID` en tu tag de Meta Pixel para deduplicar con AFRUS CAPI** |
| `goal` | string | Contexto del lead |

**Configuración recomendada de Meta Pixel tag en GTM:**

```
Event Name: Lead
Event Parameters:
  eventID: {{DLV - ecommerce.lead_id}}   ← clave de deduplicación
```

Donde `{{DLV - ecommerce.lead_id}}` es una variable Data Layer Variable apuntando a `ecommerce.lead_id`.

### Evento `purchase` (Purchase / purchase) — **conversión clave**

| Campo en `dataLayer` | Tipo | Uso para deduplicación |
|----------------------|------|------------------------|
| `event` | `'purchase'` | Trigger |
| `ecommerce.transaction_id` | string | Referencia del gateway (uso interno) |
| `ecommerce.transaction_gateway_code` | string | **Úsalo como `eventID` en tu tag de Meta Pixel para deduplicar con AFRUS CAPI** |
| `ecommerce.transaction_gateway_code` | string | **Úsalo también como `transaction_id` en tu tag de GA4 para deduplicar la compra** |
| `ecommerce.lead_id` | string o number | Identifica al donante |
| `ecommerce.items[0].price` | number | Monto donado |
| `ecommerce.currency` | string | Moneda |

> **Importante:** AFRUS usa `transaction_gateway_code` (la referencia que devuelve la pasarela de pago) como `event_id` para deduplicar con su evento de CAPI. Si tu tag de GTM usa `transaction_id` (el ID interno de AFRUS) en lugar de `transaction_gateway_code`, NO deduplicará con AFRUS — terminarás con doble conteo. Asegúrate de mapear el campo correcto.

**Configuración recomendada de Meta Pixel tag en GTM:**

```
Event Name: Purchase
Event Parameters:
  eventID: {{DLV - ecommerce.transaction_gateway_code}}   ← deduplicación con CAPI
  value: {{DLV - ecommerce.items.0.price}}
  currency: {{DLV - ecommerce.currency}}
```

**Configuración recomendada de GA4 tag en GTM:**

```
Event Name: purchase
Event Parameters:
  transaction_id: {{DLV - ecommerce.transaction_gateway_code}}   ← deduplicación GA4
  value: {{DLV - ecommerce.items.0.price}}
  currency: {{DLV - ecommerce.currency}}
```

### Evento `register` (CompleteRegistration o Lead / sign_up)

> **Nota:** el evento `register` se dispara en el widget pero NO se hace `dataLayer.push` para él. Si necesitas que tu tag de GTM reaccione al registro, usa el evento `generate_lead` que sí se publica.

---

## Resumen de mapeo por conversión

| Conversión AFRUS | Campo dataLayer a usar como `eventID` (Meta Pixel) | Campo dataLayer a usar como `transaction_id` (GA4 Purchase) |
|------------------|----------------------------------------------------|------------------------------------------------------------|
| Lead / Registro (cualquier goal) | `ecommerce.lead_id` | N/A (GA4 no deduplica `sign_up` / `generate_lead`) |
| Compra (`donation` goal) | `ecommerce.transaction_gateway_code` | `ecommerce.transaction_gateway_code` |

---

## Consideraciones importantes

1. **El widget AFRUS ya envía estos eventos por su cuenta.** Tu tag de GTM duplicaría el envío si no usa la deduplicación. Esto es opcional — si confías solo en lo que AFRUS envía, puedes desactivar tus tags de conversión en GTM y ahorrarte la configuración.

2. **Los eventos de funnel (`add_to_cart`, `begin_checkout`, `add_payment_info`) no tienen claves de deduplicación cross-canal en este momento.** AFRUS genera UUIDs por disparo en el navegador y los envía al servidor, pero esos UUIDs no se exponen al `dataLayer`. Si necesitas que tus tags de GTM disparen estos eventos sin duplicar contra los de AFRUS, contacta al equipo de AFRUS para coordinar una mejora.

3. **PII (email, teléfono, nombre):** AFRUS **excluye** información personal del `dataLayer` por privacidad. Los campos `lead` o `customer_data` que tu tag necesite para Advanced Matching los tendrá que obtener de tu propia recolección de datos en el formulario o servicios complementarios. AFRUS los envía hasheados directamente a Meta CAPI sin pasar por el navegador.

4. **El `dataLayer` push ocurre síncronamente en el momento del evento.** Si tu tag de GTM usa un trigger de tipo `Custom Event` con el nombre exacto (`start`, `add_to_cart`, `purchase`, etc.), se disparará correctamente. No requiere timers ni esperas.

---

## Cómo verificar que la deduplicación funciona

1. **Meta Events Manager → Test Events tab:** dispara un evento de conversión desde un widget AFRUS configurado. Deberías ver una sola fila para el evento (no dos), con la columna `Event ID` poblada con el valor de `lead_id` o `transaction_gateway_code` correspondiente.

2. **Meta Events Manager → Event Deduplication tab** para el evento `Lead` o `Purchase`: el porcentaje de `Event ID coverage` debería superar el 75% (umbral recomendado por Meta).

3. **GA4 DebugView:** dispara un evento `purchase` y verifica que aparezca con el campo `transaction_id` poblado con el valor de `transaction_gateway_code`. Si dos eventos tienen el mismo `transaction_id`, GA4 los dedupea automáticamente.

4. **Panel AFRUS de eventos analíticos** (`/processes/analytics-events`): confirma que el evento se registró del lado servidor. La columna `description` contendrá `ANALYTIC-EVENT-FACEBOOK-LEAD` o `ANALYTIC-EVENT-FACEBOOK-PURCHASE`, y el `event_id` en el payload debe coincidir con el `eventID` que envía tu tag de GTM.

---

## Troubleshooting

Si tu configuración no deduplica correctamente después de seguir esta guía:

1. Verifica que el evento llega tanto desde AFRUS (panel de eventos analíticos) como desde tu GTM (Meta Test Events tab, fila marcada como `Browser`).
2. Confirma que el `event_id` (lado AFRUS) coincide **exactamente** con el `eventID` (lado tu GTM) — no debe haber espacios, ni transformaciones de tipo `number`-a-`string` que cambien el valor.
3. Confirma que ambos eventos usan el mismo `event_name` (Meta es case-sensitive: `Lead` ≠ `lead`).
4. Contacta al equipo AFRUS con un ejemplo concreto (URL de la página, pixel ID, valores de `event_id` y `eventID` de ambos lados).
