# Validación de Email

La funcionalidad de Email permite realizar solicitudes para validar y obtener información detallada sobre una dirección de correo electrónico, incluyendo la verificación de su estado, capacidad de entrega y riesgo de fraude asociado.

## POST `/email`

### Headers requeridos

```http
Content-Type: application/json
Authorization: Bearer <tu_jwt_token>
```

### Parámetros del request

```json
{
  "email": "usuario@example.com",
  "contract_id": 1
}
```

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| **email** | string | ✅ | Dirección de correo electrónico que se va a validar |
| **contract_id** | number | ✅ | Identificador único del contrato asociado a la validación |

### Ejemplo de solicitud

```bash
curl --location 'https://api.svi.becomedigital.net/api/v1/email' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer <tu_jwt_token>' \
--data '{
    "email": "usuario@hotmail.com",
    "contract_id": 1
}'
```

### Respuesta de la API

#### ✅ **200 OK - Validación exitosa**

```json
{
  "deliverability": "high",
  "dmarc_record": true,
  "dns_valid": true,
  "domain_age": "29 years ago",
  "first_seen": "1 day ago",
  "fraud_score": 15,
  "frequent_complainer": false,
  "honeypot": false,
  "recent_abuse": false,
  "risky_tld": false,
  "sanitized_email": false,
  "spam_trap_score": "none",
  "spf_record": true,
  "suspect": false
}
```

## Descripción de campos de respuesta

### Campos de validación

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `deliverability` | string | Nivel de capacidad de entrega: `high`, `medium`, `low` |
| `dmarc_record` | boolean | `true` si el dominio tiene un registro DMARC configurado |
| `dns_valid` | boolean | `true` si el dominio tiene registros DNS válidos |
| `domain_age` | string | Tiempo desde que se registró el dominio (ej: "29 years ago") |
| `first_seen` | string | Fecha aproximada en que el correo fue visto por primera vez |
| `fraud_score` | number | Puntuación de riesgo de fraude (0-100, valores altos = más riesgo) |
| `frequent_complainer` | boolean | `true` si el correo pertenece a un usuario que frecuentemente presenta quejas |
| `honeypot` | boolean | `true` si el correo electrónico es un honeypot (trampa para spam) |
| `recent_abuse` | boolean | `true` si el correo ha sido asociado recientemente con actividades abusivas |
| `risky_tld` | boolean | `true` si el dominio tiene un TLD considerado de alto riesgo |
| `sanitized_email` | boolean | `true` si el correo fue manipulado para cumplir con un formato válido |
| `spam_trap_score` | string | Nivel de probabilidad de ser trampa de spam: `none`, `low`, `medium`, `high` |
| `spf_record` | boolean | `true` si el dominio tiene un registro SPF configurado |
| `suspect` | boolean | `true` si el correo es considerado sospechoso |

## Interpretación de resultados

### ✅ Email válido y seguro

```json
{
  "deliverability": "high",
  "fraud_score": 10,
  "honeypot": false,
  "spam_trap_score": "none",
  "suspect": false,
  "dns_valid": true,
  "dmarc_record": true,
  "spf_record": true
}
```

**Interpretación:**
- ✅ Alta capacidad de entrega
- ✅ Bajo riesgo de fraude (score: 10)
- ✅ No es honeypot ni spam trap
- ✅ Dominio con configuración de seguridad adecuada

**Acción:** Aprobar

### ⚠️ Email con riesgo medio

```json
{
  "deliverability": "medium",
  "fraud_score": 55,
  "honeypot": false,
  "spam_trap_score": "low",
  "suspect": false,
  "recent_abuse": false
}
```

**Interpretación:**
- ⚠️ Capacidad de entrega media
- ⚠️ Riesgo de fraude moderado (score: 55)
- ⚠️ Baja probabilidad de ser spam trap

**Acción:** Revisar o solicitar confirmación por email

### ❌ Email de alto riesgo

```json
{
  "deliverability": "low",
  "fraud_score": 91,
  "honeypot": true,
  "spam_trap_score": "high",
  "suspect": true,
  "recent_abuse": true
}
```

**Interpretación:**
- ❌ Baja capacidad de entrega
- ❌ Alto riesgo de fraude (score: 91)
- ❌ Identificado como honeypot
- ❌ Actividad abusiva reciente

**Acción:** Rechazar

## Niveles de validación

### Fraud Score (Puntuación de fraude)

| Rango | Nivel de riesgo | Recomendación |
|-------|----------------|---------------|
| **0-25** | Bajo | ✅ Aprobar automáticamente |
| **26-50** | Medio-Bajo | ✅ Aprobar con monitoreo |
| **51-75** | Medio-Alto | ⚠️ Revisar antes de aprobar |
| **76-100** | Alto | ❌ Rechazar o validación adicional |

### Deliverability (Capacidad de entrega)

| Nivel | Descripción | Acción |
|-------|-------------|--------|
| **high** | Alta probabilidad de que el email sea válido y activo | ✅ Aprobar |
| **medium** | Capacidad de entrega incierta | ⚠️ Enviar email de confirmación |
| **low** | Baja probabilidad de entrega exitosa | ❌ Solicitar email alternativo |

### Spam Trap Score

| Nivel | Descripción | Acción |
|-------|-------------|--------|
| **none** | No es spam trap | ✅ Continuar |
| **low** | Baja probabilidad de spam trap | ⚠️ Monitorear |
| **medium** | Probabilidad media | ⚠️ Revisar |
| **high** | Alta probabilidad de spam trap | ❌ Rechazar |

## Indicadores de seguridad del dominio

### DMARC, SPF y DNS

| Indicador | Significado | Importancia |
|-----------|-------------|-------------|
| `dmarc_record: true` | El dominio tiene política DMARC | ✅ Mayor seguridad anti-suplantación |
| `spf_record: true` | El dominio tiene registro SPF | ✅ Validación de remitentes autorizada |
| `dns_valid: true` | Registros DNS válidos | ✅ Dominio configurado correctamente |

**Configuración ideal:**
```json
{
  "dmarc_record": true,
  "spf_record": true,
  "dns_valid": true
}
```

## Casos de uso

### 1. Validación en registro de usuarios

```python
def validar_email_registro(email):
    response = email_validation(email)
    result = response.json()
    
    # Verificar que no sea honeypot
    if result["honeypot"]:
        return False, "Email identificado como honeypot"
    
    # Verificar fraud score
    if result["fraud_score"] > 75:
        return False, "Email de alto riesgo"
    
    # Verificar deliverability
    if result["deliverability"] == "low":
        return False, "Email con baja capacidad de entrega"
    
    # Verificar que no sea spam trap
    if result["spam_trap_score"] in ["medium", "high"]:
        return False, "Posible spam trap"
    
    return True, "Email válido"
```

### 2. Detección de emails desechables

```python
def detectar_email_desechable(email):
    response = email_validation(email)
    result = response.json()
    
    # Dominios de emails temporales suelen tener:
    # - Domain age bajo
    # - Risky TLD
    # - Fraud score alto
    
    if result["risky_tld"] and result["fraud_score"] > 60:
        return True, "Posible email desechable"
    
    return False, "Email permanente"
```

### 3. Validación antes de campañas de email

```python
def validar_lista_emails(emails):
    resultados = {
        "validos": [],
        "revisar": [],
        "invalidos": []
    }
    
    for email in emails:
        response = email_validation(email)
        result = response.json()
        
        if result["deliverability"] == "high" and result["fraud_score"] < 30:
            resultados["validos"].append(email)
        elif result["deliverability"] == "medium":
            resultados["revisar"].append(email)
        else:
            resultados["invalidos"].append(email)
    
    return resultados
```

### 4. Análisis de riesgo completo

```python
def analizar_riesgo_email(email):
    response = email_validation(email)
    result = response.json()
    
    factores_riesgo = []
    
    if result["fraud_score"] > 70:
        factores_riesgo.append("Alto fraud score")
    
    if result["honeypot"]:
        factores_riesgo.append("Es honeypot")
    
    if result["spam_trap_score"] in ["medium", "high"]:
        factores_riesgo.append("Posible spam trap")
    
    if result["recent_abuse"]:
        factores_riesgo.append("Actividad abusiva reciente")
    
    if result["frequent_complainer"]:
        factores_riesgo.append("Usuario quejoso frecuente")
    
    if result["suspect"]:
        factores_riesgo.append("Marcado como sospechoso")
    
    return {
        "riesgo": "alto" if len(factores_riesgo) >= 2 else "medio" if len(factores_riesgo) == 1 else "bajo",
        "factores": factores_riesgo
    }
```

## Dominios comunes y sus características

### Proveedores confiables

Dominios como Gmail, Outlook, Yahoo típicamente tienen:
- ✅ `domain_age`: Muy antiguo (20+ años)
- ✅ `dmarc_record`: true
- ✅ `spf_record`: true
- ✅ `dns_valid`: true
- ✅ `risky_tld`: false

### Dominios temporales/desechables

- ⚠️ `domain_age`: Muy reciente
- ⚠️ `risky_tld`: Frecuentemente true
- ⚠️ `fraud_score`: Alto (70+)
- ⚠️ `deliverability`: low

## Flags de alerta

### 🚩 Honeypot = true

**Significado:** El email es una trampa para capturar spammers.

**Acción:** ❌ **NO ENVIAR emails** a esta dirección. Rechazar.

### 🚩 Spam Trap Score = high

**Significado:** Alta probabilidad de ser una trampa de spam.

**Acción:** ❌ Rechazar o remover de listas de email.

### 🚩 Recent Abuse = true

**Significado:** Email asociado con actividades abusivas recientes.

**Acción:** ⚠️ Aplicar validación adicional.

### 🚩 Frequent Complainer = true

**Significado:** Usuario marca emails como spam frecuentemente.

**Acción:** ⚠️ Evitar enviar emails promocionales.

## Formato del email

### Ejemplos válidos

- `usuario@example.com` ✅
- `nombre.apellido@empresa.co` ✅
- `user+tag@domain.com` ✅
- `123@numbers.com` ✅

### Ejemplos inválidos

- `usuario@` ❌
- `@domain.com` ❌
- `usuario domain.com` ❌
- `usuario@@domain.com` ❌

## Beneficios del servicio

- ✅ **Reducción de rebotes** al validar emails antes de enviar
- ✅ **Prevención de fraude** mediante análisis de riesgo
- ✅ **Protección de reputación** evitando spam traps y honeypots
- ✅ **Mejora de deliverability** filtrando emails inválidos
- ✅ **Ahorro de costos** en servicios de email marketing

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
  "msg": "Email inválido"
}
```

**Causa:** El formato del email no es válido.

### **412 - Precondition Failed**

```json
{
  "msg": "El contrato no contiene este servicio"
}
```

**Causa:** El servicio de validación de email no está habilitado en el contrato.

## Siguientes pasos

- [Validación Telco →](telco.md)
- [Verificación ANI Colombia →](ani-compliance.md)
- [Crear verificación de identidad →](verification-add.md)

