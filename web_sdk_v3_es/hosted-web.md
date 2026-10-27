# Integración Web

Integración basada en URL para WhatsApp, email, o enlaces directos. El usuario recibe un enlace y completa la verificación en un flujo a pantalla completa alojado en `https://websdk.svi.becomedigital.net/`.

## Formato del Enlace

```
https://websdk.svi.becomedigital.net/?contractId=contract_123&accessToken=eyJ...&flow=onboarding&locale=es
```

## Parámetros

Todos los parámetros se pasan como query parameters de la URL.

### Requeridos

| Parámetro | Descripción |
|-----------|-------------|
| `contractId` | Identificador del contrato |
| `accessToken` | Token de acceso JWT |

### Opcionales

| Parámetro | Descripción | Valores |
|-----------|-------------|---------|
| `userId` | Identificador de usuario | Opcional en onboarding, requerido en reverification |
| `flow` | Tipo de flujo | `onboarding` (default), `reverification` |
| `locale` | Locale de idioma | `es` (default), `en`, `pt` |
| `country` | Código de país ISO 3166 | Omite selección de país |
| `state` | Código de estado/provincia | Se usa con country |
| `docType` | Tipo de documento | `national-id`, `passport`, `driving-license` |
| `nationalIdType` | Subtipo de ID nacional | ej. `CC`, `CE`, `TI` |
| `nationalIdTypeChoices` | Lista de subtipos válidos | Separados por comas |
| `documentNumber` | Número de documento | |
| `redirectUrl` | URL tras completación | |
| `webhookUrl` | URL de callback backend | |
| `mode` | Modo de visualización | `fullscreen` (forzado para integración web) |
| `environment` | Entorno de API | `prod` (default), `dev` |
| `themeMode` | Qué paleta está activa | `light` (usado si se omite), `dark`, `system` |
| `externaltxid` | ID de transacción externa | Para trazabilidad |

### Razón del Theme Mode

El parámetro `themeMode` es una **decisión explícita del cliente** porque el SDK debe respetar la identidad visual de cada cliente en lugar de imponer un esquema de color basado en suposiciones.

Diferentes clientes tienen diferentes requisitos de branding:

- **Solo light:** La marca del cliente define solo un tema visual claro.
- **Solo dark:** La marca del cliente define solo un tema visual oscuro.
- **Light y dark:** El cliente soporta ambos y puede seleccionar el tema según sus propias preferencias de usuario o configuración de la aplicación.
- **Paletas personalizadas:** Una paleta puede contener colores de apariencia oscura mientras sigue siendo el tema claro intencionado del cliente. El SDK no debe inferir el tema desde los colores mismos.

El SDK no puede determinar de forma fiable qué tema es apropiado a partir de la paleta, el sistema operativo del usuario, o suposiciones genéricas sobre modo oscuro. Imponer un tema automáticamente podría introducir colores que entren en conflicto con la identidad de marca del cliente.

Por tanto, el cliente **debe** comunicar explícitamente el modo de theming intencionado a través del parámetro de consulta `themeMode`.

El valor `system` es un **comportamiento opt-in** para clientes que quieren que el SDK siga la preferencia del sistema del usuario. No es el default universal para propósitos de branding. Si la marca de un cliente requiere un tema específico independientemente de la configuración del SO del usuario, el cliente debe pasar `light` o `dark` explícitamente.

### Alias de Parámetros

Los siguientes alias también se aceptan para compatibilidad hacia atrás:

| Alias | Parámetro Canónico |
|-------|---------------------|
| `contract-id` | `contractId` |
| `access-token` | `accessToken` |
| `user-id` | `userId` |
| `flow-type` | `flow` |
| `theme-mode` | `themeMode` |
| `redirect-url` | `redirectUrl` |
| `webhook-url` | `webhookUrl` |

## Comportamiento

- La página alojada lee los query parameters y ejecuta el flujo de verificación en fullscreen. La tarjeta se mantiene dentro de la ventana del navegador.
- El parámetro `mode` se **fuerza a `fullscreen`** independientemente de lo que se pase.
- Si el startup falla, la página muestra un mensaje para el `locale` que pasaste. Un control de retry aparece solo cuando la falla puede reintentarse. La página no muestra el access token. Salir del flujo y una verificación completada no son fallas de startup. Tu aplicación no es notificada de esa falla en el navegador.
- Cuando la verificación tiene éxito y `redirectUrl` está configurado, la página alojada navega el navegador a esa URL. Una falla de startup, un error fatal, o el usuario saliendo del flujo no navega allí. Si la página sigue abierta cuando el usuario termina, navega a `redirectUrl` de nuevo.
- Si `webhookUrl` está configurado, se envía con la verificación para que tu servidor pueda ser llamado cuando el proceso complete. Esa llamada es un callback HTTP a tu backend. No es un evento del navegador, y `document.addEventListener()` en tu página no lo recibirá.
- La persona pasa por el mismo flujo de verificación que en el web component.
- Lenguaje visual, colores, y densidad vienen de la configuración pública del contrato, las mismas opciones que el web component. No son parámetros de URL. Ver [web-sdk.md](web-sdk.md).

## Ejemplo

```
https://websdk.svi.becomedigital.net/?contractId=contract_123&accessToken=eyJhbGciOiJIUzI1NiIs...&flow=onboarding&locale=es&country=CO&redirectUrl=https://myapp.com/thank-you
```

## Notas

- `userId` es opcional en `onboarding`. Si no se provee, el backend genera un UUID.
- `userId` es requerido en `reverification` para matching.
- `nationalIdType` es un subtipo de `national-id` específico de cada país (ej. `CC`, `CE`, `TI`, `PPT` en Colombia).
- `redirectUrl` es la página que el navegador de la persona abre tras una verificación exitosa. No es un callback de JavaScript.
- `webhookUrl` es el callback HTTP de tu servidor. No es un evento del navegador.
- Esta página no emite `become-kyc:ready`, `become-kyc:event`, `become-kyc:error`, `become-kyc:warning`, o eventos `become:*` a tu aplicación. Escuchar en tu propia página, o en una página que solo enmarca esta URL, no los recibe.
- Puedes mostrar la URL alojada en un iframe. La página circundante sigue sin recibir esos eventos. Usa `redirectUrl` y `webhookUrl`.
- Esto es separado de los scripts que añades a tu propia aplicación. La gente abre un enlace. No instala un paquete.

## Relación con Web Components

| Aspecto | Integración Web | Web Components |
|---------|-----------------|----------------|
| Despliegue | App full-page alojada | Script que añades a tu página |
| Cómo se usa | Abre un enlace alojado | Añade `<become-kyc-sdk>` a tu página |
| Modo | Siempre `fullscreen` | `embed`, `overlay`, `fullscreen` |
| Configuración | Solo query parameters | Atributos + `setConfig()` |
| Completación | `redirectUrl` en navegador, `webhookUrl` en tu servidor | Eventos DOM en `<become-kyc-sdk>`. Ver [events.md](events.md) |
| Caso de uso | WhatsApp, email, enlaces directos | Embebido en tu aplicación |

Ambas opciones ejecutan el mismo flujo de verificación. Embébelo en tu aplicación con el web component. Usa esta página alojada solo cuando necesites enviar un enlace, como desde WhatsApp o email.