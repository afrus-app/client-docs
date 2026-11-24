# Crear las plantillas de WhatsApp para entradas

Cómo crear, en AFRUS, las dos plantillas de WhatsApp que usa Entradas (la del enlace de registro y la del recordatorio), cómo conectarlas al evento y cómo probar que funcionan. Si en la sección de WhatsApp del evento ves **«No lista»**, aquí está el motivo de cada aviso y cómo resolverlo.

*Módulo: Eventos · Ubicación: consola > WhatsApp Business y Eventos > Detalles · Audiencia: equipos de eventos y fundraising, Customer Success*

> Esta guía complementa la [Guía de uso de Entradas](./event-ticketing-guide.md). Para entender qué se envía por WhatsApp y por qué, lee [Cómo funciona Entradas](./how-it-works.md).

---

## Qué se envía por WhatsApp

Además del correo, AFRUS puede avisar **al comprador** por WhatsApp con el enlace para registrar a sus invitados y con el recordatorio de registro. Se usan **dos plantillas**, una para cada mensaje:

| Mensaje | Para qué sirve |
|---|---|
| **Enlace de registro** | Llega después de que se envió el correo con el enlace de registro; trae el botón para registrar a los invitados. |
| **Recordatorio** | Llega después de que se envió el correo de recordatorio; trae el mismo botón. |

Cosas que conviene saber:

- WhatsApp es **un extra del correo**: el mensaje solo sale cuando el correo correspondiente ya se envió. Si WhatsApp falla, el correo no se ve afectado.
- Solo se envía **al comprador**, desde la cuenta de WhatsApp Business (Meta) de la organización.
- Los reenvíos que se piden a mano desde la consola **no** generan mensaje de WhatsApp.
- Está **desactivado en cada evento** hasta que lo activas en Detalles → WhatsApp.

---

## Antes de empezar

1. **Una cuenta de WhatsApp Business (Meta) conectada a la organización**, con al menos un número de teléfono. Se conecta en el menú **WhatsApp Business** de la consola. Si no está conectada, la sección de WhatsApp del evento muestra el aviso «La organización no tiene conectada una cuenta de WhatsApp Business (Meta)» y no deja guardar nada.
2. **Permiso para crear plantillas** en esa cuenta. Las plantillas las crea la organización y **Meta debe aprobarlas** antes de poder usarlas.
3. **Dos plantillas distintas**, una para el enlace de registro y otra para el recordatorio. Pueden tener la misma estructura y distinto texto.

---

## Las reglas que debe cumplir cada plantilla

Las dos plantillas deben cumplir exactamente lo mismo. AFRUS revisa estas reglas cada vez que guardas la plantilla y otra vez al enviar cada mensaje:

| Regla | Qué debes hacer |
|---|---|
| **Categoría** | **Utilidad**. Si Meta la cambia después a Marketing, deja de servir. |
| **Estado** | **Aprobada** por Meta. |
| **Texto (cuerpo)** | Hasta **tres variables**, numeradas en orden: `{{1}}`, `{{2}}`, `{{3}}`. Puedes usar menos de tres, nunca más. |
| **Encabezado** | Sin encabezado, o un encabezado de **texto fijo sin variables**. No puede ser imagen, video ni documento. |
| **Pie de página** | Opcional, de texto fijo. |
| **Botón** | **Exactamente un** botón de enlace (URL) con URL dinámica. Puede haber otros tipos de botón, pero solo un botón de enlace dinámico. |
| **URL del botón** | La **dirección de registro de Entradas** seguida de `{{1}}`, tal cual la muestra la consola (ver más abajo). |
| **Idioma** | Cualquier idioma pasa la revisión de AFRUS. AFRUS usa el idioma del comprador si es uno de los idiomas del evento; si no, el primer idioma del evento. La fecha de `{{3}}` se escribe en inglés si ese idioma es el inglés y en español en cualquier otro caso. Ver «Dos idiomas». |

### Qué se pone en cada variable

AFRUS rellena las variables por posición. El orden no se puede cambiar:

| Variable | Qué contiene | Si falta el dato |
|---|---|---|
| `{{1}}` | Nombre del comprador | Su apellido; si tampoco hay, «asistente» (en inglés, «guest») |
| `{{2}}` | Nombre del evento | `-` |
| `{{3}}` | Fecha y hora límite de registro, en la zona horaria del evento | `-` |

Si repites una variable (por ejemplo `{{1}}` dos veces) cuenta como una sola.

### El botón con el enlace de registro

El botón lleva a cada comprador a su página de registro. AFRUS pone en `{{1}}` el código personal de ese comprador, por eso la URL debe terminar en `{{1}}`.

La URL **base** es siempre la misma y la muestra la consola. Ábrela en el evento → **Detalles** → sección de WhatsApp → «Plantillas de WhatsApp (toda la organización)»: verás la línea **«URL del botón de cada plantilla»** con una dirección como:

```
https://<sitio-de-entradas>/claim/{{1}}
```

- Copia **exactamente** esa dirección, sin cambiar mayúsculas ni agregar parámetros. No se distingue entre mayúsculas y minúsculas, pero cualquier otra diferencia (otra ruta, otro dominio, texto extra) hace que la plantilla no sirva.
- Si esa línea no aparece, el sitio de entradas no está configurado: avisa a soporte de AFRUS.

---

## Paso a paso: crear una plantilla

Repite estos pasos dos veces: una para el enlace de registro y otra para el recordatorio.

1. En la consola, entra a **WhatsApp Business** y abre las plantillas («Plantillas de WhatsApp Meta»). Pulsa **Crear plantilla**.
2. **Información básica:**
   - **Nombre de la plantilla:** solo letras minúsculas, números y guiones bajos, empezando con una letra. Por ejemplo `entradas_registro` y `entradas_recordatorio`.
   - **Categoría:** **Utilidad**.
   - **Idioma del mensaje:** Español (o English).
   - Pulsa **Continuar**.
3. **Diseño del mensaje:**
   - **Encabezado:** déjalo vacío (o un texto fijo sin variables). No subas imagen, video ni documento.
   - **Cuerpo del mensaje:** pega el texto del ejemplo de abajo. Para cada variable usa **Agregar variable** y escribe un ejemplo de muestra (por ejemplo «María», «Cena benéfica 2026», «15 de diciembre, 18:00»). Meta pide esas muestras al revisar la plantilla.
   - **Pie de página:** opcional.
   - **Botones:** agrega **un** botón de tipo **Ir al sitio web**, con tipo de URL **URL dinámica (con parámetro)**. En **URL base** escribe la dirección de registro que muestra la consola, **sin** el `{{1}}` final: AFRUS lo agrega solo. En «La URL final será» debes ver la misma dirección que muestra la consola, terminada en `[valor dinámico]`. En **Texto del botón** escribe, por ejemplo, «Registrar asistentes».
4. Pulsa **Enviar a Meta**.
5. Espera la aprobación de Meta. Mientras tanto la plantilla aparece sin aprobar; usa **Actualizar estados** para refrescar. Si Meta la rechaza, corrige lo que indique y envíala de nuevo.

### Ejemplos para copiar y pegar

**Plantilla de enlace de registro** (nombre sugerido `entradas_registro`)

| Campo | Español | English |
|---|---|---|
| Idioma | Español | English |
| Categoría | Utilidad | Utility |
| Cuerpo | `Hola {{1}}, gracias por tu compra para {{2}}. Registra a los asistentes de tus entradas antes del {{3}} con el botón de abajo.` | `Hi {{1}}, thank you for your purchase for {{2}}. Please register the guests for your tickets before {{3}} using the button below.` |
| Texto del botón | `Registrar asistentes` | `Register guests` |
| URL base del botón | la que muestra la consola, sin `{{1}}` | la misma |

**Plantilla de recordatorio** (nombre sugerido `entradas_recordatorio`)

| Campo | Español | English |
|---|---|---|
| Idioma | Español | English |
| Categoría | Utilidad | Utility |
| Cuerpo | `Hola {{1}}, recuerda registrar a los asistentes de {{2}} antes del {{3}}. Usa el botón de abajo.` | `Hi {{1}}, a reminder to register the guests for {{2}} before {{3}}. Please use the button below.` |
| Texto del botón | `Registrar asistentes` | `Register guests` |
| URL base del botón | la que muestra la consola, sin `{{1}}` | la misma |

Muestras para las variables al enviarlas a Meta: `{{1}}` = «María», `{{2}}` = «Cena benéfica 2026», `{{3}}` = «15 de diciembre, 18:00».

### Dos idiomas (opcional)

Puedes crear la misma plantilla en español y en inglés, con **el mismo nombre** y la misma estructura. AFRUS busca primero la versión en el idioma del comprador si es uno de los idiomas del evento, o en el primer idioma del evento si no lo es (por ejemplo, un comprador que habla inglés en un evento solo en español recibe la versión en español). Si esa versión no existe, usa la del idioma que elegiste al guardar. Dos reglas importantes:

- **Todos los idiomas aprobados deben cumplir las reglas.** Si el español está bien y el inglés aprobado no (por ejemplo, otro botón), la plantilla completa aparece como «No lista».
- Publicar un mismo nombre en dos idiomas es una opción de Meta; si prefieres evitarlo, crea solo una plantilla en un idioma.

---

## Conectar las plantillas al evento

Las plantillas se eligen **una sola vez para toda la organización**; después cada evento decide si usa WhatsApp.

1. Abre el evento en la consola → pestaña **Detalles** → baja a la sección **WhatsApp: enlace de registro y recordatorios**.
2. Abre **Plantillas de WhatsApp (toda la organización)**.
3. Para **Enlace de registro**: elige la **Plantilla aprobada** (solo aparecen las que cumplen), elige el **Número que envía** y pulsa **Guardar**. Repite para **Recordatorio**.
4. Cuando las dos muestren **«Lista»**, activa el interruptor **«Enviar por WhatsApp el enlace de registro y los recordatorios de este evento»**. Mientras no estén listas, el interruptor está bloqueado.

Las plantillas que no sirven aparecen más abajo, en **«Plantillas que no sirven y por qué»**, con el motivo de cada una.

---

## Qué significa cada aviso de «No lista» y cómo resolverlo

| Aviso (resumen) | Qué pasa | Cómo resolverlo |
|---|---|---|
| **Meta todavía no aprobó la plantilla** | La plantilla está en revisión, o Meta la rechazó. | Espera la aprobación y pulsa **Actualizar estados**. Si la rechazó, corrígela y vuelve a enviarla. |
| **No es de categoría Utilidad** | La plantilla es de Marketing o de otra categoría. | Crea otra con categoría **Utilidad**. Meta a veces cambia la categoría a Marketing; si pasa, crea la plantilla de nuevo. |
| **El encabezado tiene variables o imagen/archivo** | El encabezado no puede llevar variables ni multimedia. | Crea la plantilla sin encabezado o con un texto fijo. |
| **Más de tres variables en el texto** | El cuerpo usa `{{4}}` o más. | Deja como máximo `{{1}}`, `{{2}}` y `{{3}}`. |
| **Falta el botón con el enlace de registro** | No hay ningún botón de enlace con URL dinámica. | Agrega un botón «Ir al sitio web» con URL dinámica terminada en `{{1}}`. |
| **Más de un botón de enlace dinámico** | Hay dos o más botones de enlace con variable. | Deja solo uno: el del enlace de registro. |
| **El botón no apunta al enlace de registro** | La URL del botón no es la dirección de registro de Entradas. | Usa exactamente la URL que muestra la consola, terminada en `{{1}}`. |
| **Falta elegir la plantilla** | Todavía no guardaste una plantilla para ese mensaje. | Elige plantilla y número, y pulsa **Guardar**. |
| **Está desactivada** | La plantilla se había desactivado con el botón **Desactivar**. | Vuelve a elegirla y pulsa **Guardar**. |
| **No hay cuenta de WhatsApp Business conectada** | La organización no tiene cuenta activa. | Conéctala en el menú **WhatsApp Business**. |
| **El número que envía ya no está en la cuenta** | El número guardado fue quitado de la cuenta de WhatsApp Business. | Elige otro número y pulsa **Guardar**. |
| **La plantilla ya no existe en la cuenta** | Se borró, o no existe en el idioma elegido. | Elige otra plantilla (o crea de nuevo la que falta) y guarda. |
| **Falta configurar el sitio de entradas** | Es un ajuste de AFRUS que la organización no puede cambiar. | Avisa a soporte de AFRUS. |

---

## Probar con una compra

1. Deja las dos plantillas en **«Lista»** y activa WhatsApp en el evento.
2. Compra una entrada de prueba en el widget con un **celular válido** (que no haya rechazado recibir WhatsApp).
3. Cuando el correo del enlace de registro se envíe, poco después llega el WhatsApp con el botón. Comprueba que el nombre, el evento y la fecha estén bien y que el botón abra la página de registro.
4. El recordatorio de WhatsApp sale cuando se envía el correo de recordatorio, no antes.
5. Si algo no llega, abre el detalle de la compra: cada mensaje de WhatsApp muestra su estado y el motivo si no se envió.

Motivos habituales cuando no se envía:

| Motivo en el detalle | Qué revisar |
|---|---|
| El comprador no tiene un celular válido | Que el teléfono esté completo y con su país. |
| WhatsApp desactivado en el evento | El interruptor de Detalles → WhatsApp. |
| Plantilla no configurada | Que las dos plantillas estén guardadas y activas. |
| El número no usa WhatsApp / el comprador no acepta WhatsApp | No se puede enviar a esa persona; el correo sí llegó. |
| La plantilla cambió y ya no sirve | Meta la recategorizó o aprobó otro idioma distinto; revisa los avisos de arriba. |
| Venció sin enviarse | WhatsApp solo se envía en las 24 horas siguientes al correo. |
| El registro ya no estaba abierto / compra cancelada | El mensaje no aplica a esa compra. |

Los mensajes de WhatsApp **no se reintentan** (para no enviar el enlace dos veces); la única excepción es un límite de envío de Meta, que se reintenta durante una hora.

---

## Preguntas frecuentes

**¿Puedo usar una plantilla que ya tengo?** Sí, si cumple todas las reglas de esta guía. Si no, aparece en «Plantillas que no sirven y por qué» con el motivo.

**¿Puedo poner más texto o un pie de página?** Sí. El texto es libre mientras no pases de tres variables, no cambies su orden y mantengas el botón.

**¿El botón puede llevar a otra página?** No. La URL del botón tiene que ser la dirección de registro de Entradas.

**¿Hace falta algo más de Meta?** Meta decide las aprobaciones y sus tiempos; AFRUS no los controla. Revisa el estado en **Actualizar estados**.

---

> **[Volver al índice de la sección](./README.md)** · **[Volver al índice general](../README.md)**

*Última actualización: 2026-11-12*
