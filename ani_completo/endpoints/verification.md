# Consulta ANI — `aniValidation`

Endpoint principal para consultar la identidad de una persona en los registros oficiales de la Registraduría Nacional del Estado Civil de Colombia (ANI).

Referencia complementaria: [ANI REGISTRADURIA DE COLOMBIA (Postman)](https://documenter.getpostman.com/view/22917023/2sAY4xA1rZ).

## POST `/aniValidation`

### Headers requeridos

```http
Content-Type: application/json
Authorization: Bearer <tu_jwt_token>
```

### Parámetros del request

La consulta real solo requiere el número de documento y el contrato. El resto de datos (nombres, expedición, estado, etc.) los devuelve la API en la respuesta.

```json
{
  "document_number": 123456789,
  "contract_id": 1
}
```

### Descripción de parámetros

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| `document_number` | number | Sí | Número de cédula de ciudadanía a consultar |
| `contract_id` | number | Sí | ID del contrato asignado a tu integración |

### Ejemplo de request

```bash
curl --location 'https://api.svi.becomedigital.net/api/v1/aniValidation' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer tu_jwt_token_aqui' \
--data '{
    "document_number": 123456789,
    "contract_id": 1
}'
```

### Respuestas de la API

#### **200 — Consulta exitosa**

La respuesta es un objeto con los datos devueltos por la consulta ANI. Los campos que no aplican o no están disponibles pueden venir como `null`.

**Campos habituales**

| Campo | Descripción |
|-------|-------------|
| `codError` | Código de error de la fuente (por ejemplo `"0"` cuando la consulta fue correcta) |
| `contract_id` | Identificador del contrato utilizado |
| `nuip` | Número único de identificación personal |
| `descripcionEstado` | Texto del estado de la cédula (por ejemplo `VIGENTE`, `CANCELADA POR MUERTE`) |
| `estadoCedula` | Código de estado de la cédula (string; ver tabla más abajo) |
| `fechaHoraConsulta` | Fecha y hora de la consulta |
| `departamentoExpedicion`, `municipioExpedicion`, `fechaExpedicion` | Datos de expedición |
| `primerNombre`, `segundoNombre`, `primerApellido`, `segundoApellido`, `particula` | Datos de nombre según registro |
| `fechaNacimiento`, `lugarNacimiento`, `genero`, `estatura`, `grupoSanguineo` | Pueden ser `null` si la fuente no los retorna |
| `fechaDefuncion`, `fechaAfectacion`, `fechaReferencia`, `informante`, `lugarNovedad` | Relevantes en cancelaciones (por ejemplo por muerte) |
| `serial`, `identity_id`, `anoResolucion`, `numResolucion`, `oficinaExpedicion`, `lugarPreparacion`, `fechaPreparacion` | Adicionales; a menudo `null` |

**Ejemplo — cédula vigente** (request de ejemplo con `document_number` `123456789` y `contract_id` `1`; `nuip` coincide con el documento consultado)

```json
{
    "anoResolucion": null,
    "codError": "0",
    "contract_id": 1,
    "departamentoExpedicion": "VALLE",
    "descripcionEstado": "VIGENTE",
    "estadoCedula": "0",
    "estatura": null,
    "fechaAfectacion": null,
    "fechaDefuncion": null,
    "fechaExpedicion": "14/10/2015",
    "fechaHoraConsulta": "2026-04-27 09:27:37",
    "fechaNacimiento": null,
    "fechaPreparacion": null,
    "fechaReferencia": null,
    "genero": null,
    "grupoSanguineo": null,
    "identity_id": null,
    "informante": null,
    "lugarNacimiento": null,
    "lugarNovedad": null,
    "lugarPreparacion": null,
    "municipioExpedicion": "TULUA",
    "nuip": "123456789",
    "numResolucion": null,
    "oficinaExpedicion": null,
    "particula": "DE",
    "primerApellido": "PRADO",
    "primerNombre": "JUAN",
    "segundoApellido": "GARCIA",
    "segundoNombre": ALBERTO",
    "serial": null
}
```

**Ejemplo — cédula cancelada por muerte** (otro titular; mismo `contract_id` que en la petición)

```json
{
    "anoResolucion": null,
    "codError": "0",
    "contract_id": 173,
    "departamentoExpedicion": "VALLE",
    "descripcionEstado": "CANCELADA POR MUERTE",
    "estadoCedula": "21",
    "estatura": null,
    "fechaAfectacion": "10/02/2021",
    "fechaDefuncion": "06/02/2021",
    "fechaExpedicion": "03/08/1966",
    "fechaHoraConsulta": "2026-04-27 16:13:27",
    "fechaNacimiento": null,
    "fechaPreparacion": null,
    "fechaReferencia": "08/02/2021",
    "genero": null,
    "grupoSanguineo": null,
    "identity_id": null,
    "informante": "NOTARIA 1 TULUA",
    "lugarNacimiento": null,
    "lugarNovedad": null,
    "lugarPreparacion": null,
    "municipioExpedicion": "BUGA",
    "nuip": "9876456123",
    "numResolucion": null,
    "oficinaExpedicion": null,
    "particula": " ",
    "primerApellido": "RENDON",
    "primerNombre": "GONZALO",
    "segundoApellido": "RESTREPO",
    "segundoNombre": null,
    "serial": "0005805032135"
}
```

### Manejo de errores

#### **412 — Precondition Failed**

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

**Problema con el contrato:**

```json
{ "msg": "El contrato no contiene este servicio" }
```

#### **500 — Internal Server Error**

```json
{ "msg": "No se pudo obtener información del documento" }
```

#### **401 — Unauthorized**

```json
{ "msg": "Token JWT inválido o expirado" }
```

### Proceso de validación

1. **Validación de token JWT** — Error 401 si el token es inválido o ha expirado.
2. **Validación de `document_number`** — Error 412 si el campo está ausente.
3. **Validación de `contract_id`** — Error 412 si está ausente o no es un número entero.
4. **Validación del cuerpo** — Error 412 si faltan `document_number` o `contract_id`, o `contract_id` no es entero.
5. **Validación del contrato** — Error 412 si el contrato no incluye este servicio.
6. **Consulta ANI** — Error 500 si no se puede obtener información del documento.
7. **Respuesta exitosa** — Código 200 con el objeto de datos devuelto por la fuente (incluye `estadoCedula`, `descripcionEstado`, datos de identificación, etc.).

### Interpretación de la respuesta

- **`codError`:** Indica el resultado operativo reportado por la fuente; `"0"` suele indicar consulta sin error de negocio en la respuesta mostrada.
- **`estadoCedula` y `descripcionEstado`:** Definen el estado oficial de la cédula; usalos como criterio principal frente a reglas de negocio (vigente, cancelada, pendiente, etc.).
- **`fechaHoraConsulta`:** Momento de la consulta en el sistema.
- **`nuip`:** Identificador en el registro; debe alinearse con el documento consultado cuando aplica.
- En **cancelación por muerte**, suelen informarse `fechaDefuncion`, `fechaAfectacion`, `fechaReferencia`, `informante` y `serial`, además del cambio en `descripcionEstado`.

### Estados de cédula de ciudadanía (`estadoCedula`)

El campo `estadoCedula` llega como **string** con el código numérico del estado en los registros de la Registraduría. `descripcionEstado` amplía el significado en texto.

| **Código (`estadoCedula`)** | **Descripción del estado** |
|-----------------------------|----------------------------|
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

**Estados válidos y vigentes:** Códigos 0, 1, 12, 14.

- La cédula figura como vigente en el registro (sujeto a tu política de aceptación).

**Estados cancelados:** Códigos 21-28, 51-56.

- La cédula no es válida para identificación según el motivo indicado en `descripcionEstado`.

**Estados pendientes:** Códigos 88, 99.

- Trámite o reproceso en curso según la Registraduría.
