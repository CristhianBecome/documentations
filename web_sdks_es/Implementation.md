# Implementación

## Versiones de SDK Disponibles

| SDK | URL del Script | Descripción |
| --- | ---------- | ----------- |
| **Onboarding SDK v2** (Recomendado) | `https://onboarding-v2.svi.becomedigital.net/resources/button.js` | Interfaz mejorada con warnings en desarrollo y eventos de analytics de pasos. |
| **Reverification SDK** | `https://onboarding.svi.becomedigital.net/resources/reverification-button.js` | Flujo standalone de reverificación usando biométrica facial. |

## Pasos de Integración

### Onboarding SDK v2

Añade el siguiente script al `<head>` de tu HTML (recomendado) o antes de la etiqueta de cierre `</body>`:

```html
<script src="https://onboarding-v2.svi.becomedigital.net/resources/button.js"></script>
```

### Reverification SDK

```html
<script src="https://onboarding.svi.becomedigital.net/resources/reverification-button.js"></script>
```

## Uso en Tu Código

### Onboarding

Para incluir el botón de Become KYC en tu sitio web, simplemente añade la siguiente etiqueta HTML a tu página:

```html
<become-button
  userId="<TU_USER_ID>"
  contractId="<TU_CONTRACT_ID>"
  token="<TU_JWT_TOKEN>"
/>
```

### Reverificación

Para incluir el botón de Become KYC en tu sitio web, simplemente añade la siguiente etiqueta HTML a tu página:

```html
<become-reverification-button
  userId="<TU_USER_ID>"
  contractId="<TU_CONTRACT_ID>"
  token="<TU_JWT_TOKEN>"
/>
```

### Cómo Se Ve en Tu Página

<img src="https://gist.githubusercontent.com/Tyg0th/15c5131ef7d2b24b9effa97eb45dedce/raw/07a5e1f3e428bd1d32bfe2940591872e1ae1ec2d/become-button-example.jpg" width="211" alt="Botón web de Become" />

### Atributos del Botón

| Nombre del Atributo | Descripción |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `userId` | Un identificador único para el usuario que pasa la verificación KYC. |
| `contractId` | El identificador del contrato asociado con el proceso KYC. |
| `token` | Un token JWT creado usando credenciales de la compañía, usado para peticiones API seguras. |
| `country` | Un código de país de 2 caracteres (default: "CO" para Colombia). Ejemplo: "FR" para Francia. |
| `state` | Un código de estado de 2 caracteres (solo requerido para usuarios de US cuando el país es "US"). |
| `docType` | Define el/los tipo(s) de documentos que pueden usarse para verificación KYC. Puede ser un string único o lista separada por comas. Opciones válidas: `national-id`, `passport`, `driving-license`. |

### Ejemplo de Uso de docType

Tipo de Documento Único:
```html
<become-button
  userId="<TU_USER_ID>"
  contractId="<TU_CONTRACT_ID>"
  token="<TU_JWT_TOKEN>"
  docType="passport"
/>
```

Múltiples Tipos de Documento:
```html
<become-button
  userId="<TU_USER_ID>"
  contractId="<TU_CONTRACT_ID>"
  token="<TU_JWT_TOKEN>"
  docType="passport,national-id,driving-license"
/>
```

### Creación Dinámica con JavaScript

```js
const button = document.createElement("become-button");
button.setAttribute("userId", "user_12345");
button.setAttribute("contractId", "contract_67890");
button.setAttribute("token", "tu_jwt_token_aqui");
document.body.appendChild(button);
```

## Eventos

### Onboarding SDK

| Nombre del Evento | Descripción |
| ---------- | ----------- |
| `become:verificationStarted` | Se dispara cuando comienza el proceso de verificación KYC. |
| `become:userFinishedSdk` | Se dispara cuando el usuario completa el proceso de verificación KYC exitosamente. El payload incluye `userId` y `endpoint` - debes hacer polling a la API o usar webhooks para obtener los resultados completos. |
| `become:exitedSdk` | Se dispara cuando el usuario sale del proceso KYC manualmente. |

### Ejemplo de Event Listener (Onboarding)

```js
document
  .querySelector("become-button")
  .addEventListener("become:userFinishedSdk", (event) => {
    console.log("El usuario ha completado el proceso KYC:", event.detail);
    // event.detail contiene: { userId, endpoint }
    // Hacer polling a la API en endpoint o usar webhooks para obtener datos completos de verificación
  });
```

### Reverification SDK

| Nombre del Evento | Descripción |
| ---------- | ----------- |
| `become:sdkLoaded` | Indica que el SDK se cargó exitosamente. |
| `become:verificationStarted` | Notifica que el proceso de re-verificación ha comenzado. |
| `become:result` | Indica la finalización exitosa del proceso de re-verificación; se retornan los datos. |
| `become:userFinishedSdk` | Notifica que el usuario clicó el botón continuar para cerrar el SDK. Solo se emite cuando el proceso de verificación es exitoso. |
| `become:exitedSdk` | Indica que el usuario salió del proceso manualmente usando el botón cerrar. |

### Ejemplo de Event Listener (Reverificación)

```js
document.querySelector('become-reverification-button').addEventListener('become:result', (event) => {
    console.log('Resultado de re-verificación:', event.detail);
    // Los datos completos se retornan en el evento
});
```