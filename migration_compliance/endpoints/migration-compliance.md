# Consulta de Migración Colombia

Endpoint para consulta de migración Colombia con comparación 1 a 1 de campos. Valida los datos del documento de identidad (CE, PPT o PEP) contra los registros oficiales y retorna un resultado booleano por cada campo comparado.

## POST `/govchecks/co/migration/compliance`

### Headers requeridos

```http
Content-Type: application/json
Authorization: Bearer <tu_jwt_token>
```

### Body esperado

```json
{
    "contract_id": 173,
    "id_number": "6982611",
    "tipo_documento": "CE" | "PPT" | "PEP",
    "expedition_date": "YYYY-MM-DD",
    "fecha_expedicion_doc": "YYYY-MM-DD",
    "fecha_nacimiento": "YYYY-MM-DD",
    "fecha_vencimiento": "YYYY-MM-DD",
    "nacionalidad": "VEN",
    "nombres": "string",
    "primer_apellido": "string",
    "segundo_apellido": "string"
}
```

- **Campo opcional:** `expedition_date`. El resto son requeridos para la comparación.
- **Valores no disponibles:** Usar `"N/A"` en los campos de comparación cuando no se tenga el valor.

### Descripción de parámetros

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| `contract_id` | int | ✅ | ID del contrato asignado |
| `id_number` | string | ✅ | Número del documento de identidad |
| `tipo_documento` | string | ✅ | Tipo de documento: `"CE"`, `"PPT"` o `"PEP"` |
| `expedition_date` | string | ❌ | Fecha de expedición en formato `YYYY-MM-DD` |
| `fecha_expedicion_doc` | string | ✅ | Fecha de expedición del documento (`YYYY-MM-DD` o `"N/A"`) |
| `fecha_nacimiento` | string | ✅ | Fecha de nacimiento (`YYYY-MM-DD` o `"N/A"`) |
| `fecha_vencimiento` | string | ✅ | Fecha de vencimiento del documento (`YYYY-MM-DD` o `"N/A"`) |
| `nacionalidad` | string | ✅ | Código de nacionalidad (ej: `"VEN"`) o `"N/A"` |
| `nombres` | string | ✅ | Nombres del titular o `"N/A"` |
| `primer_apellido` | string | ✅ | Primer apellido o `"N/A"` |
| `segundo_apellido` | string | ✅ | Segundo apellido o `"N/A"` |

### Ejemplo de request (cURL con datos dummy)

```bash
curl --location 'https://api.svi.becomedigital.net/api/v1/govchecks/co/migration/compliance' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmcmVzaCI6ZmFsc2UsImlhdCI6MTc3MjcyOTE2NCwiYXVkIjoiLi4uIiwianRpIjoiLi4uIiwiZXhwIjoxODAzODMzMTY0fQ.dummy_token_replace_with_real_jwt' \
--data '{
    "id_number": "6982611",
    "tipo_documento": "PPT",
    "contract_id": 173,
    "fecha_expedicion_doc": "2022-06-08",
    "fecha_nacimiento": "2005-06-24",
    "fecha_vencimiento": "2031-05-30",
    "nacionalidad": "VEN",
    "nombres": "ARIANY DESIRE",
    "primer_apellido": "GONZALEZ",
    "segundo_apellido": "CORREA"
}'
```

### Ejemplo de body con datos dummy (solo referencia)

```json
{
    "contract_id": 173,
    "id_number": "6982611",
    "tipo_documento": "PPT",
    "expedition_date": "2022-06-08",
    "fecha_expedicion_doc": "2022-06-08",
    "fecha_nacimiento": "2005-06-24",
    "fecha_vencimiento": "2031-05-30",
    "nacionalidad": "VEN",
    "nombres": "ARIANY DESIRE",
    "primer_apellido": "GONZALEZ",
    "segundo_apellido": "CORREA"
}
```

### Respuestas de la API

#### ✅ **200 - Consulta exitosa**

La respuesta devuelve un objeto con un booleano por cada campo de comparación: `true` si coincide con el registro oficial, `false` si no coincide.

```json
{
    "fecha_expedicion_doc": true,
    "fecha_nacimiento": true,
    "fecha_vencimiento": true,
    "nacionalidad": true,
    "nombres": true,
    "primer_apellido": true,
    "segundo_apellido": true
}
```

#### Ejemplo con algún campo que no coincide (dummy)

```json
{
    "fecha_expedicion_doc": true,
    "fecha_nacimiento": true,
    "fecha_vencimiento": true,
    "nacionalidad": true,
    "nombres": true,
    "primer_apellido": false,
    "segundo_apellido": true
}
```

### Interpretación de la respuesta

| Campo | Significado |
|-------|-------------|
| `fecha_expedicion_doc` | Coincide la fecha de expedición del documento |
| `fecha_nacimiento` | Coincide la fecha de nacimiento |
| `fecha_vencimiento` | Coincide la fecha de vencimiento |
| `nacionalidad` | Coincide el código de nacionalidad |
| `nombres` | Coinciden los nombres |
| `primer_apellido` | Coincide el primer apellido |
| `segundo_apellido` | Coincide el segundo apellido |

### Tipos de documento soportados

| Valor | Descripción |
|-------|-------------|
| `CE` | Cédula de Extranjería |
| `PPT` | Permiso por Protección Temporal |
| `PEP` | Permiso Especial de Permanencia |

### Notas

- Todas las fechas en el body deben enviarse en formato **YYYY-MM-DD**.
- Para campos sin valor, enviar la cadena **"N/A"**.
- El token del header `Authorization` debe ser un JWT válido obtenido desde el endpoint `/auth`.
