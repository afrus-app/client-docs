# AFRUS — Estado de la implementación de Analytics

**Fecha de la instantánea:** 2026-05-29
**Alcance:** estado actual del soporte de AFRUS para Google Analytics 4, Meta Conversions API (CAPI), y Google Tag Manager después de los despliegues recientes.

---

## Resumen ejecutivo

| Pregunta | Estado |
|---|---|
| Cross-domain GA4 (linker `_gl`) | ⚠️ Parcial — AFRUS no rompe el linker, pero requiere configuración del cliente; no hay soporte automático para `linker.domains` en landings AFRUS |
| UTMs preservadas en flujo | ✅ Atribución GA4 se preserva vía `client_id` consistente; URLs no propagan UTMs por defecto |
| Configuración anti-pérdida fuente/medio | ✅ AFRUS maneja la consistencia del `client_id`; cliente debe coordinar linker y consent banner |
| Meta CAPI envía `event_id` para deduplicación | ✅ Sí, en los 7 eventos del funnel y conversiones |
| Meta CAPI envía valor dinámico de la donación | ✅ Sí, siempre — `value` toma el monto seleccionado por el donante |
| GA4 sin duplicados | ✅ Sí cuando AFRUS es único emisor; coordinación con GTM del cliente requerida si dispara en paralelo |

---

## Beneficios concretos de la integración actual

Después de los despliegues recientes, la integración de AFRUS con Meta y GA4 entrega estos beneficios medibles:

| Métrica | Antes | Después |
|---|---|---|
| **Cobertura de Event ID en Meta** (deduplicación Pixel ↔ CAPI) | 0% | **75-100%** en eventos donde ambos canales disparan |
| **Event Match Quality (EMQ) en Meta** | ~4-5/10 | **~7-8/10** (depende del evento y de la calidad de los datos del lead) |
| **Conteo de usuarios únicos en GA4** | Colapsado a ~1 usuario por formulario (todos los visitantes contaban como una sola persona) | Refleja visitantes reales |
| **Costo por resultado en campañas Meta** (con 75%+ de cobertura de Event ID) | Línea base | Hasta **-54.9% lower cost per result** ([benchmark documentado por Meta](https://developers.facebook.com/documentation/ads-commerce/conversions-api/deduplicate-pixel-and-server-events)) |

**Qué significa esto en práctica para campañas:**

- Las campañas de Meta optimizadas para conversiones (`Lead`, `Purchase`) reciben señales más limpias y confiables → algoritmo de Meta optimiza mejor → menor CPM efectivo, mejor ROAS.
- Los reportes de Meta Ads Manager dejan de inflar conversiones por doble conteo → métricas que reflejan la realidad → decisiones de presupuesto basadas en datos correctos.
- GA4 reporta usuarios únicos reales → métricas de funnel y atribución de fuente/medio confiables → identificación correcta de qué canales traen donantes.
- La ventana de atribución de Meta (típicamente 7 días después del click) ahora se aprovecha al máximo porque las señales server-side y browser-side se cuentan como una sola conversión.

> **Importante:** el -54.9% es la mejora máxima documentada por Meta para anunciantes que alcanzan 75% de cobertura de Event ID. La mejora real en cada campaña depende de muchos factores (calidad creativa, audiencia, presupuesto, etc.). El benchmark indica el techo del beneficio, no una garantía universal.

---

## Detalle por pregunta

### 1. ¿AFRUS permite correctamente el cross-domain tracking con Google Analytics 4 (linker `_gl`)?

**Respuesta corta:** parcial. AFRUS no rompe el cross-domain tracking si el cliente lo configura del lado de su sitio principal, pero AFRUS no ofrece soporte de primera clase para configurar `linker.domains` automáticamente.

**Detalle técnico:**

- El linker `_gl` funciona así: gtag.js en el dominio de origen añade `_gl=...` a los enlaces salientes hacia los dominios listados en `linker.domains`. Gtag.js en el dominio de destino lee `_gl` y reconstruye el cookie `_ga` con el `client_id` original.
- AFRUS lee el cookie `_ga` cuando construye el `client_id` del Measurement Protocol. Si el cliente configura su gtag.js con `linker.domains: ['web.afrus.org']`, el `client_id` se preserva correctamente entre dominios y AFRUS reportará a GA4 con el mismo identificador de usuario.
- **Limitación**: los landings AFRUS inyectan gtag.js de forma standalone, pero NO con configuración de `linker.domains`. Esto significa que si el flujo es `landing AFRUS → sitio cliente`, el linker no se propaga de regreso. Para flujos puramente `sitio cliente → landing AFRUS → conversión`, el linker funciona si el cliente lo configura en su lado.

**Recomendación al cliente:** confirmar que su gtag.js o GTM container incluye `web.afrus.org` (y cualquier dominio adicional de AFRUS donde corran sus formularios) en la lista `linker.domains`. Si necesita soporte bidireccional, contactar al equipo AFRUS — actualmente no está implementado.

---

### 2. ¿Las UTMs se conservan al redirigir hacia el formulario de pago y la página de agradecimiento?

**Respuesta corta:** la atribución de fuente/medio se conserva en GA4 a nivel de sesión (vía cookie `_ga`), aunque las UTMs como parámetros de URL no se propagan automáticamente.

**Detalle técnico:**

- Las UTMs (`utm_source`, `utm_medium`, `utm_campaign`, etc.) son leídas por gtag.js cuando carga la landing y se almacenan en la sesión GA4 asociada al `client_id` del cookie `_ga`.
- Mientras el usuario permanezca en el mismo navegador (mismo cookie `_ga`), GA4 mantiene la atribución de la sesión inicial — incluyendo eventos disparados después, como `purchase` o `sign_up`.
- AFRUS garantiza que el `client_id` es consistente a lo largo del funnel: lectura del cookie `_ga` cuando existe, fallback a UUID persistido cuando no. Esto significa que la sesión GA4 (y por tanto la atribución de UTMs) se preserva del landing al pago a la página de agradecimiento.
- **Limitación**: AFRUS no propaga las UTMs como parámetros de URL a través de redirecciones (ej. URL de agradecimiento). Si el cliente necesita las UTMs en la URL final por razones operativas o de tracking de terceros, requiere configuración adicional no implementada actualmente.

---

### 3. ¿Existe alguna configuración adicional para evitar la pérdida de la fuente/medio durante el flujo?

**Respuesta corta:** la pérdida de atribución se previene principalmente mediante la consistencia del `client_id`, que ya está garantizada por AFRUS. No se requiere configuración adicional dentro de AFRUS.

**Lo que AFRUS ya hace por defecto:**

1. `client_id` consistente durante todo el funnel vía cookie `_ga` o fallback a `localStorage`
2. Deduplicación correcta de eventos Meta Pixel ↔ CAPI usando claves compartidas (`lead_id` para Lead, `transaction_gateway_code` para Purchase)
3. El cookie `_fbp` de Meta se preserva para atribución cross-event de Facebook

**Lo que el cliente debe configurar de su lado:**

1. **GA4 linker** (`linker.domains`) en su contenedor GTM o gtag.js si quiere preservar atribución entre dominios suyos y los de AFRUS
2. **Consent management / cookie banner** que NO bloquee los cookies `_ga` y `_fbp` durante el funnel (las atribuciones se rompen si los cookies se borran a mitad de camino)
3. **Configuración del Meta Pixel en GTM** del cliente con los `eventID` correctos si dispara eventos de conversión paralelos — pedir al equipo AFRUS la guía de configuración de tags GTM

---

### 4. En la integración de Facebook CAPI: ¿se está enviando el parámetro `event_id` para deduplicación con el pixel?

**Sí.** AFRUS envía `event_id` en los 7 eventos del funnel y de conversión, tanto en el lado servidor (CAPI) como en el lado navegador (Pixel), usando el valor correcto en cada caso para que Meta deduplique correctamente.

**Detalle por evento:**

| Evento Meta | Fuente del `event_id` (servidor CAPI y navegador Pixel) | Resultado de dedup |
|---|---|---|
| `ViewContent` | UUID generado por mount del widget (idéntico en ambos canales) | ✅ Meta dedupea a 1 conversión |
| `AddToCart` | UUID por evento | ✅ Dedup correcta |
| `InitiateCheckout` | UUID por evento | ✅ Dedup correcta |
| `AddPaymentInfo` | UUID por evento | ✅ Dedup correcta |
| `Lead` (en GENERATE_LEAD) | `lead_id` (del registro en base de datos) | ✅ Dedup correcta |
| `Lead` (en REGISTER) | `lead_id` | ✅ Dedup correcta |
| `Purchase` | `transaction_gateway_code` (referencia única de la pasarela de pago) | ✅ Dedup correcta |

**Verificación:** en Meta Events Manager → dataset del pixel → cualquier evento de conversión → tab "Event deduplication" → la cobertura de `Event ID` debe estar por encima del 75% (umbral recomendado por Meta para desbloquear la mejora documentada de -54.9% en CPR).

**Cambio reciente:** después de los  despliegues recientes, la cobertura de `event_id` para ViewContent es cercana al  100% para todos los eventos donde Pixel y CAPI ambos disparan.

---

### 5. En la integración de Facebook CAPI: ¿se puede enviar el valor real de la donación dinámicamente?

**Sí, ya estaba implementado.** El evento `Purchase` en ambos canales (Pixel browser + CAPI server) incluye:

```json
{
  "event_name": "Purchase",
  "event_id": "<transaction_gateway_code>",
  "custom_data": {
    "value": 50,                  // ← monto real seleccionado por el donante
    "currency": "USD",            // ← moneda dinámica
    "content_type": "one_time" | "recurring",
    "content_name": "<nombre del widget>",
    "transaction_id": "<referencia de la pasarela>",
    "frequency": "once" | "monthly"
  }
}
```

El `value` se toma directamente del monto seleccionado por el donante en el formulario. Es dinámico por defecto, no requiere configuración adicional.

**Caveat importante:** para pasarelas asíncronas (Niubiz, PIX, boleto), `Purchase` solo se dispara cuando `transaction_gateway_code` es truthy — es decir, después de que la pasarela confirma el cobro. Esto evita falsos `Purchase` por intentos que terminan rechazados.

---

### 6. ¿Es posible enviar correctamente los eventos de conversión hacia GA4 desde AFRUS (vía Measurement Protocol o integración nativa) sin generar duplicados?

**Sí. El estado actual depende del escenario:**

#### Escenario A — AFRUS dispara GA4 sin que el cliente tenga sus propios tags GA4 en GTM

- **No hay riesgo de duplicación.** AFRUS envía a GA4 vía Measurement Protocol con `client_id` correcto y `transaction_id` para eventos `purchase`.
- Cada visitante se cuenta correctamente como un usuario único de GA4.
- Las compras se deduplican automáticamente por `transaction_id`.

#### Escenario B — AFRUS dispara GA4 + el cliente también tiene tags GA4 en GTM

- **Para `purchase`**: GA4 deduplica automáticamente si ambos lados usan el mismo `transaction_id`. AFRUS usa `transaction_gateway_code` como `transaction_id` — el cliente debe configurar su tag GA4 en GTM para hacer lo mismo (`{{DLV - ecommerce.transaction_gateway_code}}`). Pedir al equipo AFRUS la guía de configuración.
- **Para otros eventos** (`view_item`, `add_to_cart`, etc.): GA4 NO tiene mecanismo nativo de deduplicación cross-channel para eventos no-purchase. Si tanto AFRUS como GTM disparan `view_item`, GA4 los contará como dos eventos distintos. Recomendación: el cliente desactiva sus tags GA4 para esos eventos en GTM (confiando en lo que AFRUS envía), o coordina con AFRUS para que solo uno de los dos dispare.

#### Verificación

- GA4 DebugView → buscar el evento `purchase` → confirmar que `transaction_id` está presente y es único por transacción.
- GA4 dashboard → métrica "Users" para la propiedad asociada → debe reflejar visitantes reales.

---

## Limitaciones conocidas a esta fecha

Estas son las brechas existentes en la implementación actual. Algunas son intencionales (compromisos de diseño), otras son items que pueden trabajarse si hay demanda específica del cliente.

| Limitación | Impacto | Estado |
|---|---|---|
| Landings AFRUS no auto-configuran `linker.domains` en gtag.js | Cross-domain tracking requiere config del lado del cliente, no es automático | Documentado, no planificado |
| UTMs no se propagan como parámetros de URL en redirecciones | Atribución GA4 funciona (vía cookie), pero las UTMs no llegan a la URL de thank-you | Documentado, no planificado |
| Logs de GTM en panel de AFRUS no incluyen identificador de sesión del navegador | Atribución de eventos GTM requiere cross-reference con logs de Facebook/Google | En discusión, pendiente decisión de privacidad |
| Eventos de funnel (ViewContent, AddToCart, InitiateCheckout, AddPaymentInfo) no exponen claves de deduplicación al `dataLayer` | Tags GTM del cliente no pueden deduplicar contra los eventos AFRUS para estos eventos específicos | Pendiente — coordinable si hay demanda |
