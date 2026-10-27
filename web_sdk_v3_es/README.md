# Documentación para integradores

Guías para integrar Become KYC: qué producto usar, cómo cargarlo y qué atributos, métodos y eventos soporta.

## Elige un producto

| Producto | Úsalo cuando | Qué cargas |
|----------|--------------|------------|
| [Web SDK](web-sdk.md) | Incrustas la verificación en tu propia página | `become-kyc-sdk.js` y `<become-kyc-sdk>` |
| [Botón legacy](button.md) | Una integración existente ya usa el botón wrapper | `become-web-button.js` y `<become-button>` |
| [Hosted web](hosted-web.md) | Envías un enlace de verificación, incluyendo desde WhatsApp o email | Una URL en el sitio alojado. Sin instalación de script |

El Web SDK es el producto para nuevas integraciones. El botón es un wrapper de compatibilidad y no incluye los métodos de preview/editor. La página alojada es un enlace a pantalla completa. No proporciona los eventos del navegador del custom element, incluso cuando la URL se coloca en un iframe.

## Documentos

| Documento | Descripción |
|-----------|-------------|
| [web-sdk.md](web-sdk.md) | `<become-kyc-sdk>`: atributos, propiedades, configuración, métodos de producción y métodos de preview/editor |
| [web-sdk-preview.md](web-sdk-preview.md) | Contrato de preview y edición en línea del Dashboard para `<become-kyc-sdk>` |
| [button.md](button.md) | `<become-button>`: atributos legacy, configuración y eventos `become:*` |
| [hosted-web.md](hosted-web.md) | Parámetros de la URL alojada, redirect y comportamiento fullscreen |
| [events.md](events.md) | Nombres y payloads de eventos para los tres productos |

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