# Guía de Face to Face (captación presencial)

Cómo configurar un formulario de donación para captación presencial: quién recibe el aviso de cada donación, qué dice ese aviso y cómo evitar que el navegador de un dispositivo compartido le ofrezca a un donante los datos del anterior.

*Módulo: Formularios · Ubicación: editor del formulario > Avanzado > Face to Face · Audiencia: equipos de fundraising presencial, coordinadores de captadores y Customer Success*

---

## Qué es Face to Face

**Face to Face es un modo del formulario de donación pensado para la captación en la calle, en eventos o puerta a puerta.** Un captador acompaña al donante mientras completa el formulario en una tablet o un teléfono, normalmente el mismo dispositivo para muchas personas a lo largo del día.

Al activarlo en un formulario pasan dos cosas:

1. **Cada donación terminada envía un aviso por correo** al captador y, si lo configuras, a su supervisor. Así el equipo sabe al instante si la donación fue aprobada o rechazada, sin entrar a la plataforma.
2. **Puedes desactivar el autocompletado del navegador** para que el dispositivo no le sugiera a un donante el nombre, el correo, el teléfono, el documento o la tarjeta de quien donó antes.

El formulario se ve y funciona igual para el donante. Face to Face no cambia los campos ni los pasos.

## Antes de empezar

- **La sección solo aparece en formularios de donación.** Los formularios de registro u otros objetivos no la muestran.
- **Ten a mano el correo del captador.** Es obligatorio. El del supervisor es opcional.
- **Un formulario tiene un solo captador.** Todos los avisos de ese formulario van al mismo correo. Si varios captadores trabajan a la vez y cada uno debe recibir solo sus donaciones, crea un formulario por captador (puedes duplicar el formulario y cambiar el correo).

## Paso a paso

1. Abre el formulario en el editor.
2. Ve al panel **Avanzado** y despliega la sección **Face to Face**.
3. Activa el interruptor **Face to Face**.
   Al activarlo, la casilla **Desactivar autocompletado del navegador** se marca sola. Puedes desmarcarla si no la necesitas.
4. Escribe el **correo del captador** (obligatorio) y, si corresponde, el **correo del supervisor**.
5. Guarda el formulario.
6. Haz una donación de prueba y confirma que el aviso llega a los correos configurados.

> **¿Tu formulario ya tenía Face to Face activado antes de esta versión?** La casilla de autocompletado solo se marca sola al encender el interruptor. En formularios que ya estaban activados queda desmarcada: entra, márcala a mano y guarda.

## El aviso por correo

### Cuándo se envía

- Cuando una donación del formulario termina como **aprobada** o como **rechazada**.
- No se envía mientras la donación está pendiente (por ejemplo, un pago que espera confirmación del banco). Llega cuando se resuelve.
- Si el formulario no tiene ningún correo configurado, no se envía nada.
- Si el correo del captador y el del supervisor son el mismo, se envía una sola vez.

### Qué contiene

| Dato | Ejemplo |
|---|---|
| Fecha y hora (en la zona horaria de tu organización) | 2026-09-25 · 14:32:10 |
| Resultado | Donación aprobada / rechazada |
| Motivo del rechazo, si lo hubo | Mensaje devuelto por la pasarela de pago |
| Monto, moneda y frecuencia | 50.00 · BRL · Monthly |
| Medio de pago | Tarjeta de crédito |
| Identificador de la transacción en la pasarela | |
| Nombre del formulario y de la campaña | |
| Datos del donante | Nombre, apellido, correo, teléfono, país e ID en AFRUS |
| Correos del captador y del supervisor | |

> El aviso incluye datos personales del donante. Configura solo correos de personas de tu equipo que deban verlos.

## Desactivar el autocompletado del navegador

### Para qué sirve

En un dispositivo compartido, el navegador guarda lo que escribe cada donante y se lo ofrece al siguiente: al tocar el campo "Nombre" aparece la lista de nombres anteriores, y lo mismo con el correo, el teléfono, el documento, la dirección y la tarjeta. Esta opción le pide al navegador que no muestre esas sugerencias en tu formulario.

### Qué cubre

- Todos los campos de texto del formulario: datos personales, documento, teléfono, fechas, dirección y campos personalizados.
- Los campos de tarjeta que se muestran dentro del formulario, incluidos los que aparecen después de elegir el medio de pago.
- El código de verificación (OTP) no se toca: el teléfono puede seguir sugiriéndolo, porque es del propio donante.

### Qué no cubre

- **Gestores de contraseñas y extensiones del navegador.** Muchos ignoran esta indicación.
- **Pasarelas que abren su propia ventana o página de pago.** Esos campos no son nuestros y el navegador los trata aparte.
- **Datos que el dispositivo ya guardó.** La opción evita que se ofrezcan en tu formulario, pero no los borra del navegador.

**Es una ayuda, no una garantía.** Para dispositivos compartidos recomendamos además:

- Desactivar el autocompletado en la configuración del navegador de cada tablet, o usar el modo invitado o quiosco.
- Pedir a los captadores que respondan **"No"** cuando el navegador pregunte si quiere guardar una dirección o una tarjeta.

### Si el formulario está insertado en tu propio sitio

La casilla del editor es suficiente: el formulario la lee al cargar. Si prefieres controlarlo desde el código de tu página, agrega el atributo con su valor:

```html
<afrus-form form-id="TU-ID-DE-FORMULARIO" disable-autofill="true"></afrus-form>
```

El atributo tiene que llevar el valor `"true"`. Escrito solo, sin valor (`<afrus-form disable-autofill>`), **no** activa la protección. La opción queda activa si está marcada la casilla del editor **o** si la página trae el atributo.

## Cada donante empieza de cero

Después de una donación, la pantalla de agradecimiento no permite empezar otra. El siguiente donante llega a un formulario recargado, y el formulario no conserva nada de la persona anterior. Si el captador necesita empezar otra donación, debe **recargar la página** o volver a abrir el enlace.

## Buenas prácticas

- **Un formulario por captador** si necesitas saber quién consiguió cada donación.
- **Haz una donación de prueba cada vez que cambies los correos** y confirma que el aviso llega.
- **Revisa la carpeta de spam** del captador y del supervisor el primer día.
- **Recarga el formulario entre donante y donante** aunque parezca vacío.

## Preguntas frecuentes

**No veo la sección Face to Face en mi formulario.**
Solo aparece en formularios de donación. Revisa el objetivo del formulario.

**Activé Face to Face pero no llega ningún aviso.**
Confirma que guardaste el formulario, que el correo del captador está bien escrito y revisa la carpeta de spam. Recuerda que las donaciones pendientes no avisan hasta que se resuelven. Si todo está bien y sigue sin llegar, escríbenos con el nombre del formulario y la hora de la donación de prueba.

**¿El aviso llega también en los cobros mensuales de una donación recurrente?**
El aviso está pensado para el momento de la captación. Si ves avisos en cobros posteriores de un donante recurrente, cuéntanos para revisarlo en tu caso.

**Marqué la casilla de autocompletado y el navegador sigue sugiriendo datos.**
Revisa si la sugerencia viene de un gestor de contraseñas o de una extensión: esos no se pueden controlar desde el formulario. Si viene del propio navegador, recarga la página para cargar la versión nueva del formulario y avísanos qué navegador y dispositivo usas.

**¿La casilla de autocompletado funciona sin activar Face to Face?**
Sí. Es independiente del interruptor: si la marcas, el formulario la aplica. Solo se marca sola cuando enciendes Face to Face.

**¿Desactivar el autocompletado afecta a los formularios que no son presenciales?**
No. Solo se aplica al formulario donde la marcas.

---

*Última actualización: 2026-09-25*
