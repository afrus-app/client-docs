# Mapa de eventos de Analytics — Referencia técnica

**Audiencia:** equipos técnicos de clientes (ad-ops, integradores, analistas de marketing) que necesitan saber exactamente qué eventos AFRUS envía a Meta y a Google Analytics 4, desde dónde los envía, y cómo se deduplican.

**Última actualización:** 2026-05-29

---

## Cómo AFRUS envía eventos de tracking

AFRUS dispara eventos de analytics desde **dos lugares diferentes**, según el momento del funnel del donante:

### 1. Envío desde el servidor (post-pago)

Después de que la pasarela de pago confirma una transacción o un registro, el backend de AFRUS envía un evento directamente a Meta Conversions API.

- Plataformas destino: **solo Meta**
- Cuándo se dispara: después de un webhook de pago exitoso o de creación de un lead confirmado
- No requiere navegador — funciona aunque el usuario cierre la página antes de que cargue el tracking del lado del cliente

### 2. Envío desde el navegador (durante el funnel)

A medida que el donante avanza por el flujo del widget (ver landing → ingresar monto → completar datos → confirmar pago), el widget AFRUS dispara eventos en cada paso.

- Plataformas destino: **Meta** (vía Conversions API) y **Google Analytics 4** (vía Measurement Protocol)
- Cuándo se dispara: a cada acción del usuario en el widget (vista, selección de monto, ingreso de datos, confirmación, etc.)
- Requiere navegador activo durante el flujo

### Por qué hay dos envíos

El envío server-side garantiza que las conversiones críticas (donaciones confirmadas y leads creados) se reporten incluso si el navegador del usuario tiene bloqueadores, falla la conexión, o el usuario cierra la pestaña antes de que cargue todo el tracking.

El envío browser-side captura todo el funnel previo (no solo la conversión final) y agrega señales del lado del cliente que Meta y GA4 valoran para mejorar la calidad de atribución (cookies de tracking, características del dispositivo, etc.).

**Cuando ambos envíos disparan para el mismo evento, AFRUS coordina la deduplicación** (ver sección más abajo) para que cuente como una sola conversión.

---

## Qué eventos se envían a cada plataforma

### Por tipo de widget

| Tipo de widget | Paso del funnel | Evento en Meta | Evento en GA4 |
|---|---|---|---|
| Todos | Vista inicial del widget | `ViewContent` | `view_item` |
| Todos | Selección de monto / interés inicial | `AddToCart` | `add_to_cart` |
| Todos | Captura del lead (datos personales) | `Lead` | `generate_lead` |
| Donación | Inicio del checkout | `InitiateCheckout` | `begin_checkout` |
| Donación | Selección del método de pago | `AddPaymentInfo` | `add_payment_info` |
| Donación ★ | Pago confirmado por la pasarela | `Purchase` *(o `Donate` si la organización lo tiene configurado)* | `purchase` |
| Registro | Registro completado | `Lead` | `sign_up` |

★ = evento de conversión (es lo que se cuenta como "conversión" en Meta Ads Manager y en GA4 para optimización de campañas).

### Notas importantes

- **`Purchase` solo se dispara cuando la pasarela confirma el cobro.** Para pagos asíncronos (Niubiz, PIX, boleto), AFRUS espera la confirmación de la pasarela antes de enviar el evento. Esto previene `Purchase` "fantasma" por intentos que terminan rechazados.
- **`Lead` se dispara cuando el lead se crea exitosamente en la base de datos.** Si falta información requerida, AFRUS no envía el evento (en lugar de enviar uno con datos vacíos que ensuciaría las métricas).
- **Las organizaciones pueden configurar el evento de donación** para enviarse como `Donate` (evento estándar de Meta para sin fines de lucro) en lugar de `Purchase` (evento estándar de e-commerce). Esta configuración se hace a nivel de organización en el backend de AFRUS.
- **El evento `Lead` puede dispararse dos veces durante el flujo** (en captura del lead mid-funnel + al confirmar el registro). AFRUS usa el mismo `lead_id` en ambos, por lo que Meta los deduplica a uno.

---

## Cómo se deduplican los eventos

### Para eventos de conversión que se envían tanto desde servidor como desde navegador

AFRUS envía ambas señales (server-side + browser-side) usando una **clave compartida** que permite a Meta reconocerlas como un mismo evento:

| Evento | Clave de deduplicación que usa AFRUS |
|---|---|
| `ViewContent`, `AddToCart`, `InitiateCheckout`, `AddPaymentInfo` | UUID único generado por cada disparo (compartido entre servidor y navegador) |
| `Lead` (captura del lead y registro completado) | `lead_id` de AFRUS (estable y único por lead) |
| `Purchase` | `transaction_gateway_code` (referencia única que devuelve la pasarela de pago) |

Meta deduplica eventos que comparten `(event_name, event_id)` dentro de una ventana de 48 horas. Si los identificadores coinciden, los cuenta como **una sola conversión** con la información combinada de ambos canales — mejor calidad de match, mejor atribución, mejor optimización de campaña.

### Para GA4

GA4 solo deduplica automáticamente eventos `purchase` que comparten el mismo `transaction_id`. AFRUS envía `transaction_gateway_code` como `transaction_id`, garantizando que las compras se deduplican correctamente.

Para otros eventos (`view_item`, `add_to_cart`, etc.), GA4 no tiene mecanismo nativo de deduplicación cross-canal. Si el cliente tiene además sus propios tags GA4 en GTM disparando estos eventos, ver la [guía de deduplicación GTM](./gtm-deduplication-guide.md).

### Resumen de cuántas conversiones cuenta Meta después de la deduplicación

| Tipo de widget | Lo que Meta cuenta después de dedup |
|---|---|
| Donación | 1 × `Purchase` (servidor + navegador colapsados) |
| Registro / Signup | 1 × `Lead` total (los disparos de captura de lead + de registro completado se deduplican entre sí y contra el envío server-side) |

---

## Configuración necesaria por escenario

### Para que un widget envíe eventos correctamente a Meta (vía Conversions API)

En el editor del widget (admin de AFRUS), pestaña **Avanzado**, sección **Facebook CAPI**:

- **Pixel ID**: el ID del pixel de Meta donde se reportarán los eventos
- **Access Token**: el token de acceso para que AFRUS pueda enviar eventos a la API de Meta en nombre del pixel

Ambos campos son obligatorios. Si falta alguno, el envío server-side a Meta no se dispara y se loguea como error en el panel interno de AFRUS.

### Para que un widget envíe eventos a Google Analytics 4

En el editor del widget, pestaña **Avanzado**, sección **GA4 Measurement Protocol**:

- **Measurement ID** (formato `G-XXXXXXXX`)
- **API Secret**

Ambos campos son obligatorios. Si falta alguno, el envío server-side a GA4 no se dispara.

### Para que la landing dispare el Pixel base de Meta (PageView, captura de cookie `_fbc`)

En el editor del landing, pestaña **Datos avanzados**:

- **Facebook Pixel** (Pixel ID): inyecta el SDK de Meta Pixel en la landing y dispara `PageView` al cargar la página

Este paso es importante para atribución: el Pixel base de la landing es lo que captura el cookie `_fbc` cuando el visitante viene desde un click en un anuncio de Meta (vía `?fbclid=...` en la URL). El widget luego incluye ese `_fbc` en los eventos enviados al servidor de Meta.

### Para que la landing reporte a GA4 desde el lado del navegador (gtag)

En el editor del landing, pestaña **Datos avanzados**:

- **Analytics track** → seleccionar `Google Analytics`
- **Google Analytics ID** (formato `G-XXXXXXXXXX`)

Esto inyecta `gtag.js` en la landing, que dispara `PageView` browser-side y captura las UTMs de la URL en la sesión GA4. El widget complementa con eventos del funnel server-side usando el mismo `client_id` del cookie `_ga` que `gtag.js` configura.

### Configuración mínima por tipo de widget

| Tipo de widget | Configuración mínima en widget | Configuración recomendada en landing |
|---|---|---|
| Donación (única) | Pixel ID + Access Token (Meta), Measurement ID + API Secret (GA4) | Facebook Pixel ID, Google Analytics ID |
| Donación (recurrente / suscripción) | Igual que arriba | Igual que arriba |
| Registro / Signup | Igual que arriba | Igual que arriba |

---

## Cómo verificar end-to-end

### Lado Meta

1. **Events Manager → dataset del pixel → Test events**: dispara un evento de conversión desde un widget AFRUS configurado. Deberías ver el evento con el campo `Event ID` poblado.
2. **Events Manager → Event Deduplication tab** (para `Lead` o `Purchase`): la cobertura de `Event ID` debería superar el 75% (umbral recomendado por Meta para desbloquear hasta -54.9% de mejora en cost-per-result).
3. **Events Manager → Event Match Quality (EMQ)**: el score debería estar idealmente sobre 7/10 para eventos de conversión.

### Lado GA4

1. **GA4 DebugView**: dispara un evento `purchase` y verifica que aparezca con `transaction_id` poblado.
2. **GA4 Reports → Users / Sessions**: la cuenta de usuarios debe reflejar visitantes reales (no inflada ni colapsada).
3. **GA4 DebugView**: confirma que el `client_id` se mantiene estable entre eventos del mismo navegador (mismo `_ga` cookie).

### Lado AFRUS

1. **Panel interno → Procesos → Analytics Events** (`/processes/analytics-events`): muestra cada evento que el servidor de AFRUS envió a Meta o a GA4, con el payload completo y la respuesta de la plataforma destino. Útil para confirmar qué se envió exactamente y verificar que la deduplicación funciona.

---

## Si el cliente tiene sus propios tags Meta Pixel o GA4 en GTM

Si la landing además tiene un contenedor de GTM del cliente con sus propias tags configuradas para disparar eventos de conversión, esos tags duplican el envío a Meta y GA4 — a menos que se configuren para usar las mismas claves de deduplicación que AFRUS.

Ver guía detallada: [**Configuración GTM para deduplicación de eventos**](./gtm-deduplication-guide.md)

---

## Limitaciones conocidas

- **Cross-domain GA4 (linker `_gl`)**: los landings AFRUS no auto-configuran `linker.domains` en gtag.js. Si el flujo cruza dominios (sitio cliente ↔ landing AFRUS), el cliente debe configurar `linker.domains` en su propio gtag.js para preservar el `client_id` entre dominios.
- **UTMs en URLs de redirección**: la atribución GA4 se preserva a través del cookie `_ga`, pero las UTMs no se propagan como parámetros en las URLs de página de agradecimiento.
- **Eventos de funnel (ViewContent, AddToCart, InitiateCheckout, AddPaymentInfo) no exponen claves de deduplicación al `dataLayer`**: si un cliente tiene tags GTM disparando estos eventos en paralelo, no podrán deduplicar contra los eventos AFRUS para estos eventos específicos. Solo aplica a `Lead` y `Purchase` la deduplicación cross-canal vía GTM.

---

*Para preguntas frecuentes adicionales sobre AFRUS y analytics, ver el [FAQ](./faq.md).*
