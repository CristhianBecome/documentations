# Verificación de Identidad - Crear

Endpoint para crear una nueva verificación de identidad enviando documentos e información del usuario.

## POST `/newIdentity`

### Headers requeridos

```http
Content-Type: multipart/form-data
Authorization: Bearer <tu_jwt_token>
```

### Parámetros del request (FormData)

#### Campos requeridos básicos

| Parámetro | Tipo | Descripción |
|-----------|------|-------------|
| **contract_id** | string | Define la configuración del flujo de verificación. Proporcionado por Become Digital |
| **user_id** | string | Identificador único configurable por el cliente. No debe contener caracteres especiales. Máximo 50 caracteres. Se puede utilizar un timestamp o UUID |
| **country** | string | Código ISO-2 del país del documento de identidad (ej: CO, MX, US, AR) |
| **file_type** | string | Tipo de documento: `national-id`, `passport`, `driving-license`, `proof-of-residency` |

#### Archivos requeridos

| Parámetro | Tipo | Descripción |
|-----------|------|-------------|
| **document1** | file | Imagen de la cara frontal del documento (requerido) |
| **document2** | file | Imagen de la cara trasera del documento (requerido si `file_type` es `national-id` o `driving-license`) |
| **video** | file | Video selfie para validación de vida y cotejo facial (requerido si el contrato lo define) |

#### Campos opcionales

| Parámetro | Tipo | Descripción |
|-----------|------|-------------|
| **state** | string | Región o estado. **Requerido solo si** `country` es `US` |
| **specific_document_type** | string | Tipo específico del documento dentro de `file_type` (ej: INE, CC, DNI) |
| **phone_number** | string | Número de teléfono del usuario en formato estándar (E.164 recomendado) |
| **phone_number_enc** | string | Número de teléfono encriptado en formato `b64urlsecure` |
| **email_address** | string | Correo electrónico del usuario |
| **nit** | string | Número de Identificación Tributaria (si aplica) |

### Ejemplo de solicitud

```bash
curl --location 'https://api.svi.becomedigital.net/api/v1/newIdentity' \
--header 'Authorization: Bearer <tu_jwt_token>' \
--form 'contract_id="1"' \
--form 'user_id="usuario_12345_1699123456"' \
--form 'country="CO"' \
--form 'file_type="national-id"' \
--form 'document1=@"/ruta/documento_frontal.jpg"' \
--form 'document2=@"/ruta/documento_trasero.jpg"' \
--form 'video=@"/ruta/selfie_video.mp4"'
```

### Respuestas de la API

#### ✅ **200 OK - Verificación creada**

```json
{
  "code": 201,
  "message": "El recurso fue creado",
  "url_resource": "https://api.svi.becomedigital.net/api/v1/identity/usuario_12345_1699123456",
  "user_id": "usuario_12345_1699123456"
}
```

**Descripción de campos:**

| Campo | Descripción |
|-------|-------------|
| `code` | Código de respuesta (201 = creado exitosamente) |
| `message` | Mensaje descriptivo del resultado |
| `url_resource` | URL para consultar los resultados de esta verificación |
| `user_id` | Identificador del usuario que se verificó |

**Siguientes pasos:**

Usa el `user_id` para consultar los resultados en el endpoint [GET /identity/\<user_id\>](verification-results.md)

## Consideraciones sobre archivos

### Imágenes (document1, document2)

- **Formatos aceptados:** `jpeg`, `jpg`, `png`
- **Resolución óptima:** 600x600 píxeles
- **Peso máximo:** 15 MB
- **Causas comunes de falla:**
  - Resolución insuficiente
  - Imágenes borrosas
  - Documento cortado o parcialmente visible
  - Reflejos o sombras excesivas

### Videos (video)

- **Formato aceptado:** `MP4`
- **Duración máxima:** 7 segundos
- **Peso máximo:** 50 MB
- **Causas comunes de falla:**
  - Más de un rostro detectado
  - Rostros no detectados
  - Múltiples rostros simultáneamente
  - Video borroso o con poca iluminación

## Consideraciones sobre campos

### user_id

- **Restricción:** Sin caracteres especiales ni espacios
- **Longitud máxima:** 50 caracteres
- **Sugerencia:** Concatenar un identificador único con un timestamp para garantizar unicidad
- **Ejemplo:** `usuario_12345_1699123456`

### file_type - Tipos permitidos

- `national-id` - Cédula de ciudadanía, DNI, INE, etc.
- `passport` - Pasaporte
- `driving-license` - Licencia de conducir
- `proof-of-residency` - Comprobante de domicilio

## Manejo de errores

### **Errores de validación de solicitud**

| Mensaje de Error | Descripción |
|------------------|-------------|
| `Debe enviar datos a través de multipart/form` | El encabezado `Content-Type` no empieza con `multipart/form-data` |
| `Faltan datos o archivos` | Faltan datos en `request.form` o archivos en `request.files` |
| `No pueden haber valores null` | Algún valor en los campos enviados es `None` |
| `Falta el ''` | Falta una clave obligatoria (e.g., `file_type`, `user_id`, `contract_id`) |
| `El user_id no debe contener caracteres especiales` | `user_id` contiene caracteres no válidos |
| `El file_type no es válido` | `file_type` no corresponde a los permitidos |
| `El código del país está malo` | El código `country` no es válido según la lista ISO-2 |
| `Debe enviar el estado si selecciona US` | Falta `state` cuando `country` es US |
| `El código del estado está malo` | El estado no corresponde a la lista válida |
| `No debe enviar un estado si no selecciona US` | Se envía `state` pero el `country` no es US |

### **Errores relacionados con contratos**

| Mensaje de Error | Descripción |
|------------------|-------------|
| `No se encontró el contrato` | No se encuentra un contrato válido con el `contract_id` proporcionado |
| `Faltan archivos en la solicitud` | No hay archivos en `request_files` |
| `Su contrato necesita el documento pasaporte adicional` | Falta el pasaporte adicional solicitado por el contrato |
| `Su contrato no necesita el documento pasaporte adicional` | Se envió un pasaporte adicional no permitido por el contrato |
| `Verifica tu cupo` | No hay capacidad disponible para realizar la verificación según el contrato |

### **Errores en validación de documentos**

| Código | Mensaje de Error | Descripción |
|--------|------------------|-------------|
| `400` | `Hubo un problema con el envío de los archivos` | Error general al procesar los archivos enviados |
| `documentError: 1` | `code: "noDocument"` | No se detectó la cara frontal del documento |
| `documentError: 2` | `code: "noDocument"` | No se detectó la cara trasera del documento |
| `documentError: 3` | `code: "noSelfie"` | No se detectó un selfie válido o no corresponde al documento esperado |

### **401 - Unauthorized**

```json
{
  "msg": "Missing Authorization Header"
}
```

**Causa:** No se incluyó el token JWT en el header `Authorization`.

**Solución:** Incluye el header `Authorization: Bearer <tu_jwt_token>` en la solicitud.

## Proceso de validación

La API sigue este flujo:

1. **Validación de autenticación** → Error 401 si falta o es inválido el token
2. **Validación de formato** → Error si no es `multipart/form-data`
3. **Validación de campos obligatorios** → Error si faltan campos requeridos
4. **Validación de contract_id** → Error si el contrato no existe o está inactivo
5. **Validación de archivos** → Error si faltan archivos requeridos
6. **Procesamiento de documentos** → Detección de documento, OCR, validaciones
7. **Creación del recurso** → Código 200 con información del recurso creado

## Flujo asíncrono

> **Importante:** El proceso de verificación es **asíncrono**. La respuesta 200 indica que la verificación fue creada exitosamente, pero los resultados de la validación estarán disponibles gradualmente.

**Para consultar el estado:**

1. Espera unos segundos (recomendado: 5-10 segundos mínimo)
2. Consulta el endpoint [GET /identity/\<user_id\>](verification-results.md)
3. Verifica el campo `verification_status`:
   - `pending` - Aún procesando
   - `completed` - Verificación completada
   - `error` - Ocurrió un error

### Siguientes pasos

- [Consultar resultados de verificación →](verification-results.md)
- [Listar todas las verificaciones →](verification-getall.md)
- [Re-verificación (autenticación) →](re-verification.md)

