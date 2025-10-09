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

## Códigos de Estado de Cédula

### Estados válidos y vigentes

| Código | Estado | Descripción |
|--------|--------|-------------|
| **0** | Vigente | Cédula activa y válida |
| **1** | Vigente | Cédula activa y válida |
| **12** | Vigente | Vigente con Pérdida o Suspensión de los Derechos Políticos |
| **14** | Vigente | Vigente con Interdicción Judicial |

### Estados cancelados

| Código | Estado | Descripción |
|--------|--------|-------------|
| **21** | Cancelada | Cancelada por Muerte |
| **22** | Cancelada | Cancelada por Doble Cedulación |
| **23** | Cancelada | Cancelada por Suplantación |
| **24** | Cancelada | Cancelada por Menoría de Edad |
| **25** | Cancelada | Cancelada por Extranjería |
| **26** | Cancelada | Cancelada por Mala Elaboración |
| **27** | Cancelada | Cancelada con reasignación de cupo numérico |
| **28** | Cancelada | Cancelada por Extranjería sin Carta de Naturaleza |
| **51** | Cancelada | Cancelada por Muerte Facultad Ley 1365 de 2009 |
| **52** | Cancelada | Cancelada por Intento de Doble Cedulación NO Expedida |
| **53** | Cancelada | Cancelada por Falsa Identidad |
| **54** | Cancelada | Cancelada por Menoría de Edad NO Expedida |
| **55** | Cancelada | Cancelada por Extranjería NO Expedida |
| **56** | Cancelada | Cancelada por Mala Elaboración NO Expedida |

### Estados pendientes

| Código | Estado | Descripción |
|--------|--------|-------------|
| **88** | Pendiente | Pendiente Solicitud en Reproceso |
| **99** | Pendiente | Pendiente por estar en Proceso de Expedición |

## Interpretación de Resultados

### ✅ Verificación Exitosa

Todos los campos de comparación en `true` y cédula vigente:

```json
{
  "document_number_match": true,
  "departamentoExpedicion_match": true,
  "fechaExpedicion_match": true,
  "municipioExpedicion_match": true,
  "primerApellido_match": true,
  "segundoNombre_match": true,
  "primerNombre_match": true,
  "segundoApellido_match": true,
  "nuip_status": "VIGENTE",
  "nuip_status_code": "0"
}
```

**Interpretación:** 
- ✅ Todos los datos coinciden con los registros oficiales
- ✅ La cédula está vigente
- ✅ Puede utilizarse para procesos de identificación

### ⚠️ Datos Inconsistentes

Uno o más campos de comparación en `false`:

```json
{
  "document_number_match": true,
  "departamentoExpedicion_match": true,
  "fechaExpedicion_match": false,
  "municipioExpedicion_match": true,
  "primerApellido_match": true,
  "segundoNombre_match": false,
  "primerNombre_match": true,
  "segundoApellido_match": true,
  "nuip_status": "VIGENTE",
  "nuip_status_code": "0"
}
```

**Interpretación:**
- ⚠️ La fecha de expedición no coincide
- ⚠️ El segundo nombre no coincide
- ℹ️ La cédula está vigente, pero hay discrepancias en los datos

**Acción recomendada:**
- Solicitar corrección de datos
- Verificar si hay errores de digitación
- Solicitar documentación adicional

### ❌ Documento No Vigente

```json
{
  "document_number_match": true,
  "departamentoExpedicion_match": true,
  "fechaExpedicion_match": true,
  "municipioExpedicion_match": true,
  "primerApellido_match": true,
  "segundoNombre_match": true,
  "primerNombre_match": true,
  "segundoApellido_match": true,
  "nuip_status": "CANCELADA POR MUERTE",
  "nuip_status_code": "21"
}
```

**Interpretación:**
- ❌ Aunque los datos coinciden, la cédula fue cancelada
- ❌ No puede utilizarse para procesos oficiales

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

## Casos de Uso

### 1. Validación de Identidad

Confirmar que los datos proporcionados por el usuario coinciden exactamente con los registros oficiales:

```python
def validar_identidad_colombia(datos_usuario):
    response = requests.post(
        "https://api.svi.becomedigital.net/api/v1/aniCompliance",
        headers={"Authorization": f"Bearer {token}"},
        json=datos_usuario
    )
    
    result = response.json()
    
    # Verificar que todos los campos coincidan
    matches = [
        result["document_number_match"],
        result["departamentoExpedicion_match"],
        result["fechaExpedicion_match"],
        result["municipioExpedicion_match"],
        result["primerNombre_match"],
        result["segundoNombre_match"],
        result["primerApellido_match"],
        result["segundoApellido_match"]
    ]
    
    # Verificar estado vigente
    vigente = result["nuip_status_code"] in ["0", "1"]
    
    return all(matches) and vigente
```

### 2. Detección de Inconsistencias

Identificar campos específicos que no coinciden:

```python
def detectar_inconsistencias(result):
    campos_incorrectos = []
    
    if not result["primerNombre_match"]:
        campos_incorrectos.append("Primer nombre")
    if not result["segundoNombre_match"]:
        campos_incorrectos.append("Segundo nombre")
    if not result["primerApellido_match"]:
        campos_incorrectos.append("Primer apellido")
    if not result["segundoApellido_match"]:
        campos_incorrectos.append("Segundo apellido")
    if not result["fechaExpedicion_match"]:
        campos_incorrectos.append("Fecha de expedición")
    
    return campos_incorrectos
```

### 3. Verificación de Estado

Validar que la cédula esté vigente:

```python
def verificar_estado_cedula(result):
    codigo = result["nuip_status_code"]
    estado = result["nuip_status"]
    
    if codigo in ["0", "1"]:
        return "vigente", "Cédula válida y vigente"
    elif codigo in ["12", "14"]:
        return "vigente_con_restricciones", f"Cédula vigente pero: {estado}"
    elif codigo in ["88", "99"]:
        return "pendiente", "Cédula en proceso"
    else:
        return "cancelada", f"Cédula cancelada: {estado}"
```

## Formato de datos

### Fecha de expedición

**Formato requerido:** `DD/MM/YYYY`

**Ejemplos válidos:**
- `14/06/1971`
- `01/01/2000`
- `25/12/1985`

**Ejemplos inválidos:**
- `1971-06-14` ❌
- `06/14/1971` ❌
- `14-06-1971` ❌

### Nombres y apellidos

- Deben estar en **MAYÚSCULAS**
- Sin caracteres especiales (tildes permitidas)
- Sin espacios adicionales al inicio o final

### Departamento y municipio

- En **MAYÚSCULAS**
- Nombres oficiales según la Registraduría
- Ejemplos: `VALLE`, `ANTIOQUIA`, `CUNDINAMARCA`

## Siguientes pasos

- [Crear verificación de identidad completa →](verification-add.md)
- [Validación Telco →](telco.md)
- [Validación Email →](email.md)

