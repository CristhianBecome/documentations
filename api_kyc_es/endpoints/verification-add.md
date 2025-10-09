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

### Ejemplo de request

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

### Requisitos de Archivos

#### Imágenes (document1, document2)

- **Formatos aceptados:** `.jpg`, `.jpeg`, `.png`
- **Resolución recomendada:** Mínimo 800x600 píxeles
- **Peso máximo recomendado:** 5 MB por archivo
- **Calidad:**
  - Documento completo visible en el encuadre
  - Buena iluminación sin reflejos
  - Imagen nítida y enfocada
  - Sin sombras excesivas

#### Videos/Selfie (video)

- **Formatos aceptados:** `.mp4`, `.mov`, `.wmv`, `.jpg`, `.png`
- **Peso máximo recomendado:** 10 MB
- **Para video:**
  - Duración recomendada: 3-7 segundos
  - Rostro centrado y completamente visible
  - Buena iluminación
  - Solo un rostro en el encuadre
- **Para imagen:**
  - Rostro frontal y completo
  - Sin accesorios que obstruyan (gafas oscuras, mascarillas)

### Recomendaciones para user_id

- **Sin caracteres especiales:** Evitar `~!@#$%^&*()_+{}":/;'`
- **Longitud máxima:** 50 caracteres
- **Usar identificadores únicos:** UUIDs, timestamps concatenados
- **Ejemplos válidos:**
  - `user-12345-abc` ✅
  - `usuario_789_1699123456` ✅
  - `550e8400-e29b-41d4-a716-446655440000` ✅
- **Ejemplos inválidos:**
  - `user@123#invalid!` ❌
  - `juan.perez@gmail.com` ❌

### Manejo de errores

#### **400 - Bad Request**

**Campo faltante o inválido:**

```json
{ "error": "Falta el 'user_id'" }
```

```json
{ "error": "El user_id no debe contener caracteres especiales o espacios" }
```

```json
{ "error": "El file_type no es válido" }
```

```json
{ "error": "Debe enviar el estado si selecciona US" }
```

```json
{ "error": "El código del país está malo" }
```

```json
{ "error": "El campo document1 solo acepta archivos de extensión *.png, *.jpg, *.jpeg" }
```

**Error en archivos:**

```json
{
  "code": 400,
  "message": "Hubo un problema con el envío de los archivos",
  "error": [
    {
      "documentError": 1,
      "code": "noDocument"
    }
  ]
}
```

**Errores de contrato:**

```json
{ "code": 200, "message": "No se encontró el contrato" }
```

```json
{ "code": 200, "message": "Verifica tu cupo" }
```

```json
{ "code": 200, "message": "Su contrato no admite video ni fotos" }
```

```json
{ "code": 200, "message": "Debe adjuntar un video o imagen" }
```

#### **401 - Unauthorized**

```json
{ "msg": "Missing Authorization Header" }
```

```json
{ "msg": "Token has expired" }
```

```json
{ "msg": "Usuario no autorizado para enviar validaciones" }
```

### Proceso de validación

La API sigue este flujo:

1. **Validación de autenticación** → Error 401 si falta o es inválido el token
2. **Validación de formato** → Error si no es `multipart/form-data`
3. **Validación de campos obligatorios** → Error si faltan campos requeridos (`user_id`, `contract_id`, `file_type`, `country`)
4. **Validación de user_id existente** → Si existe, retorna recurso existente
5. **Validación de contrato** → Error si el contrato no existe, está inactivo o sin cupo
6. **Validación de archivos** → Error si faltan archivos requeridos o tienen formato incorrecto
7. **Procesamiento y almacenamiento** → Validación de documentos y subida de archivos
8. **Creación del recurso** → Respuesta 201 con URL del recurso creado
9. **Procesamiento asíncrono** → OCR, cotejo facial, validaciones adicionales

## Flujo asíncrono

> **Importante:** El proceso de verificación es **asíncrono**. La respuesta 201 indica que la verificación fue creada y está en proceso, pero los resultados finales estarán disponibles después de que se completen las validaciones.

### Tiempos de procesamiento

El tiempo de procesamiento puede variar dependiendo de múltiples factores:

- **Verificaciones estándar:** 10-30 segundos
- **Verificaciones con validaciones adicionales:** 30-60 segundos
- **Verificaciones complejas o con alertas de riesgo:** Hasta 2 minutos

**Factores que afectan el tiempo de procesamiento:**

**Calidad técnica:**
- Resolución de las imágenes
- Claridad y nitidez del documento
- Calidad del video/selfie

**Detección de riesgo y fraude:**
- Detección de pantallas (foto de una pantalla)
- Detección de fotocopias
- Detección de alteraciones en el documento
- Análisis de patrones de fraude
- Validaciones contra listas de riesgo

**Configuración:**
- Cantidad de validaciones habilitadas en el contrato
- Servicios adicionales (ANI, Telco, Email, etc.)

### Obtención de resultados

Una vez creada la verificación, tienes dos opciones para obtener los resultados:

**Opción 1: Consulta manual (Polling)**
- Consulta periódicamente el estado usando el endpoint [GET /identity/\<user_id\>](verification-results.md)
- Ver documentación completa en: [Consultar Resultados →](verification-results.md)

**Opción 2: Webhook (Recomendado)**
- Configura un webhook para recibir notificaciones automáticas
- Ver configuración detallada en la sección [Webhooks](#webhooks) más abajo

## Webhooks

### Configuración de Webhook

Become Digital permite configurar una URL de webhook para recibir notificaciones automáticas cuando se completa una verificación. Esto elimina la necesidad de hacer polling constante.

#### Cómo configurar

1. **Proporciona tu URL de webhook** al equipo de soporte de Become Digital
2. **La URL debe ser:**
   - Método: `POST`
   - Protocolo: `HTTPS` (recomendado)
   - Accesible públicamente
   - Capaz de responder con código 200 OK
3. **El webhook se configura** internamente en tu `contract_id`
4. **Una vez configurado**, cada verificación enviará automáticamente los resultados a tu URL

#### Ejemplo de URL de webhook

```
https://tusitio.com/api/webhooks/become-verification
https://api.tuempresa.com/callbacks/kyc-results
```

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

- **`event`:** Tipo de evento, siempre `"verification_completed"`
- **`user_id`:** Identificador único del usuario enviado en la petición
- **`identity_id`:** ID interno de la verificación
- **`contract_id`:** ID del contrato utilizado
- **`data`:** Objeto con información extraída del documento (nombre, fecha de nacimiento, número de documento, etc.) y resultados de verificación
- **`data.verification`:** Objeto con resultados de las validaciones realizadas (face_match, liveness, alteration, template, etc.)

> **Nota:** Para la descripción completa de todos los campos y resultados de verificación, consulta la documentación de [GET /identity/\<user_id\>](verification-results.md)

### Validación del Webhook

**Tu endpoint debe:**

1. **Validar el origen** de la petición (opcional pero recomendado)
2. **Procesar el payload** y almacenar/actualizar los resultados
3. **Responder con código 200 OK** para confirmar recepción
4. **Responder en menos de 30 segundos** para evitar timeouts



### Consulta manual posterior

**⚠️ Importante:** Una vez recibido el webhook, **debes realizar una consulta manual** al endpoint [GET /identity/\<user_id\>](verification-results.md) para obtener toda la información completamente actualizada y completa.

**¿Por qué es necesario?**
- El webhook es una **notificación**, no contiene todos los detalles
- La consulta manual garantiza obtener la información completa
- Asegura sincronización total de datos

**Flujo correcto:**

1. **Recibir webhook** → Notificación de que se completó
2. **Consultar GET /identity/\<user_id\>** → Obtener datos completos actualizados (OBLIGATORIO)
3. **Procesar y almacenar** → Guardar toda la información en tu sistema

### Si el webhook falla

**⚠️ No hay reintentos automáticos.** Si tu endpoint no está disponible o falla:

- El webhook no se reenviará
- Deberás usar **consulta manual (polling)** para obtener los resultados
- Implementa monitoreo para detectar webhooks fallidos
- Considera tener un proceso de respaldo que consulte periódicamente

### Ventajas del Webhook + Consulta Manual

- ✅ **Notificación inmediata** cuando se completa la verificación
- ✅ **Datos completos y actualizados** mediante consulta posterior
- ✅ **No requiere polling constante** durante el procesamiento
- ✅ **Mejor sincronización** de datos
- ✅ **Mayor confiabilidad** al combinar ambos métodos

## Mejores prácticas

### ✅ Recomendaciones

**1. Generación de user_id:**
- Usar UUIDs o IDs únicos generados por tu sistema
- Evitar datos sensibles (correo, teléfono, documento)
- Ejemplo: `user-550e8400-e29b-41d4-a716-446655440000`

**2. Calidad de imágenes:**
- Resolución mínima: 800x600 píxeles
- Formato recomendado: JPEG
- Iluminación adecuada y uniforme
- Documento completo en el encuadre
- Sin reflejos ni sombras excesivas
- **Mejor calidad = Procesamiento más rápido**

**3. Calidad de video/selfie:**
- Capturar en buena iluminación
- Rostro centrado y sin obstrucciones
- Solo un rostro en el encuadre
- Evitar movimientos bruscos

**4. Obtención de resultados:**
- **Recomendado:** Configurar webhook para notificaciones automáticas
- **Alternativa:** Implementar polling con intervalos de 10-15 segundos
- Esperar al menos 15-30 segundos antes del primer polling
- Timeout máximo: 2 minutos

**5. Manejo de errores:**
- Implementar reintentos automáticos para errores 500
- Validar datos antes de enviar
- Capturar y registrar respuestas de error para debugging
- Manejar timeouts en webhooks

**6. Performance:**
- Comprimir imágenes antes de enviar (máximo 5MB recomendado)
- Implementar timeouts adecuados (60-90 segundos para creación)
- Usar el mismo token JWT para múltiples requests
- Configurar webhook para evitar polling constante

**7. Seguridad:**
- Renovar tokens JWT regularmente
- No exponer tokens en logs o URLs
- Transmitir solo por HTTPS
- Validar origen de webhooks en tu servidor

## Siguientes pasos

- [Consultar resultados de verificación →](verification-results.md)
- [Listar todas las verificaciones →](verification-getall.md)
- [Re-verificación (autenticación) →](re-verification.md)

