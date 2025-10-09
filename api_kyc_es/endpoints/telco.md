# Validación Telco

La funcionalidad de Telco permite realizar solicitudes para validar y obtener información detallada sobre un número de teléfono móvil de cualquier parte del mundo, utilizando servicios adicionales como el puntaje de riesgo, validación de contacto y verificación de cambios de SIM.

## POST `/telcoValidation`

### Headers requeridos

```http
Content-Type: application/json
Authorization: Bearer <tu_jwt_token>
```

### Parámetros del request

```json
{
  "phone_number": "573000000000",
  "contract_id": 1
}
```

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| **phone_number** | string | ✅ | Número de teléfono que se desea validar, incluyendo el código del país (formato E.164 recomendado) |
| **contract_id** | number | ✅ | Identificador único del contrato |

### Ejemplo de solicitud

```bash
curl --location 'https://api.svi.becomedigital.net/api/v1/telcoValidation' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer <tu_jwt_token>' \
--data '{
    "phone_number": "573117631081",
    "contract_id": 1
}'
```

### Respuesta de la API

#### ✅ **200 OK - Validación exitosa**

```json
{
  "phone_number": "573117631081",
  "telesign_phone_id": {
    "blocklisting": {
      "block_code": 0,
      "block_description": "Not blocked",
      "blocked": false
    },
    "carrier": {
      "name": "Comcel"
    },
    "contact": {
      "address1": "",
      "address2": "",
      "address3": "",
      "address4": "",
      "city": "TULUÁ",
      "country": "CO",
      "date_of_birth": null,
      "email_address": "",
      "first_name": "CRISTHIAN",
      "last_name": "SANCHEZ",
      "state_province": "VALLE DEL CAUCA",
      "status": {
        "code": 2800,
        "description": "Request successfully completed"
      },
      "zip_postal_code": ""
    },
    "location": {
      "city": "Countrywide",
      "coordinates": {
        "latitude": null,
        "longitude": null
      },
      "country": {
        "iso2": "CO",
        "iso3": "COL",
        "name": "Colombia"
      },
      "time_zone": {
        "name": null,
        "utc_offset_max": "-5",
        "utc_offset_min": "-5"
      }
    },
    "phone_type": {
      "code": "2",
      "description": "MOBILE"
    },
    "sim_swap": {
      "risk_indicator": 1,
      "status": {
        "code": 2800,
        "description": "Request successfully completed"
      },
      "swap_date": "2022-11-26",
      "swap_time": "15:55:54"
    },
    "status": {
      "code": 300,
      "description": "Transaction successfully completed"
    }
  },
  "telesign_score": {
    "risk": {
      "level": "medium-low",
      "recommendation": "allow",
      "score": 301
    },
    "risk_insights": {
      "a2p": [22007, 20011, 20101],
      "category": [10010],
      "p2p": [30201],
      "status": 800
    }
  }
}
```

## Descripción de campos de respuesta

### Campo principal

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `phone_number` | string | Número validado, incluyendo el código de país |

### Objeto `telesign_phone_id`

#### `blocklisting`

Información sobre si el número está en listas de bloqueo:

| Campo | Descripción |
|-------|-------------|
| `block_code` | Código de bloqueo (0 = No bloqueado) |
| `block_description` | Descripción del estado de bloqueo |
| `blocked` | `true` si está bloqueado, `false` si no |

#### `carrier`

| Campo | Descripción |
|-------|-------------|
| `name` | Nombre del operador móvil (ej: "Comcel", "Movistar", "Claro") |

#### `contact`

Información adicional del contacto asociado al número (si está disponible):

| Campo | Descripción |
|-------|-------------|
| `first_name` | Primer nombre del titular |
| `last_name` | Apellido del titular |
| `city` | Ciudad de registro |
| `state_province` | Departamento/estado/provincia |
| `country` | Código ISO-2 del país |
| `email_address` | Correo electrónico (si disponible) |

#### `location`

Detalles geográficos del número:

| Campo | Descripción |
|-------|-------------|
| `country.name` | Nombre del país |
| `country.iso2` | Código ISO-2 del país |
| `country.iso3` | Código ISO-3 del país |
| `city` | Ciudad asociada al número |
| `time_zone.utc_offset_min` | Offset UTC mínimo |
| `time_zone.utc_offset_max` | Offset UTC máximo |

#### `phone_type`

| Campo | Descripción |
|-------|-------------|
| `code` | Código del tipo de línea |
| `description` | Descripción del tipo: `MOBILE`, `LANDLINE`, `VOIP`, etc. |

**Tipos comunes:**
- `MOBILE` (código 2) - Línea móvil
- `LANDLINE` (código 1) - Línea fija
- `VOIP` (código 3) - VoIP

#### `sim_swap`

Información sobre cambios recientes en la tarjeta SIM:

| Campo | Descripción |
|-------|-------------|
| `risk_indicator` | Indicador de riesgo (1 = bajo, valores más altos = mayor riesgo) |
| `swap_date` | Fecha del último cambio de SIM (formato YYYY-MM-DD) |
| `swap_time` | Hora del último cambio (formato HH:MM:SS) |
| `status.code` | Código de estado de la consulta |
| `status.description` | Descripción del estado |

### Objeto `telesign_score`

#### `risk`

Análisis de riesgo del número:

| Campo | Descripción |
|-------|-------------|
| `level` | Nivel de riesgo: `low`, `medium-low`, `medium`, `medium-high`, `high` |
| `recommendation` | Recomendación: `allow`, `flag`, `block` |
| `score` | Puntuación de riesgo (0-999, mayor = más riesgo) |

#### `risk_insights`

Códigos de insights de riesgo por categoría:

| Campo | Descripción |
|-------|-------------|
| `a2p` | Insights de aplicación a persona (Application-to-Person) |
| `category` | Categoría general de riesgo |
| `p2p` | Insights de persona a persona (Person-to-Person) |
| `email` | Insights relacionados con email |
| `ip` | Insights relacionados con IP |
| `status` | Estado general de los insights |

## Interpretación de resultados

### ✅ Número válido y de bajo riesgo

```json
{
  "telesign_phone_id": {
    "blocklisting": { "blocked": false },
    "phone_type": { "description": "MOBILE" },
    "sim_swap": { "risk_indicator": 1 }
  },
  "telesign_score": {
    "risk": {
      "level": "low",
      "recommendation": "allow",
      "score": 150
    }
  }
}
```

**Interpretación:**
- ✅ No está bloqueado
- ✅ Es un número móvil válido
- ✅ Sin cambios recientes de SIM sospechosos
- ✅ Bajo riesgo general

**Acción:** Aprobar

### ⚠️ Número con riesgo medio

```json
{
  "telesign_score": {
    "risk": {
      "level": "medium",
      "recommendation": "flag",
      "score": 500
    }
  }
}
```

**Interpretación:**
- ⚠️ Riesgo medio detectado
- ⚠️ Se recomienda revisar

**Acción:** Solicitar validación adicional

### ❌ Número bloqueado o de alto riesgo

```json
{
  "telesign_phone_id": {
    "blocklisting": { "blocked": true }
  },
  "telesign_score": {
    "risk": {
      "level": "high",
      "recommendation": "block",
      "score": 850
    }
  }
}
```

**Interpretación:**
- ❌ Número bloqueado
- ❌ Alto riesgo de fraude

**Acción:** Rechazar

## Análisis de SIM Swap

### Cambio reciente de SIM (riesgo alto)

```json
{
  "sim_swap": {
    "risk_indicator": 5,
    "swap_date": "2025-01-08",
    "swap_time": "14:30:00"
  }
}
```

**Interpretación:**
- ⚠️ SIM cambiada hace pocos días
- ⚠️ Posible ataque de SIM swapping
- ⚠️ Requiere validación adicional

### Sin cambios recientes (riesgo bajo)

```json
{
  "sim_swap": {
    "risk_indicator": 1,
    "swap_date": "2022-11-26",
    "swap_time": "15:55:54"
  }
}
```

**Interpretación:**
- ✅ Último cambio hace más de 2 años
- ✅ Bajo riesgo de SIM swapping

## Niveles de riesgo

### Puntuación de riesgo (score)

| Rango | Nivel | Recomendación |
|-------|-------|---------------|
| **0-200** | Low | ✅ Allow - Aprobar automáticamente |
| **201-400** | Medium-Low | ✅ Allow - Aprobar con monitoreo |
| **401-600** | Medium | ⚠️ Flag - Revisar antes de aprobar |
| **601-800** | Medium-High | ⚠️ Flag - Validación adicional requerida |
| **801-999** | High | ❌ Block - Rechazar |

### Indicador de riesgo SIM Swap

| Valor | Interpretación |
|-------|----------------|
| **1** | Sin cambios recientes o cambio antiguo (bajo riesgo) |
| **2-3** | Cambio moderadamente reciente (riesgo medio) |
| **4-5** | Cambio muy reciente (alto riesgo) |

## Casos de uso

### 1. Validación en registro de usuarios

```python
def validar_telefono_registro(phone_number):
    response = telco_validation(phone_number)
    result = response.json()
    
    # Verificar que no esté bloqueado
    if result["telesign_phone_id"]["blocklisting"]["blocked"]:
        return False, "Número bloqueado"
    
    # Verificar nivel de riesgo
    risk_score = result["telesign_score"]["risk"]["score"]
    if risk_score > 600:
        return False, "Número de alto riesgo"
    
    # Verificar tipo de teléfono
    if result["telesign_phone_id"]["phone_type"]["description"] != "MOBILE":
        return False, "Debe ser un número móvil"
    
    return True, "Número válido"
```

### 2. Detección de fraude en transacciones

```python
def detectar_fraude_sim_swap(phone_number):
    response = telco_validation(phone_number)
    sim_swap = response.json()["telesign_phone_id"]["sim_swap"]
    
    # Verificar si hubo cambio de SIM reciente
    swap_date = datetime.strptime(sim_swap["swap_date"], "%Y-%m-%d")
    dias_desde_swap = (datetime.now() - swap_date).days
    
    if dias_desde_swap < 7 and sim_swap["risk_indicator"] >= 4:
        return True, "Posible ataque SIM swap detectado"
    
    return False, "Sin indicios de SIM swap"
```

### 3. Validación de operador

```python
def validar_operador(phone_number, operadores_permitidos):
    response = telco_validation(phone_number)
    carrier = response.json()["telesign_phone_id"]["carrier"]["name"]
    
    if carrier not in operadores_permitidos:
        return False, f"Operador {carrier} no permitido"
    
    return True, f"Operador {carrier} válido"
```

## Formato del número telefónico

### Formato recomendado: E.164

**Estructura:** `[+][código de país][número]`

**Ejemplos válidos:**
- `573117631081` (Colombia) ✅
- `+573117631081` (Colombia con +) ✅
- `5215512345678` (México) ✅
- `12125551234` (USA) ✅

**Ejemplos inválidos:**
- `311 763 1081` (con espacios) ❌
- `(311) 763-1081` (con paréntesis y guiones) ❌
- `3117631081` (sin código de país) ❌

## Beneficios del servicio

- ✅ **Verificación en tiempo real** del número telefónico
- ✅ **Detección de SIM swap** para prevenir fraudes
- ✅ **Análisis de riesgo** basado en múltiples factores
- ✅ **Validación de operadora** y tipo de línea
- ✅ **Información de contacto** (cuando disponible)
- ✅ **Geolocalización** del número

## Manejo de errores

### **401 - Unauthorized**

```json
{
  "msg": "Missing Authorization Header"
}
```

### **400 - Bad Request**

```json
{
  "msg": "Número de teléfono inválido"
}
```

**Causa:** El formato del número no es válido.

### **412 - Precondition Failed**

```json
{
  "msg": "El contrato no contiene este servicio"
}
```

**Causa:** El servicio Telco no está habilitado en el contrato.

## Siguientes pasos

- [Validación de Email →](email.md)
- [Verificación ANI Colombia →](ani-compliance.md)
- [Crear verificación de identidad →](verification-add.md)

