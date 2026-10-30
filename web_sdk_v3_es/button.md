# Web Button (Legacy)

**Integración legacy.**

El custom element `<become-button>` proporciona un botón ligero opcional que carga el SDK perezosamente al hacer click. Abre el SDK en un overlay o contenedor embebido.

> **Nota**: El nombre del custom element registrado es `become-button`, no `become-web-button`. Este es un wrapper legacy. Para nuevas integraciones, usa `<become-kyc-sdk>` directamente. Ver [web-sdk.md](web-sdk.md).

## Incluir el Bundle

```html
<script src="https://websdk.svi.becomedigital.net/sdk/become-web-button.js"></script>
```

El bundle `<become-button>` es autónomo. No requiere `<become-kyc-sdk>`.

## Uso Básico

```html
<become-button
  contract-id="contract_123"
  access-token="eyJhbGciOiJIUzI1NiIs..."
  flow="onboarding"
  button-text="Iniciar verificación"
></become-button>
```

## Atributos

El botón lee los mismos atributos de verificación que `<become-kyc-sdk>` (`contract-id`, `access-token`, `user-id`, `flow`, `locale`, `country`, `doc-type`, `redirect-url`, `webhook-url`, `theme-mode`, y el resto), más los suyos:

| Atributo | Descripción |
|----------|-------------|
| `mode` | `embed` o `overlay` (default: `overlay`) |
| `button-text` | Texto de la etiqueta del botón (default: "Iniciar verificación") |
| `loading-text` | Texto de carga (default: "Cargando...") |
| `hide-icon` | Ocultar el icono de escudo por defecto (atributo booleano) |

Los valores se leen al hacer click en el botón. La lista completa de atributos, los alias V2 (`token`, `data-user-id`, `flowId`, `theme`, `language`, y otros) y las propiedades de JavaScript están en la [referencia de API](api.md#become-button).

## Etiqueta e icono

El botón renderiza su propia etiqueta e icono. El contenido hijo dentro de `<become-button>` se reemplaza cuando el botón se renderiza. Personalízalo con:

- `button-text` y `loading-text` para la etiqueta.
- `hide-icon` para quitar el icono de escudo por defecto.
- La propiedad `icon` para tu propio markup SVG. No se refleja como atributo. Usa solo markup que controlas.

```javascript
const button = document.querySelector('become-button');
button.icon = '<svg width="18" height="18" viewBox="0 0 24 24" aria-hidden="true">…</svg>';
```

Los colores, el logo y los textos dentro del flujo vienen de la configuración de tu contrato, igual que en el Web SDK. Ver [customization.md](customization.md).

## Métodos

| Método | Descripción |
|--------|-------------|
| `setConfig(publicConfig: Partial<PublicConfig>): void` | Actualiza configuración pública de UI |
| `getConfig(): PublicConfig \| null` | Obtiene la configuración pública resuelta mientras el flujo está abierto |
| `click(): void` | Abre el flujo, como lo hace un click del usuario |

## Eventos

El botón emite eventos diferentes a `<become-kyc-sdk>`. Estos eventos legacy se disparan en el elemento botón (`bubbles: true`, `composed: true`). Los payloads están en [events.md](events.md#eventos-del-web-button-become-button).

| Evento | Cuándo |
|--------|-------|
| `become:result` | Flujo completado exitosamente |
| `become:userFinishedSdk` | Usuario terminó el flujo |
| `become:exitedSdk` | Usuario cerró el SDK |
| `become:errorSdk` | Errores fatales o de token |

## Notas Importantes

- El Web Button carga el SDK perezosamente al hacer click.
- Abre el flujo en su propia capa sobre el contenido de tu página, un hijo directo de `<body>`, y la página no hace scroll detrás. Tras `become:userFinishedSdk` o `become:exitedSdk` el contenedor se cierra y el botón puede abrir el flujo de nuevo.
- Los eventos se emiten en el elemento botón, no en `document.documentElement`.
- Los nombres de eventos y payloads difieren del KYC Web Component (eventos `become-kyc:*`).
- El modo por defecto es `overlay` (no `embed` como `<become-kyc-sdk>`).
- No hay función `createBecomeButton()`. El uso programático es `document.createElement('become-button')`, property setters, y `click()`.
- Una falla de startup aparece al hacer click en el botón. Dónde se muestra depende de `mode`. Ver [Troubleshooting](troubleshooting.md#web-button-legacy).

## Ejemplo Mínimo

```html
<!DOCTYPE html>
<html>
<head>
  <title>KYC Verification Button</title>
</head>
<body>
  <script src="https://websdk.svi.becomedigital.net/sdk/become-web-button.js"></script>
  <become-button
    contract-id="contract_123"
    access-token="eyJhbGciOiJIUzI1NiIs..."
    flow="onboarding"
    button-text="Iniciar verificación"
  ></become-button>
  <script>
    const button = document.querySelector('become-button');
    button.addEventListener('become:result', (e) => {
      console.log('Verification completed:', e.detail);
    });
    button.addEventListener('become:errorSdk', (e) => {
      console.error('Verification error:', e.detail);
    });
  </script>
</body>
</html>
```
