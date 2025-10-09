# Verificación de Identidad - Crear

Endpoint principal para crear y procesar verificaciones de identidad. Este servicio permite validar documentos de identidad, y comparación facial.

## POST `/newIdentity`

### Headers requeridos

```http
Content-Type: multipart/form-data
Authorization: Bearer <tu_jwt_token>
```

### Parámetros del Request

La petición debe enviarse como `multipart/form-data` e incluir campos de texto y archivos.

#### ✅ Campos OBLIGATORIOS

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `user_id` | string | Identificador único del usuario (sin caracteres especiales ni espacios). Máximo 50 caracteres |
| `contract_id` | string | ID del contrato activo de la empresa |
| `file_type` | string | Tipo de documento: `national-id`, `passport`, `driving-license`, `proof-of-residency` |
| `country` | string | Código ISO-2 del país (ej: CO, MX, US, AR) |

#### 📍 Campos CONDICIONALES

| Campo | Tipo | Obligatorio Si | Descripción |
|-------|------|----------------|-------------|
| `state` | string | `country = "US"` | Código del estado estadounidense (ej: NY, CA) |

**⚠️ Restricción importante:**
- Si `country = "US"`, el campo `state` **ES OBLIGATORIO**
- Si `country != "US"`, **NO se debe enviar** el campo `state`

#### 📱 Campos OPCIONALES

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `city` | string | Ciudad del usuario |
| `street` | string | Dirección/calle |
| `zip` | string | Código postal |
| `phone_number` | string | Número telefónico (formato E.164 recomendado: +573001234567) |
| `email_address` | string | Correo electrónico del usuario |
| `nit` | string | NIT (para empresas en Colombia) |
| `nationalIdType` | string | Tipo específico de ID nacional (ej: CC, TI, CE) |
| `specific_document_type` | string | Subtipo de documento específico |
| `data_policy_consent` | string | Consentimiento de políticas de datos (S/N) |

#### 📁 Archivos REQUERIDOS

**Para `national-id` o `driving-license`:**

| Campo | Descripción | Formato | Requerido |
|-------|-------------|---------|-----------|
| `document1` | Imagen frontal del documento | `.jpg`, `.jpeg`, `.png` | ✅ Sí |
| `document2` | Imagen trasera del documento | `.jpg`, `.jpeg`, `.png` | ✅ Sí |
| `video` | Video/foto de rostro (selfie) | `.mp4`, `.mov`, `.wmv`, `.jpg`, `.png` | ⚠️ Según contrato |

**Para `passport`:**

| Campo | Descripción | Formato | Requerido |
|-------|-------------|---------|-----------|
| `document1` | Imagen del pasaporte | `.jpg`, `.jpeg`, `.png` | ✅ Sí |
| `video` | Video/foto de rostro (selfie) | `.mp4`, `.mov`, `.wmv`, `.jpg`, `.png` | ⚠️ Según contrato |

**⚠️ IMPORTANTE:**
- Para pasaportes, **NO se debe enviar `document2`**
- El campo `video` puede ser imagen o video según configuración del contrato

### Requisitos de calidad

#### Imágenes (document1, document2)
- **Formatos:** `.jpg`, `.jpeg`, `.png`
- **Resolución:** Mínimo 800x600 píxeles
- **Peso máximo:** 5 MB
- **Documento completo visible, buena iluminación, imagen nítida**

#### Videos/Selfie (video)
- **Formatos:** `.mp4`, `.mov`, `.wmv` (video) o `.jpg`, `.png` (imagen)
- **Peso máximo:** 10 MB
- **Duración:** 3-7 segundos (si es video)
- **Rostro centrado, buena iluminación, solo una persona**

#### user_id recomendado
- **Sin caracteres especiales:** Solo letras, números, guiones (`-`) y guiones bajos (`_`)
- **Longitud máxima:** 50 caracteres
- **Ejemplos válidos:** `user-12345-abc`, `550e8400-e29b-41d4`, `usuario_789_1699123456`
- **Ejemplos inválidos:** `user@123#invalid!`, `juan.perez@gmail.com`

### Ejemplos de request

**Verificación de Cédula Colombiana:**

```bash
curl --location 'https://api.svi.becomedigital.net/api/v1/newIdentity' \
--header 'Authorization: Bearer tu_jwt_token_aqui' \
--form 'user_id="col-user-789xyz"' \
--form 'contract_id="23"' \
--form 'file_type="national-id"' \
--form 'country="CO"' \
--form 'city="Bogotá"' \
--form 'phone_number="+573001234567"' \
--form 'email_address="juan.perez@example.com"' \
--form 'nationalIdType="CC"' \
--form 'data_policy_consent="S"' \
--form 'document1=@"cedula_frente.jpg"' \
--form 'document2=@"cedula_atras.jpg"' \
--form 'video=@"selfie_video.mp4"'
```

**Verificación de Licencia USA:**

```bash
curl --location 'https://api.svi.becomedigital.net/api/v1/newIdentity' \
--header 'Authorization: Bearer tu_jwt_token_aqui' \
--form 'user_id="usa-driver-456abc"' \
--form 'contract_id="5"' \
--form 'file_type="driving-license"' \
--form 'country="US"' \
--form 'state="NY"' \
--form 'city="New York"' \
--form 'phone_number="+12125551234"' \
--form 'document1=@"license_front.jpg"' \
--form 'document2=@"license_back.jpg"' \
--form 'video=@"selfie.jpg"'
```

**Verificación de Pasaporte:**

```bash
curl --location 'https://api.svi.becomedigital.net/api/v1/newIdentity' \
--header 'Authorization: Bearer tu_jwt_token_aqui' \
--form 'user_id="mex-passport-321def"' \
--form 'contract_id="26"' \
--form 'file_type="passport"' \
--form 'country="MX"' \
--form 'phone_number="+525512345678"' \
--form 'document1=@"passport.jpg"' \
--form 'video=@"selfie_video.mp4"'
```

**⚠️ Nota:** Para pasaportes NO se envía `document2`

### Respuestas de la API

#### ✅ **201 - Recurso creado exitosamente**

```json
{
  "code": 201,
  "message": "El recurso fue creado",
  "url_resource": "https://api.svi.becomedigital.net/api/v1/identity/usuario_12345_1699123456",
  "user_id": "usuario_12345_1699123456"
}
```

#### ℹ️ **200 - Registro ya existe**

```json
{
  "code": 200,
  "message": "Ya existe un registro con el mismo ID para esta compañia",
  "url_resource": "https://api.svi.becomedigital.net/api/v1/identity/usuario_12345_1699123456"
}
```

**Explicación:** Si ya existe una verificación con el mismo `user_id` para la empresa, se retorna el recurso existente sin crear uno nuevo

### Errores comunes

**400 - Campos inválidos:**
- `"Falta el 'user_id'"` - Campo obligatorio faltante
- `"El user_id no debe contener caracteres especiales"` - Usar solo letras, números, `-` y `_`
- `"Debe enviar el estado si selecciona US"` - Falta campo `state` para country=US
- `"El file_type no es válido"` - Usar: national-id, passport, driving-license, proof-of-residency

**400 - Archivos:**
- `"Hubo un problema con el envío de los archivos"` - Documento no detectado o formato incorrecto
- `documentError: 1` - No se detectó documento frontal
- `documentError: 2` - No se detectó documento trasero

**401 - Autenticación:**
- `"Missing Authorization Header"` - Falta token JWT
- `"Token has expired"` - Token expirado, renovar

**Errores de contrato:**
- `"No se encontró el contrato"` - contract_id inválido
- `"Verifica tu cupo"` - Límite de verificaciones alcanzado

## Flujo asíncrono

> **Importante:** El proceso de verificación es **asíncrono**. La respuesta 201 indica que la verificación fue creada exitosamente y está en proceso de validación. Los resultados finales estarán disponibles una vez se completen todas las validaciones.

### Obtención de resultados

Una vez creada la verificación, tienes dos opciones para obtener los resultados:

**Opción 1: Consulta manual (Polling)**
- Consulta periódicamente el estado usando el endpoint [GET /identity/\<user_id\>](verification-results.md)
- Ver tiempos de procesamiento y documentación completa en: [Consultar Resultados →](verification-results.md)

**Opción 2: Webhook (Recomendado)**
- Configura un webhook para recibir notificaciones automáticas cuando se complete la verificación
- Ver configuración detallada en la sección [Webhooks](#webhooks) más abajo

## Webhooks

Los webhooks permiten recibir notificaciones automáticas cuando se completa una verificación, eliminando la necesidad de hacer polling constante.

### Configuración

1. **Proporciona tu URL** al equipo de soporte de Become Digital
2. **Requisitos de la URL:**
   - Método POST, protocolo HTTPS
   - Accesible públicamente
   - Responder con código 200 OK en menos de 30 segundos
3. **Se configura** internamente en tu `contract_id`

**Ejemplo:** `https://tusitio.com/api/webhooks/become-verification`

### Payload del Webhook

**⚠️ Importante:** El webhook se envía **únicamente cuando el proceso pasa a estado `completed`**.

Recibirás un POST con el siguiente formato:

```json
{
  "event": "verification_completed",
  "data": {
    "timestamp": "2025-10-01T13:45:44.445711",
    "created_at": "2025-10-01 02:33:12.774566",
    "fullname": "JUAN PEREZ LOPEZ",
    "first_name": "JUAN",
    "last_name": "PEREZ LOPEZ",
    "birth": "1990-01-15",
    "document_type": "national-id",
    "document_number": "123456789",
    "verification": {
      "face_match": true,
      "template": true,
      "alteration": false,
      "watch_list": null,
      "verification_status": "completed",
      "liveness": null,
      "liveness_doc": null,
      "ubica": null,
      "ip_validation": true
    },
    "ip": "123.45.67.89"
  },
  "contract_id": 999,
  "identity_id": 123456,
  "user_id": "TESTUSER123456"
}
```

**Llaves principales:**
- `event` - Siempre `"verification_completed"`
- `user_id` - Tu identificador único
- `identity_id` - ID interno de Become Digital
- `contract_id` - ID del contrato
- `data` - Información del documento y resultados
- `data.verification` - Validaciones realizadas (face_match, liveness, alteration, etc.)

> **Nota:** Descripción completa de campos en [GET /identity](verification-results.md)

### Flujo con Webhook

**⚠️ Importante:** El webhook es una **notificación**. Después de recibirlo, **debes consultar** [GET /identity](verification-results.md) para obtener los datos completos.

**Flujo:**
1. Recibir webhook → Notificación de completado
2. Consultar GET /identity/\<user_id\> → Datos completos (OBLIGATORIO)
3. Procesar y almacenar

**Si el webhook falla:**
- ⚠️ No hay reintentos automáticos
- Usar polling como respaldo
- Implementar monitoreo de webhooks

## Mejores prácticas

**Calidad de archivos:**
- Mejor calidad de imagen = Procesamiento más rápido
- Comprimir antes de enviar (máximo 5MB por archivo)
- Verificar iluminación y nitidez antes de subir

**Seguridad:**
- Renovar tokens JWT regularmente
- No exponer tokens en logs o URLs
- Transmitir únicamente por HTTPS
- Validar origen de webhooks

**Performance:**
- Configurar webhook en lugar de polling constante
- Reutilizar el mismo token JWT para múltiples requests
- Implementar timeouts adecuados (60-90 segundos)
- Validar datos antes de enviar para evitar errores

**Manejo de errores:**
- Implementar reintentos automáticos para errores 500
- Registrar respuestas de error para debugging
- Tener proceso de respaldo si el webhook falla

## Siguientes pasos

- [Consultar resultados de verificación →](verification-results.md)
- [Listar todas las verificaciones →](verification-getall.md)
- [Re-verificación (autenticación) →](re-verification.md)

