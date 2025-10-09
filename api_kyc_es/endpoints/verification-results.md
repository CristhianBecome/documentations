# Verificación de Identidad - Consultar Resultados

Consulta el estado y resultado completo de una verificación de identidad previamente creada.

## GET `/identity/<user_id>`

### Headers requeridos

```http
Authorization: Bearer <tu_jwt_token>
```

### Parámetros

| Parámetro | Ubicación | Tipo | Descripción |
|-----------|-----------|------|-------------|
| `user_id` | URL | string | Identificador único del usuario definido al crear la verificación |

### Ejemplo de solicitud

```bash
curl --location 'https://api.svi.becomedigital.net/api/v1/identity/usuario_12345_1699123456' \
--header 'Authorization: Bearer <tu_jwt_token>'
```

## Tiempos de procesamiento

**Tiempos típicos:**
- Verificaciones estándar: 10-30 segundos
- Con validaciones adicionales: 30-60 segundos
- Con alertas de riesgo: Hasta 2 minutos

**Factores que afectan:**
- Calidad de imágenes (resolución, nitidez, iluminación)
- Detección de fraude (pantallas, fotocopias, alteraciones)
- Servicios adicionales (ANI, Telco, Email)

**Recomendación:** Espera 15-30 segundos antes de la primera consulta. Si está `pending`, consulta cada 10-15 segundos hasta completar (máximo 2 minutos).

## Respuestas

### ✅ **200 - Verificación completada**

```json
{
  "id": 12345,
  "contract_id": 100,
  "company": "Empresa Demo",
  "created_at": "Sat, 25 Jul 2020 15:40:00 GMT",
  "fullname": "JUAN CARLOS GARCIA LOPEZ",
  "first_name": "JUAN CARLOS",
  "last_name": "GARCIA LOPEZ",
  "birth": "1990-05-15",
  "birth_place": "BOGOTA (CUNDINAMARCA)",
  "document_type": "national-id",
  "document_number": "1012345678",
  "dni_number": "1012345678",
  "gender": "M",
  "emission_date": "Mon, 10 Mar 2015 00:00:00 GMT",
  "expiration_date": "Thu, 15 May 2030 00:00:00 GMT",
  "type_id": "TYPE_ID",
  "face_match_score": 95.5,
  "estimated_age": 34,
  "processingStatus": "SUCCESS",
  "frontProcessingStatus": "SUCCESS",
  "backProcessingStatus": "SUCCESS",
  "error_code": "1000 - OK",
  "verification": {
    "verification_status": "completed",
    "face_match": true,
    "liveness": true,
    "alteration": true,
    "template": true,
    "estimated_age": true,
    "one_to_many_result": false,
    "watch_list": false,
    "liveness_doc": null,
    "ubica": null
  },
  "one_to_many_score": 0.0,
  "one_to_many_user_id_matched": null,
  "comply_advantage": {
    "comply_advantage_result": null,
    "comply_advantage_url": null
  },
  "media": {
    "selfiImageUrl": "https://api.svi.becomedigital.net/api/v1/media/abc123-def456-ghi789/selfieImg",
    "frontImgUrl": "https://api.svi.becomedigital.net/api/v1/media/abc123-def456-ghi789/frontImg",
    "backImgUrl": "https://api.svi.becomedigital.net/api/v1/media/abc123-def456-ghi789/backImg"
  },
  "userAgent": {
    "browser_name": "Chrome",
    "browser_version": "118.0.0",
    "os_name": "Windows",
    "os_version": "10",
    "device_type": "pc",
    "device_vendor": "Unknown",
    "device_model": "Unknown"
  },
  "dataMatchResult": {
    "dataMatchResult": "SUCCESS",
    "dateOfBirth": "SUCCESS",
    "documentNumber": "SUCCESS"
  },
  "ip": "192.168.1.100",
  "data_policy_consent": "S",
  "status": "1",
  "uuid": "a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0u1v2w3x4y5z6a7b8c9d0e1f2"
}
```

### ⏳ **202 - Verificación pendiente**

```json
{
  "id": 12346,
  "contract_id": "23",
  "company": "Mi Empresa",
  "created_at": "2025-10-09T14:35:00Z",
  "fullname": null,
  "document_type": "national-id",
  "document_number": null,
  "verification": {
    "verification_status": "pending",
    "face_match": null,
    "liveness": null,
    "alteration": null,
    "template": null
  }
}
```

**Explicación:** Cuando está `pending`, la mayoría de campos son `null` porque el procesamiento no ha terminado. Consulta nuevamente en 10-15 segundos.

## Interpretación de resultados

### Campos principales

**Información básica:**
- `id` - ID interno de la verificación
- `contract_id` - ID del contrato utilizado
- `company` - Nombre de la empresa
- `created_at` - Fecha y hora de creación
- `uuid` - Hash único del estado de verificación (cambia con cada actualización)

**Datos extraídos del documento (OCR):**
- `fullname` - Nombre completo
- `first_name` - Primer nombre
- `last_name` - Apellidos
- `document_type` - Tipo: `national-id`, `passport`, `driving-license`
- `document_number` - Número del documento
- `dni_number` - Número DNI/NUIP
- `birth` - Fecha de nacimiento
- `birth_place` - Lugar de nacimiento
- `gender` - Género (M/F)
- `emission_date` - Fecha de emisión del documento
- `expiration_date` - Fecha de expiración del documento

**Resultados biométricos:**
- `face_match_score` - Puntuación de cotejo facial (0-100)
- `estimated_age` - Edad estimada del rostro

**Estados de procesamiento OCR (informativos):**
- `processingStatus` - Estado general del OCR
- `frontProcessingStatus` - Estado del OCR en imagen frontal
- `backProcessingStatus` - Estado del OCR en imagen trasera
- `error_code` - Código de error (ej: "1000 - OK")

**Valores posibles para processingStatus:**
- `SUCCESS` - Procesamiento exitoso
- `INVALID_CHARACTERS_FOUND` - Se encontraron caracteres inválidos
- `Exception` - Error general de procesamiento
- `AWAITING_OTHER_SIDE` - Esperando procesar el otro lado del documento
- `Recognition process failed to complete in time.` - El proceso de reconocimiento no se completó a tiempo
- `SCANNING_WRONG_SIDE` - Se está escaneando el lado incorrecto del documento
- `RequestException` - Error en la solicitud
- `UNSUPPORTED_CLASS` - Tipo de documento no soportado
- `Unsupported image type` - Tipo de imagen no soportado
- `LOW_CONFIDENCE` - Baja confianza en el reconocimiento
- `BARCODE_RECOGNITION_FAILED` - Falló la lectura del código de barras
- `MRZ_PARSING_FAILED` - Falló el análisis de la zona de lectura mecánica
- `FIELD_IDENTIFICATION_FAILED` - Falló la identificación de campos
- `MANDATORY_FIELD_MISSING` - Campo obligatorio faltante
- `IMAGE_PREPROCESSING_FAILED` - Falló el preprocesamiento de la imagen

**Otros campos:**
- `type_id` - Tipo de ID detectado (ver [Tipos de documento](#tipos-de-documento-detectados))
- `status` - Estado numérico
- `ip` - Dirección IP del usuario
- `data_policy_consent` - Consentimiento de políticas (S/N)

### Objeto `verification` (Resumen de validaciones)

**⚠️ Importante:** Este objeto es un **resumen rápido** de las validaciones principales.

| Campo | Descripción | Valores |
|-------|-------------|---------|
| `verification_status` | Estado general | `pending`, `completed`, `error` |
| `face_match` | Cotejo facial selfie vs documento | `true`, `false`, `null` |
| `liveness` | Prueba de vida (solo con SDK) | `true`, `false`, `null` |
| `alteration` | Detección de alteraciones | `true`, `false`, `null` |
| `template` | Validación de plantilla | `true`, `false`, `null` |
| `estimated_age` | Edad estimada coincide | `true`, `false`, `null` |
| `one_to_many_result` | Coincidencia 1:N (duplicado encontrado) | `true`, `false`, `null` |
| `watch_list` | En listas de vigilancia | `true`, `false`, `null` |
| `ip_validation` | Validación de IP | `true`, `false`, `null` |

**Interpretación de valores:**

- ✅ **`true`** = Validación exitosa, todo seguro
- ⚠️ **`false`** = Se detectó un problema o alerta
- ℹ️ **`null`** = No aplica o no configurada en el contrato

### Razones de alteración y template

**Cuando `alteration: false` o `template: false`**, el sistema ha detectado problemas específicos que pueden incluir:

**Razones de Alteración (`alteration: false`):**
- **Problemas de Registraduría (ANI):** Estado inválido, nombres faltantes o inconsistentes
- **Lista Negra:** Usuario detectado en listas de vigilancia
- **Validación de Documento:** Fallos en la validación del número de documento
- **Estado del Documento:** Fotocopias, fotos de pantallas, o documentos manipulados
- **Modelo de IA:** Detección de alteraciones con alta probabilidad (90%+)
- **Data Match:** Inconsistencias entre información del frente y reverso del documento

**Razones de Template (`template: false`):**
- **Problemas de OCR:** Datos vacíos o falla en identificación de campos
- **Calidad de Imagen:** Documento muy deteriorado, borroso o ilegible
- **Problemas de País:** País no coincide, no está en lista de países aceptados por el cliente
- **Problemas de Tipo:** Tipo de documento incorrecto o no reconocido
- **Problemas de Formato:** Template no reconocido, formato no válido, o error en procesamiento
- **Problemas de Validación:** Documento expirado o información inconsistente

**Importante:** 

- **Solo verás el flag:** El API únicamente retorna `alteration: false` o `template: false` cuando detecta problemas
- **Razón específica:** Para conocer la razón exacta (código específico) se requiere **revisión manual** por parte del equipo de Become Digital
- **Propósito:** El flag sirve como alerta inmediata para que el cliente sepa que hay un problema y tome acción (rechazar, escalar, etc.)
- **Revisión posterior:** Si necesitas la razón específica para casos legales o auditoría, contacta al equipo de soporte con el `user_id` correspondiente

**Ejemplos:**
- `alteration: true` → **No** se detectaron alteraciones ✅
- `alteration: false` → **Sí** se detectaron alteraciones ⚠️
- `watch_list: false` → **No** está en listas de vigilancia ✅
- `watch_list: true` → **Sí** está en listas de vigilancia ⚠️
- `one_to_many_result: false` → **Sí** se encontró duplicado en la cuenta ⚠️
- `one_to_many_result: true` → **No** se encontró duplicado en la cuenta ✅

### Nota sobre Liveness

**⚠️ El liveness es una característica exclusiva del SDK de Become Digital.**

- **Peticiones vía SDK:** El flag `liveness` tendrá valor `true` o `false`
- **Peticiones vía API directa:** El flag `liveness` será `null`

Si integras directamente vía API REST sin usar el SDK, no se realizará la validación de prueba de vida y este campo siempre retornará `null`.

### Objeto `media`

URLs para acceder a las imágenes procesadas:

- `selfiImageUrl` - Selfie/video del usuario
- `frontImgUrl` - Imagen frontal del documento
- `backImgUrl` - Imagen trasera (no incluido en pasaportes)

### Coincidencia 1:N (One-to-Many)

**⚠️ Importante:** Este servicio compara el rostro actual con **todas las validaciones previas de la misma cuenta del cliente**.

Si el servicio 1:N está habilitado y encuentra coincidencia:

- `one_to_many_result` - `false` si el rostro coincide con una validación previa de la misma cuenta (se encontró duplicado)
- `one_to_many_user_id_matched` - User ID de la validación previa con la que coincidió
- `one_to_many_score` - Puntuación de similitud (0-1, donde 1 es idéntico)

**Cómo funciona:**
- ✅ Compara el rostro actual con validaciones anteriores **de la misma cuenta**
- ✅ Si encuentra coincidencia con un `user_id` diferente de la misma cuenta → `one_to_many_result: false` (duplicado detectado)
- ❌ **NO** cruza información con otras cuentas/clientes internos
- ❌ **NO** accede a bases de datos externas

### Objeto `userAgent`

Información del dispositivo usado:

```json
{
  "browser_name": "Chrome Mobile",
  "browser_version": "140.0.0",
  "os_name": "Android",
  "os_version": "10",
  "device_type": "mobile",
  "device_vendor": "Generic_Android",
  "device_model": "K"
}
```

### Objeto `dataMatchResult`

Resultados de coincidencia de datos del documento:

- Estados posibles: `SUCCESS`, `FAILED`, `NOT_PERFORMED`
- Valida: `dateOfBirth`, `dateOfExpiry`, `documentNumber`, etc.

### Objeto `comply_advantage`

Verificación contra listas de sanciones internacionales:

- `comply_advantage_result` - `true` si hay coincidencia, `false` si no, `null` si no aplica
- `comply_advantage_url` - URL del reporte completo (si aplica)

## Campos adicionales condicionales

Dependiendo de la configuración del contrato y los servicios habilitados, la respuesta puede incluir:

### Validación de Email (si se envió `email_address`)

```json
{
  "email_validation": {
    "recent_abuse": false,
    "fraud_score": 15,
    "first_seen": "1 month",
    "domain_age": "10 years",
    "deliverability": "high",
    "dns_valid": true,
    "spf_record": true,
    "dmarc_record": true,
    "honeypot": false,
    "frequent_complainer": false,
    "suspect": false,
    "spam_trap_score": "none",
    "risky_tld": false,
    "sanitized_email": "user@example.com"
  },
  "verification": {
    "email_deliverable": true,
    "email_safe": true
  }
}
```

### Validación Telefónica (si se envió `phone_number`)

```json
{
  "phone_id_validation": {
    "carrier": "Comcel",
    "contact_first_name": null,
    "contact_last_name": null,
    "sim_swap_date": null,
    "sim_swap_score": 1
  },
  "phone_number": "573117631081",
  "phone_score_validation": {
    "phone_score": 301
  }
}
```

### Validación de IP

```json
{
  "ip_validation": {
    "country": "Colombia",
    "region": "Cundinamarca",
    "city": "Bogotá",
    "is_safe": true
  }
}
```

### Registros Gubernamentales (Colombia)

**Objeto `registry` (ANI/Registraduría/Migración):**

```json
{
  "registry": {
    "status": "ACTIVA",
    "nuip": "1012345678"
  }
}
```

**Nota:** Los valores de `status` en el objeto `registry` son los mismos que se utilizan en la documentación de `ani_compliance`. Consulta esa documentación para ver todos los códigos de estado disponibles: [Ver códigos de estado ANI Compliance](./ani_compliance/endpoints/verification.md)

**Objeto `ubica_response` (Validación RUES empresarial):**

```json
{
  "ubica_response": {
    "full_name": "JUAN CARLOS GARCIA LOPEZ",
    "document_number": "1012345678",
    "document_status": "ACTIVA"
  }
}
```



**Campos:**
- `result` - `true` si está en lista negra
- `score` - Similitud (0-1, donde 1 es coincidencia exacta)
- `id` - ID del registro coincidente (si aplica)



### Validación Ubica georefenciacion (Detallada)

Incluye historial de direcciones, teléfonos y emails:

```json
{
  "ubica_response": {
    "full_name": "GARCIA LOPEZ JUAN CARLOS",
    "document_number": "1012345678",
    "document_status": "VIGENTE",
    "age_range": "30-35",
    "addreses": [
      {
        "addresses": "CL 100 # 15 - 20",
        "city": "BOGOTA (CUNDINAMARCA)",
        "location_type": "RES",
        "first_report": "Mon, 15 Jan 2020 00:00:00 GMT",
        "last_report": "Tue, 30 Sep 2024 00:00:00 GMT"
      }
    ],
    "cellphones": [
      {
        "cellphone_number": "3001234567",
        "is_activate": "SI",
        "first_report": "Wed, 20 Mar 2021 00:00:00 GMT",
        "last_report": "Mon, 15 Oct 2024 00:00:00 GMT"
      }
    ],
    "mails": [
      {
        "email": "JUAN.GARCIA@EXAMPLE.COM",
        "first_report": "Fri, 10 Jun 2022 00:00:00 GMT",
        "last_report": "Wed, 01 Oct 2024 00:00:00 GMT"
      }
    ],
    "phones": [
      {
        "phone_number": "6012345678",
        "prefix": "601",
        "city": "BOGOTA (CUNDINAMARCA)",
        "is_activate": "SI"
      }
    ]
  }
}
```




## Tipos de documento detectados

El campo `type_id` indica el tipo específico de documento detectado por el motor OCR:

### Documentos de Identidad Nacional (national-id)

| Tipo | Descripción |
|------|-------------|
| `TYPE_ID` | Cédula o documento nacional de identidad |
| `TYPE_MINORS_ID` | Documento de identidad de menores de edad |
| `TYPE_ALIEN_ID` | Documento de identidad para extranjeros residentes |
| `TYPE_RESIDENT_ID` | Documento de identificación para residentes legales |
| `TYPE_RESIDENCE_PERMIT` | Permiso de residencia |
| `TYPE_TEMPORARY_RESIDENCE_PERMIT` | Permiso de residencia temporal |
| `TYPE_CITIZENSHIP_CERTIFICATE` | Certificado de ciudadanía |
| `TYPE_MULTIPURPOSE_ID` | Documento multipropósito (acceso a varios servicios) |
| `TYPE_VOTER_ID` | Documento para votar |
| `TYPE_PROOF_OF_AGE_CARD` | Tarjeta para acreditar mayoría de edad |
| `TYPE_GREEN_CARD` | Residencia permanente (Green Card) |

### Licencias de Conducción (driving-license)

| Tipo | Descripción |
|------|-------------|
| `TYPE_DRIVER_CARD` | Tarjeta de conductor profesional |
| `TYPE_DRIVING_PRIVILEGE_CARD` | Tarjeta de privilegios de conducción |
| `TYPE_DL` | Licencia de conducción estándar |
| `TYPE_DL_PUBLIC_SERVICES_CARD` | Licencia para transporte público o servicios especiales |

### Pasaportes (passport)

| Tipo | Descripción |
|------|-------------|
| `TYPE_PASSPORT_CARD` | Tarjeta de pasaporte (formato reducido) |
| `TYPE_CONSULAR_PASSPORT` | Pasaporte expedido por consulados |
| `TYPE_MINORS_PASSPORT` | Pasaporte especial para menores de edad |
| `TYPE_ALIEN_PASSPORT` | Pasaporte para extranjeros apátridas o refugiados |
| `TYPE_PASSPORT` | Pasaporte estándar de viaje |

Este valor es informativo y ayuda a identificar automáticamente qué tipo de documento fue procesado.

## Importante: Motor de información, no de decisión

**⚠️ Become Digital NO es un motor de decisión.**

Nosotros proporcionamos:
- ✅ Información detallada sobre la verificación
- ✅ Resultados de validaciones técnicas (OCR, biometría, liveness, etc.)
- ✅ Puntuaciones y flags de riesgo
- ✅ Datos extraídos de registros oficiales

**Tu empresa debe:**
- 🎯 Analizar los resultados proporcionados
- 🎯 Definir tus propios umbrales y reglas de negocio
- 🎯 Tomar decisiones de aprobación/rechazo según tu nivel de riesgo
- 🎯 Implementar lógica de decisión según tus políticas internas

### Soporte del equipo comercial

Nuestro equipo comercial puede ayudarte a:
- 📊 Definir guías de interpretación de resultados
- 🎯 Establecer umbrales según tu industria y nivel de riesgo
- 📋 Crear flujos de decisión personalizados
- 🔍 Analizar casos específicos y patrones

**Contacto:** Solicita asistencia al equipo de soporte de Become Digital.

## Ejemplos de interpretación

### ✅ Verificación exitosa

```json
{
  "verification": {
    "verification_status": "completed",
    "face_match": true,
    "liveness": true,
    "alteration": true,
    "template": true,
    "watch_list": false,
    "one_to_many_result": false
  },
  "face_match_score": 95.5
}
```
**Interpretación:** 
- ✅ Cotejo facial exitoso (`face_match: true`)
- ✅ Prueba de vida superada (`liveness: true`)
- ✅ Sin alteraciones detectadas (`alteration: true`)
- ✅ No está en listas de vigilancia (`watch_list: false`)

### ⚠️ Cotejo facial fallido

```json
{
  "verification": {
    "verification_status": "completed",
    "face_match": false,
    "liveness": true,
    "alteration": true
  },
  "face_match_score": 45.2
}
```
**Interpretación:** 
- ❌ La persona en el selfie NO coincide con la foto del documento (`face_match: false`)
- Aunque la prueba de vida es válida, hay discrepancia en la identidad

### ❌ Documento alterado detectado

```json
{
  "verification": {
    "verification_status": "completed",
    "face_match": true,
    "alteration": false,
    "template": true
  }
}
```
**Interpretación:** 
- ❌ Se detectaron alteraciones en el documento (`alteration: false`)
- Posible fraude o documento manipulado
- Aunque el cotejo facial coincide, el documento no es confiable

### ⚠️ En lista de vigilancia

```json
{
  "verification": {
    "verification_status": "completed",
    "face_match": true,
    "liveness": true,
    "watch_list": true
  }
}
```
**Interpretación:**
- ⚠️ La persona está en listas de vigilancia (`watch_list: true`)
- Requiere revisión adicional o escalación según tus políticas

## Errores comunes

**400 - Error en verificación:**
- `"La verificacion tuvo un error"` - El procesamiento falló (OCR, liveness, servicios externos)

**401 - Autenticación:**
- `"Missing Authorization Header"` - Falta token JWT
- `"Token has expired"` - Token expirado, renovar

**404 - No encontrado:**
- `"No se encontró el usuario con ID <user_id>"` - user_id no existe para la empresa

**Códigos HTTP:**
- `200` - Verificación completada
- `202` - Verificación en proceso (pending)
- `400` - Error en el procesamiento
- `401` - Token inválido
- `404` - user_id no encontrado

## Polling (Consulta periódica)

Si no usas webhook, consulta el estado periódicamente:

**Estrategia:**
1. Esperar 15-30 segundos después de crear la verificación
2. Consultar este endpoint
3. Si es 202 (pending), esperar 10-15 segundos y volver a consultar
4. Repetir hasta recibir 200 (completed) o 400 (error)
5. Timeout máximo: 2 minutos

**Ejemplo (Python):**
```python
import time, requests

def esperar_resultado(user_id, token, max_intentos=12):
    url = f"https://api.svi.becomedigital.net/api/v1/identity/{user_id}"
    headers = {"Authorization": f"Bearer {token}"}
    
    for i in range(max_intentos):
        response = requests.get(url, headers=headers)
        
        if response.status_code == 200:
            return response.json()  # Completado
        elif response.status_code == 202:
            time.sleep(10)  # Esperar 10 segundos
        else:
            raise Exception(f"Error: {response.status_code}")
    
    raise Exception("Timeout")
```

## Siguientes pasos

- [Listar todas las verificaciones →](verification-getall.md)
- [Re-verificación (autenticación) →](re-verification.md)
- [Crear nueva verificación →](verification-add.md)

