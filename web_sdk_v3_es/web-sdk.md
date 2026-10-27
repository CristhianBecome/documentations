# KYC Web Component

Método principal de integración para Become KYC. Incrusta el flujo de verificación en tu aplicación usando el custom element `<become-kyc-sdk>`.

## Incluir el Bundle

```html
<script src="https://websdk.svi.becomedigital.net/sdk/become-kyc-sdk.js"></script>
```

El script es autónomo. No se requiere instalación adicional.

## Uso Básico

```html
<become-kyc-sdk
  mode="embed"
  contract-id="contract_123"
  access-token="eyJhbGciOiJIUzI1NiIs..."
  user-id="USER456"
  flow="onboarding"
  locale="es"
  country="CO"
  redirect-url="https://myapp.com/thank-you"
></become-kyc-sdk>
```

## Atributos

### Requeridos

| Atributo | Descripción |
|----------|-------------|
| `contract-id` | Identificador del contrato |
| `access-token` | Token de acceso JWT |

### Opcionales

| Atributo | Descripción | Valores |
|----------|-------------|---------|
| `user-id` | Identificador de usuario | Requerido para reverificación |
| `flow` | Tipo de flujo | `onboarding` (default), `reverification` |
| `locale` | Locale de idioma | `es` (default), `en`, `pt` |
| `country` | Código de país ISO 3166 | Omite el paso de selección de país |
| `state` | Código de estado/provincia | Se usa con country |
| `doc-type` | Tipo de documento | `national-id`, `passport`, `driving-license` |
| `national-id-type` | Subtipo de ID nacional | ej. `CC`, `CE`, `TI` |
| `national-id-type-choices` | Lista de subtipos válidos | Separados por comas |
| `document-number` | Número de documento | |
| `redirect-url` | URL tras completación | |
| `webhook-url` | URL de callback backend | |
| `mode` | Modo de visualización | `embed` (default), `overlay`, `fullscreen` |
| `environment` | Entorno de API | `prod` (default), `dev` |
| `theme-mode` | Qué paleta está activa | `light` (usado si se omite), `dark`, `system` |
| `loading-text` | Texto de carga personalizado | |

### Razón del Theme Mode

El parámetro `theme-mode` es una **decisión explícita del cliente** porque el SDK debe respetar la identidad visual de cada cliente en lugar de imponer un esquema de color basado en suposiciones.

Diferentes clientes tienen diferentes requisitos de branding:

- **Solo light:** La marca del cliente define solo un tema visual claro.
- **Solo dark:** La marca del cliente define solo un tema visual oscuro.
- **Light y dark:** El cliente soporta ambos y puede seleccionar el tema según sus propias preferencias de usuario o configuración de la aplicación.
- **Paletas personalizadas:** Una paleta puede contener colores de apariencia oscura mientras sigue siendo el tema claro intencionado del cliente. El SDK no debe inferir el tema desde los colores mismos.

El SDK no puede determinar de forma fiable qué tema es apropiado a partir de la paleta, el sistema operativo del usuario o suposiciones genéricas sobre modo oscuro. Imponer un tema automáticamente podría introducir colores que entren en conflicto con la identidad de marca del cliente.

Por tanto, el cliente **debe** comunicar explícitamente el modo de theming intencionado a través del atributo `theme-mode` (o parámetro de consulta `themeMode` para la integración web).

El valor `system` es un **comportamiento opt-in** para clientes que quieren que el SDK siga la preferencia del sistema del usuario. No es el default universal para propósitos de branding. Si la marca de un cliente requiere un tema específico independientemente de la configuración del SO del usuario, el cliente debe pasar `light` o `dark` explícitamente.

### Alias de Atributos (Soporte Legacy)

El componente también acepta los siguientes nombres de atributos legacy para compatibilidad hacia atrás:

| Atributo Legacy | Mapea a |
|-----------------|---------|
| `contractId` | `contract-id` |
| `accessToken` | `access-token` |
| `userId` | `user-id` |
| `flowType` | `flow` |
| `themeMode` | `theme-mode` |
| `redirectUrl` | `redirect-url` |
| `webhookUrl` | `webhook-url` |
| `phoneNumberEnc` | `phone-number-enc` |
| `externaltxid` | `externaltxid` |
| `docType` | `doc-type` |
| `nationalIdType` | `national-id-type` |
| `nationalIdTypeChoices` | `national-id-type-choices` |
| `documentNumber` | `document-number` |

Cuando tanto atributos kebab-case como legacy están presentes, el valor kebab-case tiene precedencia.

## Propiedades (JavaScript API)

Además de los atributos HTML, las siguientes propiedades de JavaScript pueden establecerse/obtenerse en el elemento:

| Propiedad | Tipo | Descripción |
|-----------|------|-------------|
| `contractId` | `string \| null` | Identificador del contrato |
| `accessToken` | `string \| null` | Token de acceso JWT |
| `userId` | `string \| null` | Identificador de usuario |
| `flow` | `string \| null` | Tipo de flujo (`onboarding` \| `reverification`) |
| `mode` | `string` | Modo de visualización (`embed` \| `overlay` \| `fullscreen`), default `embed` |
| `redirectUrl` | `string \| null` | URL tras completación |
| `webhookUrl` | `string \| null` | URL de callback backend |

Establecer una propiedad actualiza el atributo HTML correspondiente automáticamente.

## Métodos

| Método | Descripción |
|--------|-------------|
| `setConfig(publicConfig: Partial<PublicConfig>): void` | Actualiza la configuración pública de UI tras el mount. **Preferido: usa el Dashboard para personalizar tu contrato**; este método permite overrides programáticos para theming, branding, textos y comportamiento de UI. |
| `getConfig(): PublicConfig \| null` | Obtiene la configuración pública resuelta |
| `getConfigContract(): ConfigContractDescriptor` | Obtiene el esquema de configuración pública para editores. Disponible antes del mount. |
| `getResolvedSteps(): ResolvedStepId[]` | Obtiene la lista de pasos resueltos para el flujo activo. Vacío antes del mount. |

## Configuración Pública

Actualiza theming, branding, textos y comportamiento de UI en runtime:

```javascript
const sdk = document.querySelector('become-kyc-sdk');

// Selecciona la paleta activa. Esto no es un campo de setConfig().
sdk.setAttribute('theme-mode', 'dark');

sdk.setConfig({
  theming: {
    colors: {
      light: { primary: '#07a8b8' },
      dark: { primary: '#07a8b8' },
    },
  },
  branding: {
    logo: {
      url: 'https://websdk.svi.becomedigital.net/sdk/logo.svg',
      position: 'top-center',
      size: 'md',
    },
  },
  texts: {
    es: {
      intro: {
        title: 'Título personalizado',
      },
    },
  },
  ui: {
    hideIntroStep: true,
    presentation: {
      contentAlignment: 'center',
    },
  },
});
```

### Dominios de Configuración

| Dominio | Descripción | Opciones Clave |
|---------|-------------|----------------|
| `theming` | Colores, fuentes, espaciado, lenguaje visual | `appearance`, `colors.light/dark`, `typography.fontFamily`, `density`, `radius`, `shadows`, `container.size` |
| `components` | Presentación de controles | `button.size`, `input.labelStyle` (`outside`, `static-top`, `floating`), `input.size`, `stepper.variant`, `errorBox.variant` |
| `branding` | Logo y assets de branding | `logo.url`, `logo.position`, `logo.size`, `logo.opacity`, `logo.shadow`, `logo.radius` |
| `texts` | Strings de texto internacionalizados | Organizados por locale (`es`, `en`, `pt`) y namespace |
| `ui` | Toggles de presentación y confirmación | `confirmations`, `hideIntroHeader`, `hideIntroStep`, `presentation` |

**Nota**: `theme-mode` (`light`, `dark`, o `system`) selecciona la paleta activa. No es parte de `setConfig()`. Cuando el atributo se omite, o el valor no es uno de esos tres, la paleta activa es `light`. `system` sigue el sistema operativo solo cuando lo configuras. Los overrides de color pertenecen bajo `theming.colors.light` y `theming.colors.dark`. Un `theming.colors.primary` plano no es la forma soportada, y `theming.mode` se ignora.

## Eventos

El componente dispara `CustomEvent` en `document.documentElement` (burbujea y composed).

| Evento | Cuándo |
|--------|-------|
| `become-kyc:ready` | SDK montado exitosamente |
| `become-kyc:event` | Eventos de lógica de negocio (ver acciones abajo) |
| `become-kyc:error` | Errores fatales o de token |
| `become-kyc:warning` | Warnings de inicialización no críticos |

### Acciones de `become-kyc:event`

| Acción | Cuándo se emite | Payload |
|--------|-----------------|---------|
| `stepLifecycleEvent` | Transición de ciclo de vida de paso | `{ step, userId?, contractId, status, timestamp? }` |
| `stepChange` | Paso actual cambió | `{ stepId }` |
| `result` | Flujo completado exitosamente | `{ flow, userId, contractId, data }` |
| `atdpDeclined` | Usuario declinó términos de tratamiento de datos | `{ contractId, userId, declinedAt }` |
| `exitedSdk` | SDK salió/cerró | `{ data: null }` |
| `userFinished` | Usuario terminó el flujo | `{ flow?, data? }` |

Ver [events.md](events.md) para referencia completa de eventos con estructuras de payload.

## Ciclo de Vida

1. **Load** — Incluye el bundle del SDK.
2. **Mount** — Añade el custom element al DOM. Hace bootstrap automáticamente.
3. **Flow** — El SDK renderiza los pasos del flujo resuelto secuencialmente.
4. **Cleanup** — El componente se auto-elimina del DOM cuando el flujo termina (`result`, `exitedSdk`) o el usuario declina ATDP (`atdpDeclined`).

## Modos de Visualización

| Modo | Comportamiento |
|------|--------------|
| `embed` | Rellena el contenedor host. Scroll interno cuando el contenido es más alto que el espacio. |
| `overlay` | Modal centrado con backdrop |
| `fullscreen` | Cobertura completa del viewport |

## Layout

El componente se adapta al tamaño del elemento en el que lo colocas. En modo `embed` ese tamaño es tu contenedor, no la ventana del navegador. Un teléfono alto con el componente dentro de un área corta se mantiene compacto. Una ventana más pequeña con un área alta obtiene el layout más abierto. Nada hay que configurar para esto.

### Embed

Establece un width y un height en el elemento host, o en un padre que ya tenga un height definido. Un height de `100%` no hace nada útil hasta que algún ancestro tenga uno.

El componente rellena esa caja. No hará layout más estrecho que 280px ni más corto que 560px. Si le das menos, hace scroll dentro de su propio elemento en lugar de desbordar tu página. Dimensiona el host al menos ese tamaño para que la persona que completa la verificación no haga scroll dentro del componente.

`theming.container.size` limita qué tan ancho crece la tarjeta (`adaptive`, `sm`, `md`, o `lg`). No establece el height. `theming.density` cambia el espaciado (`compact`, `comfortable`, o `spacious`).

### Apariencia

`theming.appearance` selecciona un lenguaje visual para todo el flujo. Omítelo, o pon `default`, y el SDK se ve como hoy.

| Valor | Qué cambia |
|-------|------------|
| `default` | Radios, sombras y bordes actuales |
| `flat` | Esquinas suaves, sin elevación. La separación viene de bordes |
| `material` | Esquinas medias, tarjetas y botones rellenos elevados, campos hundidos |

Un preset es el baseline. Estos campos aún pueden establecerse por separado, incluyendo junto con `default`:

| Campo | Valores soportados | Qué sigue |
|-------|-------------------|-----------|
| `radius.button` | `xs`, `sm`, `md`, `lg`, `xl`, `full` | Botones |
| `radius.input` | `xs`, `sm`, `md`, `lg`, `xl` | Campos y cajas de validación |
| `radius.card` | `xs`, `sm`, `md`, `lg`, `xl` | Tarjeta de paso, tarjetas de opción, diálogos, y esquina del logo |
| `radius.stepper` | `xs`, `sm`, `md`, `lg`, `xl`, `full` | Indicador de progreso |
| `shadows.card` | `none`, `subtle`, `medium`, `elevated` | Tarjeta de paso, botones, y en `material` tarjetas de documento y panel de país |
| `shadows.modal` | `none`, `subtle`, `medium`, `elevated` | Diálogos |
| `borders.width` | 0–4px | Controles y diálogos |
| `borders.style` | `solid`, `dashed`, `dotted` | Controles y diálogos |

Colores, density, fuente, tamaño de tarjeta y texto permanecen independientes del preset. El preset también establece la etiqueta del campo: en el borde para `default`, encima del campo para `flat`, y flotante para `material`. `components.input.labelStyle` o `components.input.size` reemplaza el preset cuando lo configuras.

Orden de resolución, de menor a mayor prioridad:

1. Valores de fábrica.
2. El preset seleccionado, para radius, shadow, border, y la etiqueta y tamaño del campo.
3. Un campo que configuras en esa misma configuración.
4. Una llamada posterior a `setConfig()`. Si omite `appearance`, cambia solo los campos que envía. Si incluye `appearance`, ese preset se aplica de nuevo y solo los campos de radius, shadow, y border en la nueva llamada se mantienen como overrides.

`appearance: "material"` con `shadows.card: "none"` mantiene las esquinas material y la elevación de diálogos, y elimina la sombra de tarjeta y botón. `appearance: "default"` con `radius.card: "xs"` cambia la esquina de la tarjeta y deja el resto de la UI actual en su lugar.

El tamaño de fuente base se mantiene dentro de 12px–24px. Un valor fuera de ese rango se ignora y el SDK usa 16px. El ancho de borde se mantiene dentro de 0–4px.

La esquina del logo es `branding.logo.radius` (de fábrica `lg`, 16px) y su elevación es `branding.logo.shadow` (`none` por defecto). Ninguna sigue `radius.card` o `shadows.card`. Los niveles de sombra son los mismos nombres que usa la tarjeta.

### Overlay y fullscreen

Estos modos cubren el viewport. La tarjeta se mantiene dentro del viewport y conserva un tamaño estable de paso a paso. El tamaño del elemento host no se usa.

### Qué ve la gente

En la pantalla de bienvenida inicial, y en las introducciones a captura de documento y verificación facial, se usa espacio vertical extra cuando el contenedor es más alto que el texto. Las mismas pantallas se mantienen compactas cuando el texto ya llena el área. La acción primaria permanece disponible. Si el texto es más largo que el área, el contenido hace scroll y la acción sigue alcanzable. El texto no se recorta para ajustar.

El indicador de progreso aparece en los pasos por los que la persona avanza. La pantalla de bienvenida inicial y la pantalla de resultado final no lo muestran. También se oculta mientras la cámara está en uso, y vuelve cuando el paso de cámara termina.

## Errores de Startup

`mount()` pinta el flujo solo tras el éxito del startup. Hasta entonces, `<become-kyc-sdk>` posee el error visible. Usa el modo ya seleccionado por el atributo `mode` (default `embed`):

| Cuándo | Qué ve el usuario |
|------|-------------------|
| `embed` startup falla | Un mensaje dentro del elemento, en el área embebida. Sin notice a nivel de página. |
| `overlay` o `fullscreen` startup falla antes de que el SDK abra | El elemento mismo se convierte en la superficie del viewport y muestra un diálogo de alerta. Cerrar descarta esa superficie. El elemento permanece en la página. |
| Startup puede reintentarse (`retryable`) | Un control de reintento ejecuta startup de nuevo. |
| El SDK ya está montado | El SDK pinta la falla dentro de la experiencia abierta. El elemento no añade un segundo mensaje. |
| El elemento se elimina mientras startup está en vuelo | No queda superficie de error. |

`become-kyc:error` sigue llevando el diagnóstico (`CONFIG_FETCH_FAILED`, `message`, `status`, `retryable`). La frase en el elemento es la frase del catálogo del SDK (`errors.startup`): no incluye tokens, códigos de estado, o cuerpos de respuesta. Override con `setConfig({ texts })` antes de que el elemento empiece. HTTP 401 y HTTP 403 usan la frase de autorización, no una credencial errónea de usuario final. Las etiquetas de retry y close de startup son `Reintentar` y `Cerrar`. Salir del flujo (`exitedSdk`, `userFinished`, `atdpDeclined`) no es un error de startup.

## Requisitos del Navegador

- Navegador moderno con soporte de Custom Elements y Shadow DOM
- Target JavaScript ES2020+
- Acceso de red al bundle del SDK y endpoints de API

## Ejemplo Mínimo

```html
<!DOCTYPE html>
<html>
<head>
  <title>KYC Verification</title>
</head>
<body>
  <div id="kyc-root" style="width: 100%; max-width: 480px;"></div>

  <script src="https://websdk.svi.becomedigital.net/sdk/become-kyc-sdk.js"></script>
  <script>
    const container = document.getElementById('kyc-root');
    const sdk = document.createElement('become-kyc-sdk');
    sdk.setAttribute('mode', 'embed');
    sdk.setAttribute('contract-id', 'contract_123');
    sdk.setAttribute('access-token', 'eyJhbGciOiJIUzI1NiIs...');
    sdk.setAttribute('flow', 'onboarding');
    sdk.setAttribute('locale', 'es');
    container.appendChild(sdk);

    document.addEventListener('become-kyc:event', (e) => {
      console.log('SDK event:', e.detail);
    });
  </script>
</body>
</html>
```