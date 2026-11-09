# Reverificación

## Visión General

El flujo de reverificación está diseñado para que usuarios existentes re-verifiquen su identidad, típicamente por motivos de seguridad o chequeos de cumplimiento periódicos. Este proceso se centra en verificación biométrica y detección de liveness.

### Cómo Se Ve en Tu Página

<img src="https://gist.githubusercontent.com/Tyg0th/15c5131ef7d2b24b9effa97eb45dedce/raw/07a5e1f3e428bd1d32bfe2940591872e1ae1ec2d/become-button-example.jpg" width="211" alt="Botón de reverificación de Become" />

## Pasos de Integración

Añade el siguiente script al `<head>` de tu HTML o antes de la etiqueta de cierre `</body>`:

```html
<script src="https://onboarding.svi.becomedigital.net/resources/reverification-button.js"></script>
```

## Detalles del Flujo

Durante la reverificación, el usuario será guiado a través de los siguientes pasos:

1. Verificación de liveness facial con challenge-response
2. Coincidencia biométrica contra datos de referencia almacenados
3. Puntuación de confianza de liveness
4. Generación de auditoría

## Uso en Tu Código

```html
<become-reverification-button
  userId="<TU_USER_ID>"
  contractId="<TU_CONTRACT_ID>"
  token="<TU_JWT_TOKEN>"
/>
```

### Creación Dinámica con JavaScript

```js
const button = document.createElement('become-reverification-button');
button.setAttribute('userId', 'user_12345');
button.setAttribute('contractId', 'contract_67890');
button.setAttribute('token', 'tu_jwt_token_aqui');
document.body.appendChild(button);
```

## Atributos del Botón

| Nombre del Atributo | Descripción |
| -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `userId` | El identificador único asignado durante el proceso de validación previo. |
| `contractId` | Identificador del contrato para el tipo de validación. |
| `token` | El token JWT generado para peticiones API, creado usando credenciales de la compañía. |

## Eventos

| Nombre del Evento | Descripción |
| ---------- | ----------- |
| `become:sdkLoaded` | Indica que el SDK se cargó exitosamente. |
| `become:verificationStarted` | Notifica que el proceso de re-verificación ha comenzado. |
| `become:result` | Indica la finalización exitosa del proceso de re-verificación; se retornan datos completos. |
| `become:userFinishedSdk` | Notifica que el usuario clicó el botón continuar para cerrar el SDK. Solo se emite cuando el proceso de verificación es exitoso. |
| `become:exitedSdk` | Indica que el usuario salió del proceso manualmente usando el botón cerrar. |

### Ejemplos de Event Listeners

```js
// Escuchar resultado
document.querySelector('become-reverification-button').addEventListener('become:result', (event) => {
    console.log('Resultado de re-verificación:', event.detail);
    // Los datos completos se retornan en el evento
});

// Manejar múltiples eventos
const btn = document.querySelector('become-reverification-button');
btn.addEventListener('become:result', (event) => {
    console.log('Resultado de re-verificación:', event.detail);
});
btn.addEventListener('become:userFinishedSdk', (event) => {
    console.log('Usuario terminó SDK', event.detail);
});
```

## Payload del Evento Result

Cuando se dispara el evento `result`, se retornan los siguientes datos:

```json
{
  "userId": "test_1",
  "livenessConfidence": 99.91615295410156,
  "auditImages": [
    {
      "BoundingBox": {
        "Height": 283.93048095703125,
        "Left": 218.52137756347656,
        "Top": 105.67168426513672,
        "Width": 193.6444854736328
      },
      "Bytes": ""
    }
  ],
  "referenceImage": {
    "BoundingBox": {
      "Height": 287.81195068359375,
      "Left": 206.05030822753906,
      "Top": 76.31394958496094,
      "Width": 201.10231018066406
    },
    "Bytes": ""
  },
  "data": {
    "company": "<NOMBRE_CUENTA_TU_COMPAÑÍA>",
    "user_id": "<USER_ID>",
    "result": false,
    "confidence": 0.20414,
    "liveness": 99.91615295410156,
    "executionId": "UUID"
  }
}
```

## Objeto Result

| Clave | Tipo | Descripción |
| --- | ---- | ----------- |
| `livenessConfidence` | Número (Decimal) | El porcentaje de confianza en el liveness del usuario durante el challenge. Si el score es >= 75, indica que el usuario es probablemente una persona real. |
| `auditImages` | Array | Un array que contiene las imágenes usadas como base para la verificación de liveness. |
| `referenceImage` | Objeto | La imagen de referencia usada para comparar y verificar si el usuario está vivo. |
| `data` | Objeto | El resultado de la comparación entre el selfie más reciente y el asociado al userId. |
| `userId` | String | El identificador único usado para iniciar el proceso de re-verificación. |

## Información de Data

`data` contiene la respuesta de nuestro servicio, indicando si el usuario es quien dice ser.

| Clave | Tipo | Descripción |
| --- | ---- | ----------- |
| `company` | String | La compañía desde donde el usuario hizo onboarding (tu compañía). |
| `user_id` | String | El user ID del individuo (el mismo que se proporcionó). |
| `liveness` | Número (Decimal) | El porcentaje de confianza en el liveness del usuario durante el challenge. |
| `result` | Boolean | Indica si el usuario pasó la re-verificación. `true` significa verificado; `false` significa no verificado. |
| `confidence` | Número (Decimal) | El score de comparación entre el selfie enviado y el asociado al userId. |
| `executionId` | UUID (String) | El identificador único para esta transacción. |

## Notas

- `auditImages.Bytes` y `referenceImage.Bytes` son imágenes codificadas en Base64. Estas imágenes no se guardan en nuestro lado, así que si las necesitas, asegúrate de manejarlas apropiadamente.