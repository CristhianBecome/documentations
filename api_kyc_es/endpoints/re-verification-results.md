# Resultados de Re-verificación

Una vez enviada la solicitud de re-verificación (cotejo facial y opcionalmente liveness), puedes consultar el resultado definitivo utilizando el `executionId` proporcionado en la respuesta de la primera etapa.

## POST `/matches/check`

### Headers requeridos

```http
Content-Type: application/json
Authorization: Bearer <tu_jwt_token>
```

### Parámetros del request

```json
{
  "contract_id": 0,
  "executionId": "fd5075b1-74e4-4b23-ac94-ddd9b36ecc33"
}
```

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| **contract_id** | number | ✅ | ID del contrato utilizado en la re-verificación |
| **executionId** | string | ✅ | Identificador único retornado en la respuesta del endpoint `/matches` |

### Ejemplo de solicitud

```bash
curl --location 'https://api.svi.becomedigital.net/api/v1/matches/check' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer <tu_jwt_token>' \
--data '{
    "contract_id": 1,
    "executionId": "fd5075b1-74e4-4b23-ac94-ddd9b36ecc33"
}'
```

### Respuesta de la API

#### ✅ **200 OK - Resultados disponibles**

```json
{
  "company": "YOUR_COMPANY_ACCOUNT_NAME",
  "confidence": 95.4115853309631348,
  "executionId": "fd5075b1-74e4-4b23-ac94-ddd9b36ecc33",
  "liveness": 99.89696502685547,
  "result": true,
  "timestamp": "2025-01-27 14:18:58",
  "user_id": "usuario_12345_1699123456"
}
```

### Descripción de campos de respuesta

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `company` | string | Nombre de la compañía responsable del proceso |
| `confidence` | number | Nivel de confianza (0-100) obtenido en la comparación facial |
| `executionId` | string | Identificador que vincula esta respuesta con la solicitud inicial |
| `liveness` | number | Valor que indica el resultado de la prueba de vida (0-100) |
| `result` | boolean | `true` = Verificación satisfactoria, `false` = Verificación fallida |
| `timestamp` | string | Fecha y hora en que se procesó la verificación |
| `user_id` | string | Identificador del usuario para el cual se realizó la validación |

## Diferencias con el endpoint `/matches`

| Característica | `/matches` | `/matches/check` |
|----------------|------------|------------------|
| **Tipo de request** | FormData (con imagen) | JSON (sin imagen) |
| **Cuándo usar** | Al realizar la verificación inicial | Para consultar resultados posteriormente |
| **Respuesta** | Inmediata (puede ser preliminar) | Resultado final procesado |
| **Campo adicional** | - | `timestamp` |

## Interpretación de resultados

### ✅ Autenticación exitosa

```json
{
  "result": true,
  "confidence": 95.5,
  "liveness": 99.8
}
```

**Interpretación:**
- ✅ La persona coincide con el registro (`result: true`)
- ✅ Alta confianza en la comparación (`confidence: 95.5`)
- ✅ Alta probabilidad de ser una persona real (`liveness: 99.8`)

**Acción recomendada:** Permitir el acceso/transacción

### ❌ Autenticación fallida

```json
{
  "result": false,
  "confidence": 2.4,
  "liveness": 99.8
}
```

**Interpretación:**
- ❌ La persona NO coincide con el registro (`result: false`)
- ❌ Baja confianza en la comparación (`confidence: 2.4`)
- ℹ️ Aunque la prueba de vida es válida, el rostro no corresponde

**Acción recomendada:** Denegar el acceso/transacción

### ⚠️ Caso de atención

```json
{
  "result": true,
  "confidence": 72.5,
  "liveness": 88.3
}
```

**Interpretación:**
- ⚠️ Coincidencia con confianza media (`confidence: 72.5`)
- ⚠️ Prueba de vida con puntuación media (`liveness: 88.3`)

**Acción recomendada:** 
- Aplicar validación adicional
- Solicitar segundo intento
- Escalar a revisión manual según políticas de riesgo

## Umbrales de decisión

### Configuración conservadora (alta seguridad)

```javascript
if (result === true && confidence >= 90 && liveness >= 95) {
  // ✅ Aprobar
} else if (result === true && confidence >= 80 && liveness >= 90) {
  // ⚠️ Revisión adicional
} else {
  // ❌ Rechazar
}
```

### Configuración moderada (balance)

```javascript
if (result === true && confidence >= 80 && liveness >= 90) {
  // ✅ Aprobar
} else if (result === true && confidence >= 70 && liveness >= 85) {
  // ⚠️ Revisión adicional
} else {
  // ❌ Rechazar
}
```

### Configuración permisiva (mejor UX)

```javascript
if (result === true && confidence >= 70 && liveness >= 85) {
  // ✅ Aprobar
} else if (result === true && confidence >= 60 && liveness >= 75) {
  // ⚠️ Revisión adicional
} else {
  // ❌ Rechazar
}
```

> **Nota:** Ajusta los umbrales según el nivel de riesgo de tu aplicación y las regulaciones aplicables.

## Casos de uso

### 1. Consulta posterior

Si necesitas guardar el `executionId` y consultar el resultado más tarde:

```python
# Al momento de la re-verificación
response = requests.post("/matches", data=formdata)
execution_id = response.json()["executionId"]

# Guardar execution_id en base de datos...

# Después, cuando necesites el resultado
result = requests.post("/matches/check", json={
    "contract_id": 1,
    "executionId": execution_id
})
```

### 2. Auditoría y registros

Consultar resultados históricos de re-verificaciones para auditoría:

```python
# Recuperar execution_id de logs o base de datos
execution_id = get_execution_id_from_database(transaction_id)

# Consultar resultado
result = requests.post("/matches/check", json={
    "contract_id": 1,
    "executionId": execution_id
})

# Guardar en logs de auditoría
save_audit_log(result.json())
```

### 3. Verificación de transacciones pasadas

Revisar si una transacción pasada fue autenticada correctamente:

```python
# Obtener execution_id asociado a la transacción
execution_id = transaction.get_verification_id()

# Consultar resultado
verification = requests.post("/matches/check", json={
    "contract_id": contract_id,
    "executionId": execution_id
})

if verification.json()["result"]:
    print("Transacción autenticada correctamente")
```

## Manejo de errores

### **401 - Unauthorized**

```json
{
  "msg": "Missing Authorization Header"
}
```

**Causa:** Token faltante o inválido.

**Solución:** Incluir header `Authorization: Bearer <token>` válido.

### **404 - Not Found**

```json
{
  "msg": "Execution ID no encontrado"
}
```

**Causa:** El `executionId` no existe o es inválido.

**Solución:** Verificar que el `executionId` sea correcto.

### **400 - Bad Request**

```json
{
  "msg": "Parámetros inválidos"
}
```

**Causa:** Faltan campos requeridos o tienen formato incorrecto.

**Solución:** Verificar que se envíen `contract_id` y `executionId`.

## Buenas prácticas

### 1. Almacenamiento de execution_id

```python
# ✅ Buena práctica: Guardar execution_id relacionado con la transacción
transaction.verification_execution_id = execution_id
transaction.save()
```

### 2. Timeout de consultas

```python
# ✅ Implementar timeout para consultas
try:
    response = requests.post(url, json=data, timeout=30)
except requests.Timeout:
    # Manejar timeout
    log_error("Timeout consultando resultados")
```

### 3. Logs de auditoría

```python
# ✅ Registrar todas las consultas de verificación
log_verification_check({
    "execution_id": execution_id,
    "result": result["result"],
    "confidence": result["confidence"],
    "timestamp": datetime.now(),
    "user_id": result["user_id"]
})
```

### 4. Caché de resultados

```python
# ✅ Cachear resultados para evitar consultas repetidas
cache_key = f"verification:{execution_id}"
cached_result = cache.get(cache_key)

if cached_result:
    return cached_result

result = requests.post("/matches/check", json=data)
cache.set(cache_key, result.json(), timeout=3600)  # 1 hora
```

## Campo timestamp

El campo `timestamp` indica cuándo se procesó la verificación:

```json
{
  "timestamp": "2025-01-27 14:18:58"
}
```

**Formato:** `YYYY-MM-DD HH:MM:SS`

**Utilidad:**
- Auditoría de cuándo se realizó la verificación
- Validar que la verificación es reciente
- Detectar intentos de replay

## Diferencias clave respecto a `/matches`

| Aspecto | `/matches` | `/matches/check` |
|---------|------------|------------------|
| Envía imagen | ✅ Sí | ❌ No |
| Content-Type | multipart/form-data | application/json |
| Parámetros | user_id, image, contract_id | executionId, contract_id |
| Cuándo usar | Primera verificación | Consulta posterior |
| Incluye timestamp | ❌ No | ✅ Sí |

## Siguientes pasos

- [Realizar re-verificación →](re-verification.md)
- [Ver verificación completa →](verification-add.md)
- [Consultar resultados de verificación →](verification-results.md)

