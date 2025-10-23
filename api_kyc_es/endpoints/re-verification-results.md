# Resultados de Re-verificación

Una vez enviada la solicitud de re-verificación (cotejo facial y opcionalmente liveness), puedes consultar el resultado definitivo utilizando el `executionId` proporcionado en la respuesta de la primera etapa.

## Proceso de consulta de resultados

1. **Obtener executionId:** Guarda el `executionId` retornado por el endpoint `/matches`
2. **Consultar resultado:** Utiliza el endpoint `/matches/check` para obtener el resultado final
3. **Procesar respuesta:** Interpreta los resultados según tus umbrales de confianza

## Flujo de consulta de resultados

```
Cliente → [POST /matches con imagen] → API
API → [Procesa validación] → Retorna executionId
Cliente → [POST /matches/check con executionId] → API
API → [Busca resultado] → Retorna datos de validación
```

## POST `/matches/check`

### Descripción del servicio

Este endpoint permite consultar el resultado de una validación facial previamente realizada. Es un endpoint de consulta que busca una validación específica usando el `executionId` que se generó durante el proceso de comparación facial.

### Headers requeridos

```http
Content-Type: application/json
Authorization: Bearer <tu_jwt_token>
```

### Parámetros del Request

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

### Ejemplos de request

**Consulta de resultados de re-verificación:**

```bash
curl --location 'https://api.svi.becomedigital.net/api/v1/matches/check' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer tu_jwt_token_aqui' \
--data '{
    "contract_id": 1,
    "executionId": "fd5075b1-74e4-4b23-ac94-ddd9b36ecc33"
}'
```

## Consideraciones de seguridad

- 🔒 **Almacena de forma segura** los `executionId` en tu base de datos
- ⏰ **Los executionId expiran** después de un tiempo determinado
- 🔄 **Implementa validación** de que el `executionId` pertenece a tu contrato
- 🔑 **Nunca expongas** los `executionId` en URLs o logs públicos
- 📝 **Registra todas las consultas** para auditoría de seguridad

## Flujo del Proceso

1. **Validación de parámetros** → 400 si faltan `contract_id` o `executionId`
2. **Búsqueda del contrato** → 404 si no existe
3. **Verificación de permisos** → 400 si el servicio no está activo
4. **Búsqueda de validación** → 404 si no se encuentra la validación
5. **Búsqueda de identidad** → Se obtiene el `user_id` asociado
6. **Respuesta exitosa** → 200 con todos los datos de la validación

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

| Aspecto | `/matches` | `/matches/check` |
|---------|------------|------------------|
| **Propósito** | Realizar validación | Consultar resultado |
| **Entrada** | Imagen + datos | executionId + contract_id |
| **Proceso** | Comparación facial | Búsqueda en BD |
| **Resultado** | Nueva validación | Datos de validación existente |
| **Content-Type** | multipart/form-data | application/json |
| **Parámetros** | user_id, image, contract_id | executionId, contract_id |
| **Cuándo usar** | Primera verificación | Consulta posterior |
| **Incluye timestamp** | ❌ No | ✅ Sí |

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

### 1. Verificar el estado de una validación facial asíncrona

```python
# Al momento de la re-verificación
response = requests.post("/matches", data=formdata)
execution_id = response.json()["executionId"]

# Guardar execution_id en base de datos...
transaction.verification_execution_id = execution_id
transaction.save()

# Después, cuando necesites el resultado
result = requests.post("/matches/check", json={
    "contract_id": 1,
    "executionId": execution_id
})

if result.json()["result"]:
    print("Validación exitosa")
```

### 2. Obtener resultados de validaciones previas

```python
# Recuperar execution_id de logs o base de datos
execution_id = get_execution_id_from_database(transaction_id)

# Consultar resultado
result = requests.post("/matches/check", json={
    "contract_id": 1,
    "executionId": execution_id
})

# Procesar resultado
if result.json()["result"]:
    approve_transaction()
else:
    reject_transaction()
```

### 3. Auditoría de procesos de verificación

```python
# Consultar resultados históricos para auditoría
execution_id = get_execution_id_from_audit_log(verification_id)

# Consultar resultado
result = requests.post("/matches/check", json={
    "contract_id": contract_id,
    "executionId": execution_id
})

# Guardar en logs de auditoría
save_audit_log({
    "execution_id": execution_id,
    "result": result.json()["result"],
    "confidence": result.json()["confidence"],
    "timestamp": result.json()["timestamp"]
})
```

### 4. Integración con sistemas que necesitan consultar resultados

```python
# Sistema de monitoreo de transacciones
def check_verification_status(execution_id, contract_id):
    try:
        result = requests.post("/matches/check", json={
            "contract_id": contract_id,
            "executionId": execution_id
        }, timeout=30)
        
        return {
            "status": "completed",
            "result": result.json()["result"],
            "confidence": result.json()["confidence"]
        }
    except requests.Timeout:
        return {"status": "timeout"}
    except requests.RequestException:
        return {"status": "error"}
```

## Manejo de errores

### **400 - Bad Request - Errores de Validación**

#### 1. Campos requeridos faltantes

```json
{
  "msg": "Faltan campos requeridos: contract_id, executionId"
}
```

**Causa:** No se enviaron los campos obligatorios.

#### 2. Servicio inactivo

```json
{
  "msg": "El servicio de re-verificación no está activo para este contrato"
}
```

**Causa:** El contrato no tiene habilitado el servicio de re-verificación.

### **404 - Not Found - Recurso No Encontrado**

#### 1. Contrato no encontrado

```json
{
  "msg": "Contrato no encontrado"
}
```

**Causa:** El `contract_id` no existe en el sistema.

#### 2. Validación no encontrada

```json
{
  "msg": "Execution ID no encontrado"
}
```

**Causa:** El `executionId` no existe o es inválido.

### **500 - Internal Server Error - Errores del Servidor**

#### 1. Error general del servidor

```json
{
  "msg": "Error interno del servidor"
}
```

**Causa:** Error técnico durante el procesamiento.

### **401 - Unauthorized**

```json
{
  "msg": "Missing Authorization Header"
}
```

**Causa:** Token faltante o inválido.

**Solución:** Incluir header `Authorization: Bearer <token>` válido.

## Uso del executionId en las consultas

El `executionId` debe incluirse en el cuerpo de la petición JSON:

```json
{
  "contract_id": 1,
  "executionId": "fd5075b1-74e4-4b23-ac94-ddd9b36ecc33"
}
```

Todas las consultas al endpoint `/matches/check` deben incluir estos campos obligatorios.

## Flujo asíncrono

Este endpoint es complementario al `/matches` y permite un flujo asíncrono donde puedes realizar la validación y luego consultar el resultado por separado.

### Flujo recomendado:

1. **Realizar validación** → POST `/matches` (con imagen)
2. **Obtener executionId** → Guardar en base de datos
3. **Consultar resultado** → POST `/matches/check` (con executionId)
4. **Procesar resultado** → Aplicar lógica de negocio

## Mejores prácticas

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

### 5. Manejo de errores robusto

```python
# ✅ Implementar reintentos y manejo de errores
def check_verification_with_retry(execution_id, contract_id, max_retries=3):
    for attempt in range(max_retries):
        try:
            result = requests.post("/matches/check", json={
                "contract_id": contract_id,
                "executionId": execution_id
            }, timeout=30)
            
            if result.status_code == 200:
                return result.json()
            elif result.status_code == 404:
                # No existe la validación
                return None
            else:
                # Error del servidor, reintentar
                if attempt < max_retries - 1:
                    time.sleep(2 ** attempt)  # Backoff exponencial
                    continue
                else:
                    raise Exception(f"Error después de {max_retries} intentos")
                    
        except requests.Timeout:
            if attempt < max_retries - 1:
                time.sleep(2 ** attempt)
                continue
            else:
                raise Exception("Timeout después de múltiples intentos")
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


## Siguientes pasos

- [Realizar re-verificación →](re-verification.md)
- [Ver verificación completa →](verification-add.md)
- [Consultar resultados de verificación →](verification-results.md)

