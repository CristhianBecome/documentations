# Onboarding

## Visión General

El flujo de onboarding está diseñado para que nuevos usuarios completen su proceso de verificación de identidad. Esto incluye captura de documentos, verificaciones de liveness facial y validación de identidad.

### Cómo Se Ve en Tu Página

<img src="https://gist.githubusercontent.com/Tyg0th/15c5131ef7d2b24b9effa97eb45dedce/raw/07a5e1f3e428bd1d32bfe2940591872e1ae1ec2d/become-button-example.jpg" width="211" alt="Botón web de Become" />

## Pasos de Integración

Añade el siguiente script al `<head>` de tu HTML (recomendado) o antes de la etiqueta de cierre `</body>`:

```html
<script src="https://onboarding-v2.svi.becomedigital.net/resources/button.js"></script>
```

## Detalles del Flujo

Durante el onboarding, el usuario será guiado a través de los siguientes pasos:

1. Captura de documento (ID, pasaporte, etc.)
2. Verificación de liveness facial
3. Validaciones de identidad contra bases de datos gubernamentales
4. Pooling de datos para verificación mejorada

## Atributos del Botón

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

## Eventos

| Nombre del Evento | Descripción |
| ---------- | ----------- |
| `become:verificationStarted` | Se dispara cuando comienza el proceso de verificación KYC. |
| `become:userFinishedSdk` | Se dispara cuando el usuario completa el proceso de verificación KYC exitosamente. El payload incluye `userId` y `endpoint` - debes hacer polling a la API o usar webhooks para obtener los resultados completos. |
| `become:exitedSdk` | Se dispara cuando el usuario sale del proceso KYC manualmente. |

### Ejemplo de Event Listener

```js
document
  .querySelector("become-button")
  .addEventListener("become:userFinishedSdk", (event) => {
    console.log("El usuario ha completado el proceso KYC:", event.detail);
    // event.detail contiene: { userId, endpoint }
    // Hacer polling a la API en endpoint o usar webhooks para obtener datos completos de verificación
  });
```

## Obtener Resultados

Después de que se emite `userFinishedSdk`, debes recuperar los datos de verificación:

1. **Polling**: Llama a `{endpoint}/api/v1/identity/{userId}` para obtener el resultado completo
2. **Webhooks**: Configura una URL de webhook para recibir notificación cuando la verificación esté lista

## Payload del Evento Result

La estructura de datos de onboarding cuando se recupera de la API:

```json
{
  "data": {
    "id": 0,
    "created_at": "Sat, 25 Jul 2020 15:40:00 GMT",
    "company": "Acc_Demo",
    "fullname": "CLIENT_NAME",
    "birth": "Fri, 03 Jul 1998 00:00:00 GMT",
    "document_type": "national-id",
    "document_number": "CLIENT_DOCUMENT_NUMBER",
    "verification": {
      "alteration": true,
      "estimated_age": true,
      "face_match": true,
      "ip_validation": true,
      "liveness": true,
      "liveness_doc": null,
      "one_to_many_result": true,
      "template": true,
      "ubica": null,
      "verification_status": "completed",
      "watch_list": true
    },
    "blacklist": {
      "result": false,
      "score": 0.8766211271286011,
      "id": 556885
    },
    "comply_advantage": {
      "comply_advantage_result": null,
      "comply_advantage_url": null
    }
  },
  "userId": "user_12345",
  "endpoint": "{API_HOST}/api/v1/identity/{userId}"
}
```

Para más información detallada sobre la estructura del campo `data`, consultar la [Documentación Postman](https://documenter.getpostman.com/view/2293906/T1DtdvBk?version=latest#283c98a6-b797-48d7-ad1e-c756efca7477).