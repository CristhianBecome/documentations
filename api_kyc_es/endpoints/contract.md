# Consulta de Contrato

Este endpoint retorna la configuración de un contrato específico, siempre y cuando pertenezca a la compañía que lo consulta.

## GET `/contract/<contract_id>`

### Headers requeridos

```http
Authorization: Bearer <tu_jwt_token>
```

### Parámetros de URL

| Parámetro | Tipo | Descripción |
|-----------|------|-------------|
| **contract_id** | number | ID del contrato a consultar |

### Ejemplo de solicitud

```bash
curl --location 'https://api.svi.becomedigital.net/api/v1/contract/2' \
--header 'Authorization: Bearer <tu_jwt_token>'
```

### Respuesta de la API

#### ✅ **200 OK - Contrato encontrado**

```json
{
  "id": 2,
  "company_id": 123,
  "company_name": "Mi Empresa S.A.",
  "name": "Contrato Verificación Estándar",
  "description": "Contrato para verificación de identidad con cotejo facial",
  "active": true,
  "created_at": "2023-05-15T10:30:00Z",
  "updated_at": "2024-01-10T15:45:00Z",
  "services": {
    "face_match": true,
    "liveness": true,
    "alteration": true,
    "template": true,
    "estimated_age": false,
    "one_to_many": true,
    "watch_list": false,
    "ani_compliance": false,
    "telco_validation": false,
    "email_validation": false
  },
  "document_types": [
    "national-id",
    "passport",
    "driving-license"
  ],
  "countries": [
    "CO",
    "MX",
    "AR",
    "PE"
  ],
  "quota": {
    "total": 10000,
    "used": 3456,
    "remaining": 6544
  }
}
```

## Descripción de campos de respuesta

### Campos principales

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | number | ID único del contrato |
| `company_id` | number | ID de la empresa propietaria del contrato |
| `company_name` | string | Nombre de la empresa |
| `name` | string | Nombre descriptivo del contrato |
| `description` | string | Descripción detallada del contrato |
| `active` | boolean | Indica si el contrato está activo (`true`) o inactivo (`false`) |
| `created_at` | string | Fecha de creación del contrato |
| `updated_at` | string | Fecha de última actualización |

### Objeto `services`

Define qué servicios de verificación están habilitados en este contrato:

| Servicio | Descripción |
|----------|-------------|
| `face_match` | Cotejo facial entre selfie y documento |
| `liveness` | Prueba de vida (detección de persona real) |
| `alteration` | Detección de alteraciones en documentos |
| `template` | Validación de plantilla del documento |
| `estimated_age` | Validación de edad estimada vs documento |
| `one_to_many` | Comparación 1:N contra lista negra |
| `watch_list` | Verificación contra listas de vigilancia |
| `ani_compliance` | Verificación ANI Colombia |
| `telco_validation` | Validación de números telefónicos |
| `email_validation` | Validación de correos electrónicos |

### Array `document_types`

Lista de tipos de documentos permitidos en este contrato:

- `national-id` - Cédula de ciudadanía, DNI, INE
- `passport` - Pasaporte
- `driving-license` - Licencia de conducir
- `proof-of-residency` - Comprobante de residencia

### Array `countries`

Códigos ISO-2 de países soportados por el contrato:

```json
["CO", "MX", "AR", "PE", "CL", "BR"]
```

### Objeto `quota`

Información sobre el cupo de verificaciones:

| Campo | Descripción |
|-------|-------------|
| `total` | Cantidad total de verificaciones contratadas |
| `used` | Cantidad de verificaciones ya utilizadas |
| `remaining` | Cantidad de verificaciones restantes |

## Casos de uso

### 1. Validar configuración antes de enviar verificación

```python
# Consultar contrato
contract = get_contract(contract_id)

# Validar que el servicio esté habilitado
if not contract["services"]["face_match"]:
    raise Exception("Cotejo facial no disponible en este contrato")

# Validar que el tipo de documento esté permitido
if document_type not in contract["document_types"]:
    raise Exception(f"Tipo de documento {document_type} no permitido")

# Validar que el país esté soportado
if country not in contract["countries"]:
    raise Exception(f"País {country} no soportado")

# Proceder con la verificación
create_verification(...)
```

### 2. Verificar cupo disponible

```python
contract = get_contract(contract_id)

if contract["quota"]["remaining"] < 100:
    send_alert("Cupo bajo: solo quedan {remaining} verificaciones")

if contract["quota"]["remaining"] == 0:
    raise Exception("Cupo agotado. Contacte a soporte.")
```

### 3. Mostrar servicios disponibles en UI

```python
contract = get_contract(contract_id)
enabled_services = [
    service for service, enabled in contract["services"].items()
    if enabled
]

display_services_in_dashboard(enabled_services)
```

### 4. Validar estado del contrato

```python
contract = get_contract(contract_id)

if not contract["active"]:
    raise Exception("El contrato está inactivo. Contacte a soporte.")
```

## Manejo de errores

### **401 - Unauthorized**

```json
{
  "msg": "Missing Authorization Header"
}
```

**Causa:** Token faltante o inválido.

### **404 - Not Found**

```json
{
  "error": "Contract not found"
}
```

**Causa:** El `contract_id` no existe o no pertenece a la empresa autenticada.

**Solución:** Verificar el ID del contrato proporcionado por Become Digital.

### **403 - Forbidden**

```json
{
  "error": "No tienes permisos para acceder a este contrato"
}
```

**Causa:** El contrato pertenece a otra empresa.

## Ejemplo de validación completa

```python
def validate_verification_request(contract_id, country, document_type, services_needed):
    """
    Valida que una solicitud de verificación sea compatible con el contrato
    """
    # Obtener configuración del contrato
    contract = get_contract(contract_id)
    
    # Validar que el contrato esté activo
    if not contract["active"]:
        return False, "Contrato inactivo"
    
    # Validar cupo
    if contract["quota"]["remaining"] == 0:
        return False, "Cupo agotado"
    
    # Validar país
    if country not in contract["countries"]:
        return False, f"País {country} no soportado"
    
    # Validar tipo de documento
    if document_type not in contract["document_types"]:
        return False, f"Tipo de documento {document_type} no permitido"
    
    # Validar servicios requeridos
    for service in services_needed:
        if not contract["services"].get(service, False):
            return False, f"Servicio {service} no habilitado en el contrato"
    
    return True, "Validación exitosa"

# Uso
is_valid, message = validate_verification_request(
    contract_id=2,
    country="CO",
    document_type="national-id",
    services_needed=["face_match", "liveness"]
)

if is_valid:
    # Proceder con la verificación
    create_verification(...)
else:
    print(f"Error: {message}")
```

## Información importante

### Modificación de contratos

- ⚠️ Los contratos solo pueden ser modificados por el equipo de Become Digital
- 📧 Para solicitar cambios, contacta al soporte técnico
- 🔄 Los cambios en el contrato pueden tomar hasta 24 horas en aplicarse

### Contratos múltiples

- Una empresa puede tener **múltiples contratos** activos
- Cada contrato puede tener diferentes configuraciones y servicios
- Usa el `contract_id` apropiado según el tipo de verificación que necesites

### Cuota y facturación

- El campo `quota` se actualiza en tiempo real
- Cuando el cupo se agota, las verificaciones fallarán
- Contacta a soporte para renovar o ampliar tu cupo

## Buenas prácticas

1. **Cachea la configuración del contrato** (se actualiza raramente)
2. **Valida el contrato antes de crear verificaciones** (evita errores)
3. **Monitorea el cupo disponible** (configura alertas)
4. **Maneja el caso de contrato inactivo** (lógica de fallback)

## Siguientes pasos

- [Crear verificación con el contrato →](verification-add.md)
- [Consultar resultados →](verification-results.md)
- [Re-verificación →](re-verification.md)

