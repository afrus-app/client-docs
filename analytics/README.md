# Analytics — Integración con Meta, GA4 y GTM

Documentación para equipos técnicos que integran AFRUS con plataformas de analytics y marketing.

> ⬆️ **[Volver al índice general](../README.md)**

## Contenido de esta sección

### [Preguntas frecuentes sobre Analytics](./faq.md)

Estado actual del soporte de AFRUS para Google Analytics 4, Meta Conversions API y Google Tag Manager. Responde las preguntas más comunes:

- ¿AFRUS permite cross-domain tracking con GA4 (linker `_gl`)?
- ¿Las UTMs se conservan en el flujo de donación?
- ¿Meta CAPI envía `event_id` para deduplicación?
- ¿Se puede enviar el valor real de la donación dinámicamente?
- ¿Es posible enviar eventos a GA4 sin duplicados?
- Limitaciones conocidas a esta fecha

### [Guía de configuración GTM para deduplicación de eventos](./gtm-deduplication-guide.md)

Guía técnica detallada para clientes que tienen sus propias tags de Meta Pixel o GA4 configuradas en Google Tag Manager y necesitan coordinar la deduplicación contra los eventos que AFRUS envía vía CAPI / Measurement Protocol.

Incluye:

- Cómo deduplican Meta y GA4
- Qué campos están disponibles en `window.dataLayer` por cada evento del widget AFRUS
- Ejemplos completos de configuración de tags Meta Pixel y GA4 en GTM
- Cómo verificar que la deduplicación funciona
- Troubleshooting

## Audiencia

- Equipos técnicos de organizaciones clientes (ad-ops, integradores, desarrolladores)
- Customer Success de AFRUS que necesitan referencias para responder consultas técnicas
- Equipos de comunicaciones que necesitan entender qué hace y qué no hace la integración de analytics de AFRUS

## Estado de los documentos

Estos documentos reflejan el estado de la integración de AFRUS a partir de los despliegues de mayo 2026. Las funcionalidades descritas como ✅ están en producción. Las limitaciones señaladas son brechas conocidas que pueden o no resolverse a futuro según demanda.

---

*Última actualización: 2026-05-29*
