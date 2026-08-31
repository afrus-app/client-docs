# AFRUS — Guías técnicas para clientes

Este repositorio reúne las guías técnicas y de configuración que AFRUS comparte con sus clientes para ayudarles a sacar el máximo provecho de la plataforma.

## Audiencia

- **Equipos técnicos de organizaciones clientes** — integradores, ad-ops, desarrolladores
- **Customer Success de AFRUS** — referencias para responder consultas técnicas de los clientes
- **Equipos de comunicaciones y marketing** — para entender qué hace y qué no hace cada módulo

## Cómo navegar este repositorio

Cada carpeta agrupa documentación de un módulo o área temática. Dentro de cada carpeta hay un `README.md` que lista las guías disponibles. Los documentos son autocontenidos: léelos según necesidad, sin un orden obligatorio.

## Contenido disponible

### 📊 [Analytics](./analytics/)

Integración con Google Analytics 4, Meta Conversions API y Google Tag Manager.

- [Preguntas frecuentes sobre Analytics](./analytics/faq.md) — estado actual de soporte, deduplicación, cross-domain, UTMs, beneficios medibles de la integración
- [Mapa de eventos de Analytics](./analytics/event-map-reference.md) — referencia técnica que mapea cada paso del funnel a su nombre correspondiente en AFRUS, GTM, Meta y GA4, con el flujo end-to-end de deduplicación
- [Guía de configuración GTM para deduplicación de eventos](./analytics/gtm-deduplication-guide.md) — cómo coordinar tus tags de GTM con los eventos que AFRUS ya envía vía CAPI / Measurement Protocol

### [Experimentos A/B de landings](./landing-experiments/)

Creación y lectura de pruebas A/B entre versiones de una landing.

- [Guía completa de experimentos A/B](./landing-experiments/ab-testing-guide.md) — paso a paso de creación, especificación de las variantes y los pesos, ciclo de vida, reparto de tráfico con el splitter, y cómo se calcula cada estadística del panel de resultados (tasa de conversión, lift, significancia, filtrado de bots). Incluye buenas prácticas, limitaciones conocidas y preguntas frecuentes.

### [Flujos de trabajo](./workflows/)

Secuencias automatizadas de correos, etiquetas y puntos.

- [Guía completa de Flujos de trabajo](./workflows/workflows-guide.md) — paso a paso de creación, los seis tipos de paso, disparadores de entrada y salida, la meta, filtros de inscripción, inscripción retroactiva, ventana de envío, ciclo de vida y cómo leer el reporte (tasa de meta, entrega por paso, historial por persona). Incluye buenas prácticas, limitaciones conocidas y preguntas frecuentes.

## Próximamente

Las siguientes secciones están planificadas y se irán incorporando a medida que la documentación esté lista. El orden de aparición se ajustará según prioridades.

- 📧 **Email blast** — uso del módulo de envío masivo de campañas por correo
- 🔄 **Autoresponders** — diseño y configuración de respuestas automáticas
- 🤖 **Fundraiser Agent con IA (Alma)** — uso del agente conversacional de captación de donantes
- 💬 **WhatsApp** — integración con WhatsApp Business (Meta y Evolution APIs)
- 🧩 **Widgets** — configuración de widgets de donación, registro y formularios

---

## Cómo contribuir o reportar errores

Este repositorio es mantenido por el equipo AFRUS. Si encuentras información desactualizada, errores o tienes sugerencias:

- **Equipos AFRUS internos:** abrir un issue o pull request en este repositorio.
- **Clientes externos:** contactar al equipo de Customer Success de AFRUS con la observación específica.

---

*Última actualización del índice: 2026-08-31*
