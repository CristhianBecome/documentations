# Verificación ANI Compliance

Endpoint principal para verificar la identidad de una persona contra los registros oficiales de la Registraduría Nacional del Estado Civil de Colombia.

## POST `/aniCompliance`

### Request

**Headers:**
```http
Content-Type: application/json
Authorization: Bearer <tu_jwt_token>
```

**Body:**
```json
{
  "document_number": 12345678,
  "contract_id": 100,
  "departamentoExpedicion": "VALLE",
  "fechaExpedicion": "15/03/1990",
  "municipioExpedicion": "CALI",
  "primerApellido": "GARCIA",
  "segundoNombre": "MARIA",
  "primerNombre": "JUAN",
  "segundoApellido": "LOPEZ"
}
```

### Parámetros

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| `document_number` | number | ✅ | Número de cédula de ciudadanía |
| `contract_id` | number | ✅ | ID del contrato asignado |
| `departamentoExpedicion` | string | ✅ | Departamento de expedición |
| `fechaExpedicion` | string | ✅ | Fecha de expedición (DD/MM/YYYY) |
| `municipioExpedicion` | string | ✅ | Municipio de expedición |
| `primerApellido` | string | ✅ | Primer apellido |
| `segundoNombre` | string | ❌ | Segundo nombre (opcional) |
| `primerNombre` | string | ✅ | Primer nombre |
| `segundoApellido` | string | ❌ | Segundo apellido (opcional) |

### Ejemplo de Request

```bash
curl --location 'https://api.dev.svi.becomedigital.net/api/v1/aniCompliance' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer tu_jwt_token_aqui' \
--data '{
    "document_number": 12345678,
    "contract_id": 100,
    "departamentoExpedicion": "VALLE",
    "fechaExpedicion": "15/03/1990",
    "municipioExpedicion": "CALI",
    "primerApellido": "GARCIA",
    "segundoNombre": "MARIA",
    "primerNombre": "JUAN",
    "segundoApellido": "LOPEZ"
}'
```

### Response

*Agregar ejemplo de respuesta*