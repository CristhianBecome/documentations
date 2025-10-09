# Verificación de Identidad - Consultar Resultados

Este endpoint devuelve el resultado de una verificación individual utilizando el `user_id` definido al momento de crear la validación.

## GET `/identity/<user_id>`

### Headers requeridos

```http
Authorization: Bearer <tu_jwt_token>
```

### Parámetros de URL

| Parámetro | Tipo | Descripción |
|-----------|------|-------------|
| **user_id** | string | Identificador único del usuario definido al crear la verificación |

### Ejemplo de solicitud

```bash
curl --location 'https://api.svi.becomedigital.net/api/v1/identity/usuario_12345_1699123456' \
--header 'Authorization: Bearer <tu_jwt_token>'
```

## Tiempos de procesamiento

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

### Respuestas de la API

#### ✅ **200 OK - Verificación completada**

```json
{
  "id": 2,
  "created_at": "Sat, 25 Jul 2020 15:40:00 GMT",
  "company": "Acc_Demo",
  "fullname": "JUAN CARLOS GARCIA LOPEZ",
  "birth": "Fri, 03 Jul 1998 00:00:00 GMT",
  "document_type": "national-id",
  "document_number": "12345678",
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
}
```

#### ⏳ **202 Accepted - Verificación pendiente**

```json
{
  "id": 2,
  "created_at": "Sat, 25 Jul 2020 15:40:00 GMT",
  "company": "Acc_Demo",
  "fullname": "JUAN CARLOS GARCIA LOPEZ",
  "birth": "Fri, 03 Jul 1998 00:00:00 GMT",
  "document_type": "national-id",
  "document_number": "12345678",
  "verification": {
    "face_match": true,
    "template": true,
    "alteration": true,
    "watch_list": true
  },
  "comply_advantage": {
    "comply_advantage_result": null,
    "comply_advantage_url": null
  },
  "verification_status": "pending"
}
```

## Interpretación de la respuesta

### Campos principales

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | number | ID interno de la verificación |
| `created_at` | string | Fecha y hora de creación de la verificación |
| `company` | string | Nombre de la empresa/cuenta |
| `fullname` | string | Nombre completo extraído del documento mediante OCR |
| `birth` | string | Fecha de nacimiento extraída del documento |
| `document_type` | string | Tipo de documento validado |
| `document_number` | string | Número de documento extraído |

### Objeto `verification`

El objeto `verification` contiene el listado de validaciones realizadas sobre los documentos procesados.

#### Flags de verificación

Un valor `true` indica que la validación pasó satisfactoriamente.  
Un valor `false` indica que esa revisión no fue válida.  
Un valor `null` indica que esa validación no se pudo realizar o no aplica según el contrato.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `verification_status` | string | Estado general: `pending`, `completed`, `error` |
| `face_match` | boolean/null | Cotejo facial entre selfie y documento |
| `liveness` | boolean/null | Prueba de vida del video/selfie |
| `liveness_doc` | boolean/null | Detección de documento auténtico vs foto de documento |
| `alteration` | boolean/null | Detección de alteraciones en el documento |
| `template` | boolean/null | Validación de plantilla del documento |
| `estimated_age` | boolean/null | Validación de edad estimada vs edad del documento |
| `ip_validation` | boolean/null | Validación de dirección IP |
| `one_to_many_result` | boolean/null | Resultado de comparación 1:N (lista negra) |
| `watch_list` | boolean/null | Verificación contra listas de vigilancia |
| `ubica` | boolean/null | Resultado de servicio Ubica (si aplica) |

#### Estados de verificación

- **`pending`** - La verificación está en proceso
- **`completed`** - La verificación se completó
- **`error`** - Hubo un error durante la verificación

### Objeto `blacklist`

Resultado de la verificación contra listas negras biométricas.

| Campo | Descripción |
|-------|-------------|
| `result` | `false` = No está en lista negra, `true` = Coincide con lista negra |
| `score` | Puntuación de similitud (0-1, donde 1 es coincidencia exacta) |
| `id` | ID del registro en la base de datos |

### Objeto `comply_advantage`

Resultado de verificación contra listas de sanciones internacionales (si está contratado).

| Campo | Descripción |
|-------|-------------|
| `comply_advantage_result` | Resultado de la verificación |
| `comply_advantage_url` | URL con detalles del resultado |

## Interpretación de resultados

### ✅ Verificación exitosa

Cuando la verificación está en estado `completed` y los flags contratados tienen valor `true`:

```json
{
  "verification": {
    "verification_status": "completed",
    "face_match": true,
    "liveness": true,
    "alteration": true,
    "template": true
  }
}
```

**Interpretación:** 
- Prueba de vida válida (`liveness: true`)
- Cotejo facial exitoso (`face_match: true`)
- No se detectaron alteraciones (`alteration: true`)
- Plantilla del documento válida (`template: true`)

### ⚠️ Verificación con problemas

```json
{
  "verification": {
    "verification_status": "completed",
    "face_match": false,
    "liveness": true,
    "alteration": true
  }
}
```

**Interpretación:** El cotejo facial falló (`face_match: false`), lo que indica que la persona en el selfie no coincide con la foto del documento.

### ❌ Verificación incompleta

Cuando `verification_status` es `completed` pero algún flag es `null`:

```json
{
  "verification": {
    "verification_status": "completed",
    "face_match": null,
    "liveness": null,
    "alteration": true
  }
}
```

**Interpretación:** No se pudo realizar la prueba de vida ni el cotejo facial satisfactoriamente. Puede deberse a problemas con el video/selfie enviado.

## Manejo de errores

### **400 - Bad Request**

**Consulta exitosa pero con errores de verificación:**

```json
{
  "id": 5,
  "Company": "Become Digital",
  "verification": {
    "verification_status": "La verificacion tuvo un error"
  }
}
```

**Causa:** La verificación encontró errores durante el procesamiento.

### **401 - Unauthorized**

```json
{
  "msg": "Missing Authorization Header"
}
```

**Causa:** No se incluyó el token JWT o el token es inválido/expirado.

### **404 - Not Found**

```json
{
  "msg": "No se encontró el usuario con ID <user_id> para la compañía <company_id>"
}
```

**Causa:** No existe una verificación con ese `user_id` para la empresa autenticada.

**O bien:**

```json
{
  "error": "404 Not Found: The requested URL was not found on the server. If you entered the URL manually please check your spelling and try again.",
  "apimsg": "Resource not found."
}
```

## Códigos de estado HTTP

| Código | Significado |
|--------|-------------|
| **200** | La validación se ha completado correctamente |
| **202** | La validación está pendiente de procesamiento |
| **400** | Hubo un error durante el proceso de validación |
| **401** | Token faltante o inválido |
| **404** | No se encontró la validación con el `user_id` proporcionado |

## Polling recomendado

Dado que el proceso es asíncrono, se recomienda:

1. **Esperar inicial:** 5-10 segundos después de crear la verificación
2. **Consultar estado:** Realizar request a este endpoint
3. **Si es 202 (pending):** Esperar 3-5 segundos y volver a consultar
4. **Repetir:** Hasta recibir 200 (completed) o 400 (error)
5. **Timeout:** Máximo 2-3 minutos de espera

### Ejemplo de lógica de polling

```python
import time
import requests

def consultar_verificacion(user_id, token, max_intentos=30):
    url = f"https://api.svi.becomedigital.net/api/v1/identity/{user_id}"
    headers = {"Authorization": f"Bearer {token}"}
    
    for intento in range(max_intentos):
        response = requests.get(url, headers=headers)
        
        if response.status_code == 200:
            return response.json()  # Verificación completada
        elif response.status_code == 202:
            time.sleep(5)  # Esperar 5 segundos
            continue
        else:
            raise Exception(f"Error: {response.status_code}")
    
    raise Exception("Timeout: verificación no completada")
```

### Siguientes pasos

- [Listar todas las verificaciones →](verification-getall.md)
- [Re-verificación para autenticación →](re-verification.md)
- [Crear nueva verificación →](verification-add.md)

