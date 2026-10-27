# Become Onboarding/Reverification Web SDK

Este SDK proporciona una solución completa para procesos de onboarding y reverificación de usuarios, incluyendo captura de documentos, liveness facial y validaciones de identidad. Soporta pooling de datos para funcionalidad mejorada.

Este repositorio combina elementos de ambos [Become-web-button](https://github.com/Becomedigital/Become-web-button) y [Become-reverification-web-button](https://github.com/Becomedigital/Become-reverification-web-button).

## Versiones de SDK Disponibles

| SDK | URL del Script | Descripción |
| --- | ---------- | ----------- |
| **Onboarding SDK v2** (Recomendado) | `https://onboarding-v2.svi.becomedigital.net/resources/button.js` | Interfaz mejorada con warnings en desarrollo y eventos de analytics de pasos. |
| **Reverification SDK** | `https://onboarding.svi.becomedigital.net/resources/reverification-button.js` | Flujo standalone de reverificación usando biométrica facial. |

## Eventos y Parámetros del SDK

El SDK se comunica con la ventana padre mediante eventos `postMessage`.

### Eventos Emitidos

| Evento              | Descripción |
|--------------------|-------------------------------------------------------------------------------------------------------------------------------------------------|
| `result`           | Emitido cuando el SDK tiene un resultado para compartir. Para REVERIFICACIÓN: incluye datos completos. Para ONBOARDING: incluye `msg`, `userId` y `endpoint`. |
| `userFinishedSdk`  | Emitido cuando el usuario completa el flujo del SDK exitosamente. Para ONBOARDING: el payload incluye `userId` y `endpoint`. El cliente debe hacer polling a la API o usar webhooks. |
| `exitedSdk`        | Emitido cuando el usuario sale del SDK prematuramente. El payload suele ser null. |
| `initWarning`      | Emitido durante la inicialización si hay warnings (falta país, estado o tipo de documento). El payload incluye la clave del warning. |
| `stepLifecycleEvent` | Emitido para eventos de ciclo de vida de pasos (entrada/salida). v2 tiene soporte mejorado para analytics detallados de pasos. |

### Parámetros Aceptados

| Parámetro     | Tipo   | Requerido | Descripción                                                                  |
| ------------- | ------ | -------- | ---------------------------------------------------------------------------- |
| `accessToken` | string | Sí       | El token de acceso para autenticación (también puede ser `token`).           |
| `userId`      | string | Sí       | El identificador único para el usuario.                                      |
| `contractId`  | string | Sí       | El identificador del contrato para la sesión.                                |
| `country`     | string | No       | Código de país (ej. "US", "CO").                                                    |
| `docType`     | string | No       | Tipo de documento (ej. "passport", "national-id", "driving-license"). Valor único o lista separada por comas. |
| `state`       | string | No       | Código de estado, solo usado si `country` es "US".                           |

### Payload del Evento Result

**Onboarding SDK v2:**

- El evento `userFinishedSdk` retorna `{ msg, userId, endpoint }`
- Los clientes deben hacer polling a la API en el endpoint o configurar webhooks para obtener los datos completos de verificación

**Reverification SDK:**

- El evento `result` retorna datos completos incluyendo `livenessConfidence`, `auditImages`, `referenceImage`, y `data`

Para más información detallada sobre la estructura del campo `data`:

- **ONBOARDING**: [Documentación Postman](https://documenter.getpostman.com/view/2293906/T1DtdvBk?version=latest#283c98a6-b797-48d7-ad1e-c756efca7477)
- **REVERIFICACIÓN**: [Documentación Postman](https://documenter.getpostman.com/view/2293906/T1DtdvBk?version=latest#06c03291-3e2f-4f66-bf6d-a7bf179d17df)