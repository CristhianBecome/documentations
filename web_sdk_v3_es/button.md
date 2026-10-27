# Web Button (Legacy)

**Integración legacy.**

El custom element `<become-button>` proporciona un botón ligero opcional que carga el SDK perezosamente al hacer click. Abre el SDK en un overlay o contenedor embebido.

> **Nota**: El nombre del custom element registrado es `become-button`, no `become-web-button`. Este es un wrapper legacy. Para nuevas integraciones, usa `<become-kyc-sdk>` directamente.

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

| Atributo | Descripción |
|----------|-------------|
| `contract-id` | Identificador del contrato |
| `access-token` | Token de acceso JWT |
| `user-id` | Identificador de usuario |
| `flow` | Tipo de flujo: `onboarding` o `reverification` |
| `locale` | Locale de idioma |
| `country` | Código de país ISO 3166 |
| `state` | Código de estado/provincia |
| `doc-type` | Tipo de documento |
| `national-id-type` | Subtipo de ID nacional |
| `national-id-type-choices` | Lista de subtipos válidos (separados por comas) |
| `document-number` | Número de documento |
| `redirect-url` | URL tras completación |
| `webhook-url` | URL de callback backend |
| `mode` | `embed` o `overlay` (default: `overlay`) |
| `button-text` | Texto de la etiqueta del botón (default: "Iniciar verificación") |
| `loading-text` | Texto de carga (default: "Cargando...") |
| `hide-icon` | Ocultar el icono de escudo por defecto (atributo booleano) |

### Alias de Atributos (Soporte Legacy V2)

El componente acepta los siguientes nombres de atributos legacy para compatibilidad hacia atrás con integraciones V2:

| Atributo Legacy | Mapea a |
|-----------------|---------|
| `contractId` | `contract-id` |
| `accessToken` / `token` / `data-access-token` | `access-token` |
| `userId` / `data-user-id` / `userid` | `user-id` |
| `flowId` | `flow` |
| `theme` | `theme-mode` |
| `language` | `locale` |
| `redirectUrl` / `redirecturl` | `redirect-url` |
| `webhookUrl` / `webhookurl` | `webhook-url` |
| `docType` | `doc-type` |
| `nationalIdType` | `national-id-type` |

Cuando tanto atributos kebab-case como legacy están presentes, el valor kebab-case tiene precedencia.

## Propiedades (JavaScript API)

Además de los atributos HTML, las siguientes propiedades de JavaScript pueden establecerse/obtenerse en el elemento:

| Propiedad | Tipo | Descripción |
|-----------|------|-------------|
| `contractId` | `string \| null` | Identificador del contrato |
| `accessToken` | `string \| null` | Token de acceso JWT |
| `userId` | `string \| null` | Identificador de usuario |
| `flow` | `string \| null` | Tipo de flujo |
| `mode` | `string` | Modo de visualización (`embed` \| `overlay`), default `overlay` |
| `redirectUrl` | `string \| null` | URL tras completación |
| `webhookUrl` | `string \| null` | URL de callback backend |
| `icon` | `string \| null` | HTML SVG personalizado para el botón |

Establecer una propiedad actualiza el atributo HTML correspondiente. `icon` es la excepción: no se refleja como atributo.

## Métodos

| Método | Descripción |
|--------|-------------|
| `setConfig(publicConfig: Partial<PublicConfig>): void` | Actualiza configuración pública de UI |
| `getConfig(): PublicConfig \| null` | Obtiene la configuración pública resuelta |

## Contenido Personalizado del Botón

El componente soporta contenido personalizado del botón vía elementos hijos. Cuando hay elementos hijos presentes, reemplazan la etiqueta e icono por defecto del botón:

```html
<become-button contract-id="contract_123" access-token="eyJhbGciOiJIUzI1NiIs...">
  <span>Contenido personalizado del botón</span>
  <svg>...</svg>
</become-button>
```

**Nota**: El contenido personalizado se limpia antes de que el botón renderice su propia etiqueta. Un error de startup reemplaza esa área de etiqueta con el mensaje seguro.

## Eventos

El botón emite eventos diferentes a `<become-kyc-sdk>`. Estos eventos legacy se disparan en el elemento botón (`bubbles: true`, `composed: true`). Los payloads también se listan en [events.md](events.md).

| Evento | Cuándo |
|--------|-------|
| `become:result` | Flujo completado exitosamente |
| `become:userFinishedSdk` | Usuario terminó el flujo |
| `become:exitedSdk` | Usuario cerró el SDK |
| `become:errorSdk` | Errores fatales o de token |

### Payloads de Eventos

```javascript
// become:result
{
  flow: 'onboarding' | 'reverification',
  data: unknown
}

// become:userFinishedSdk
{
  flow: 'onboarding' | 'reverification',
  data: unknown
}

// become:exitedSdk
{
  payload: unknown
}

// become:errorSdk
{
  code: string | number,
  message?: string,
  status?: number,
  retryable?: boolean,
  data?: unknown,
  flow?: string
}
```

## Notas Importantes

- El Web Button carga el SDK perezosamente al hacer click.
- Crea un overlay o contenedor embebido automáticamente (posición fixed, z-index: 9999).
- Los eventos se emiten en el elemento botón, no en `document.documentElement`.
- Los nombres de eventos y payloads difieren del KYC Web Component (eventos `become-kyc:*`).
- El componente soporta contenido personalizado vía elementos hijos/slots.
- El modo por defecto es `overlay` (no `embed` como `<become-kyc-sdk>`).
- No hay función `createBecomeButton()`. El uso programático es `document.createElement('become-button')`, property setters, y `click()`.
- Una falla de startup se muestra bajo el botón y se emite como `become:errorSdk` (`code`, `message`, `status`, `retryable`). La frase de usuario es la frase del catálogo del SDK (`errors.startup`) y no incluye el texto diagnóstico. Override con `setConfig({ texts })` antes de abrir el SDK. El SDK también emite `become-kyc:error` por la misma falla. Escucha un contrato.
- `embed` y `overlay` no cambian esa superficie de startup. El SDK no está montado aún, así que el botón no puede pintar el error dentro de una experiencia abierta.
- Si el botón está oculto (`hidden`, o `display: none` / `visibility: hidden` en el botón o un ancestro), el mensaje en línea no se puede ver. Un notice de página no bloqueante muestra la misma frase segura, con retry solo cuando `retryable` es true. Cerrar o eliminar el botón elimina el notice. Un botón visible no muestra también el notice.
- Después de que el SDK ha abierto, un error se reenvía una vez como `become:errorSdk`. El botón no pinta ese error de nuevo. La salida del usuario no es una falla de startup.

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