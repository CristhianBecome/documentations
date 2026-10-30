# Documentación para integradores

Guías para integrar Become KYC: qué producto usar, cómo cargarlo y qué atributos, métodos y eventos soporta.

## Elige un producto

| Producto | Úsalo cuando | Qué cargas |
|----------|--------------|------------|
| [Web SDK](web-sdk.md) | Incrustas la verificación en tu propia página | `become-kyc-sdk.js` y `<become-kyc-sdk>` |
| [Botón legacy](button.md) | Una integración existente ya usa el botón wrapper | `become-web-button.js` y `<become-button>` |
| [Hosted web](hosted-web.md) | Envías un enlace de verificación, incluyendo desde WhatsApp o email | Una URL en el sitio alojado. Sin instalación de script |

El Web SDK es el producto para nuevas integraciones. El botón es un wrapper de compatibilidad. La página alojada es un enlace a pantalla completa. No proporciona los eventos del navegador del custom element, incluso cuando la URL se coloca en un iframe.

## Pasos de integración

1. **Elige un producto** de la tabla de arriba.
2. **Cárgalo e inícialo** con tu ID de contrato y access token: [web-sdk.md](web-sdk.md), [button.md](button.md), o [hosted-web.md](hosted-web.md).
3. **Configura tu contrato** en el Dashboard: colores, logo, textos y opciones de UI. Sobrescríbelos en una página con `setConfig()`: [customization.md](customization.md).
4. **Maneja el resultado** con eventos, o con `redirectUrl` y `webhookUrl` en la página alojada: [events.md](events.md).
5. **Revisa fallas y casos límite** antes de salir a producción: [troubleshooting.md](troubleshooting.md).

Consulta cualquier atributo, método, campo de configuración o tipo en la [referencia de API](api.md).

## Documentos

| Documento | Descripción |
|-----------|-------------|
| [web-sdk.md](web-sdk.md) | `<become-kyc-sdk>`: configuración inicial, modos de visualización, layout y ciclo de vida |
| [button.md](button.md) | `<become-button>`: configuración inicial, etiqueta del botón y eventos `become:*` |
| [hosted-web.md](hosted-web.md) | Parámetros de la URL alojada, redirect y comportamiento fullscreen |
| [customization.md](customization.md) | Dashboard y `setConfig()`, theme mode, colores, apariencia, logo, textos e íconos de estado |
| [events.md](events.md) | Nombres y payloads de eventos para el Web SDK y el botón |
| [troubleshooting.md](troubleshooting.md) | Fallas de startup, errores durante el flujo y problemas comunes |
| [api.md](api.md) | Referencia: atributos, propiedades, métodos, parámetros de URL, configuración, eventos y tipos |

## Inicio rápido

Incrustar:

```html
<script src="https://websdk.svi.becomedigital.net/sdk/become-kyc-sdk.js"></script>

<become-kyc-sdk
  mode="embed"
  contract-id="contract_123"
  access-token="eyJhbGciOiJIUzI1NiIs..."
  flow="onboarding"
  locale="es"
></become-kyc-sdk>
```

Botón legacy:

```html
<script src="https://websdk.svi.becomedigital.net/sdk/become-web-button.js"></script>

<become-button
  contract-id="contract_123"
  access-token="eyJhbGciOiJIUzI1NiIs..."
  flow="onboarding"
  button-text="Iniciar verificación"
></become-button>
```

Enlace alojado:

```
https://websdk.svi.becomedigital.net/?contractId=contract_123&accessToken=eyJhbGciOiJIUzI1NiIs...&flow=onboarding&locale=es
```
