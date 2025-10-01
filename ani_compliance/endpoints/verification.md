# Verificación ANI Compliance

Endpoint principal para verificar la identidad de una persona consultando los registros oficiales de la Registraduría Nacional del Estado Civil de Colombia.

## POST `/aniCompliance`

### Headers requeridos

```http
Content-Type: application/json
Authorization: Bearer <tu_jwt_token>
```

### Parámetros del Request

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

### Descripción de parámetros

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| `document_number` | number | ✅ | Número de cédula de ciudadanía |
| `contract_id` | number | ✅ | ID del contrato asignado |
| `departamentoExpedicion` | string | ✅ | Departamento donde se expidió la cédula |
| `fechaExpedicion` | string | ✅ | Fecha de expedición (formato DD/MM/YYYY) |
| `municipioExpedicion` | string | ✅ | Municipio donde se expidió la cédula |
| `primerApellido` | string | ✅ | Primer apellido registrado |
| `segundoNombre` | string | ✅ | Segundo nombre registrado |
| `primerNombre` | string | ✅ | Primer nombre registrado |
| `segundoApellido` | string | ✅ | Segundo apellido registrado |

### Ejemplo de request

```bash
curl --location 'https://api.svi.becomedigital.net/api/v1/aniCompliance' \
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

### Respuestas de la API

#### ✅ **200 - Verificación exitosa**
```json
{
  "document_number_match": true,
  "departamentoExpedicion_match": true,
  "fechaExpedicion_match": false,
  "municipioExpedicion_match": true,
  "primerApellido_match": true,
  "segundoNombre_match": true,
  "primerNombre_match": true,
  "segundoApellido_match": true,
  "nuip_status": "VIGENTE",
  "nuip_status_code": "1"
}
```

### Manejo de errores

#### **412 - Precondition Failed**

**Campos faltantes o inválidos:**

```json
{ "msg": "debes indicar document_number asociado" }
```

```json
{ "msg": "debes indicar contract_id asociado" }
```

```json
{ "msg": "el contract_id debe ser un valor de tipo entero" }
```

```json
{ "msg": "debes indicar departamentoExpedicion asociado" }
```

```json
{ "msg": "debes indicar fechaExpedicion asociado" }
```

```json
{ "msg": "debes indicar municipioExpedicion asociado" }
```

```json
{ "msg": "debes indicar primerApellido asociado" }
```

```json
{ "msg": "debes indicar primerNombre asociado" }
```

```json
{ "msg": "debes indicar segundoApellido asociado" }
```

```json
{ "msg": "debes indicar segundoNombre asociado" }
```

**Problema con el contrato:**

```json
{ "msg": "El contrato no contiene este servicio" }
```

#### **500 - Internal Server Error**

```json
{ "msg": "No se pudo obtener información del documento" }
```

#### **401 - Unauthorized**

```json
{ "msg": "Token JWT inválido o expirado" }
```

### Proceso de validación

La API sigue este flujo de validación:

1. **Validación de token JWT** → Error 401 si el token es inválido o ha expirado
2. **Validación de document_number** → Error 412 si el campo está ausente
3. **Validación de contract_id** → Error 412 si está ausente o no es un número entero
4. **Validación de campos obligatorios** → Error 412 si falta cualquiera de los 8 campos requeridos
5. **Validación del contrato** → Error 412 si el contrato no incluye este servicio
6. **Consulta de verificación** → Error 500 si no se puede obtener información del documento
7. **Respuesta exitosa** → Código 200 con las comparaciones y el estado de la cédula

### Interpretación de la respuesta

La respuesta exitosa incluye campos de comparación que indican si cada dato proporcionado coincide con los registros oficiales:

- `document_number_match` - Indica si el número de documento coincide
- `departamentoExpedicion_match` - Indica si el departamento de expedición coincide  
- `fechaExpedicion_match` - Indica si la fecha de expedición coincide
- `municipioExpedicion_match` - Indica si el municipio de expedición coincide
- `primerApellido_match` - Indica si el primer apellido coincide
- `segundoNombre_match` - Indica si el segundo nombre coincide
- `primerNombre_match` - Indica si el primer nombre coincide
- `segundoApellido_match` - Indica si el segundo apellido coincide
- `nuip_status` - Estado descriptivo de la cédula en formato texto legible (ej: "VIGENTE", "CANCELADA", "PENDIENTE")
- `nuip_status_code` - Código numérico del estado de la cédula

### Estados de cédula de ciudadanía

El campo `nuip_status_code` retorna un código numérico que indica el estado actual de la cédula de ciudadanía en los registros oficiales de la Registraduría Nacional. A continuación se detallan todos los códigos posibles y su significado:

| **Código** | **Descripción del estado** |
|------------|----------------------------|
| 0 | Vigente |
| 1 | Vigente |
| 12 | Vigente con Pérdida o Suspensión de los Derechos Políticos |
| 14 | Vigente con Interdicción Judicial |
| 21 | Cancelada por Muerte |
| 22 | Cancelada por Doble Cedulación |
| 23 | Cancelada por Suplantación |
| 24 | Cancelada por Menoría de Edad |
| 25 | Cancelada por Extranjería |
| 26 | Cancelada por Mala Elaboración |
| 27 | Cancelada con reasignación de cupo numérico |
| 28 | Cancelada por Extranjería sin Carta de Naturaleza |
| 51 | Cancelada por Muerte Facultad Ley 1365 de 2009 |
| 52 | Cancelada por Intento de Doble Cedulación NO Expedida |
| 53 | Cancelada por Falsa Identidad |
| 54 | Cancelada por Menoría de Edad NO Expedida |
| 55 | Cancelada por Extranjería NO Expedida |
| 56 | Cancelada por Mala Elaboración NO Expedida |
| 88 | Pendiente Solicitud en Reproceso |
| 99 | Pendiente por estar en Proceso de Expedición |

### Clasificación de estados

**Estados válidos y vigentes:** Códigos 0, 1, 12, 14
- La cédula está vigente y puede ser utilizada para identificación

**Estados cancelados:** Códigos 21-28, 51-56  
- La cédula ha sido cancelada por diferentes motivos y no es válida

**Estados pendientes:** Códigos 88, 99
- La cédula está en proceso de expedición o reproceso