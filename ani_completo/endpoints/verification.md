# Consulta ANI — `aniValidation`

Endpoint para consultar la identidad de una persona en los registros oficiales de la Registraduría Nacional del Estado Civil de Colombia.

La ruta de sandbox es otra: **POST** [`/aniValidationSB`](sandbox.md).


## POST `/aniValidation`

Hace falta un Bearer con scope de API general (`full_access`). Si el token trae `grant_type`, debe ser `standalone:ani` o `standalone:govchecks`. Un token sin `grant_type` también pasa.

Solicite el token en **POST** [`/auth`](auth.md) con `"grant_type": "standalone:ani"`.

### Headers requeridos

```http
Content-Type: application/json
Authorization: Bearer <tu_jwt_token>
```

### Parámetros del request

La consulta requiere el número de documento y el contrato. `identity_id` es opcional: si no se envía, la consulta es standalone y la respuesta trae `identity_id` en `null`.

`document_number` es string. Si llega como número, se convierte a string. `contract_id: true` se acepta y queda `1`. `identity_id: 1.5` se acepta y queda `1`.

`simulate_ani_maintenance` en `true`, `"true"` o `"1"` no consulta la fuente y devuelve el cuerpo de mantenimiento con `"simulated": true`.

```json
{
  "document_number": "123456789",
  "contract_id": 1
}
```

### Descripción de parámetros

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| `document_number` | string | Sí | Número de cédula. Un número JSON se convierte a string. `null` responde 422. `true` responde 500. |
| `contract_id` | integer | Sí | ID del contrato. `true` se acepta como `1`. Texto, `null` o un decimal responden 422. |
| `identity_id` | integer | No | Si se omite, la respuesta trae `identity_id` en `null`. Texto no numérico responde 422. `1.5` queda `1`. `true` responde 500. |
| `simulate_ani_maintenance` | boolean o string | No | `true`, `"true"` o `"1"` devuelven mantenimiento sin consultar. |

### Ejemplo de request

```bash
curl --location 'https://api.svi.becomedigital.net/api/v1/aniValidation' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer tu_jwt_token_aqui' \
--data '{
    "document_number": "123456789",
    "contract_id": 1
}'
```

### Respuestas de la API

Los strings de un registro encontrado van escapados en HTML. En standalone, `identity_id` es `null`.

#### **200 — Registro encontrado**

`codError` es `"0"`. La respuesta incluye `contract_id`, `identity_id`, `fechaHoraConsulta`, `nuip`, `estadoCedula`, `descripcionEstado`, los nombres, `particula`, `departamentoExpedicion`, `municipioExpedicion` y `fechaExpedicion`.

El resto de campos de cédula (`anoResolucion`, `estatura`, `fechaNacimiento`, `serial`, y los demás) llega informado cuando la consulta los trae. Si no, van en `null` y `particula` es un espacio (`" "`). En ese caso también puede venir `genero`.

**Campos habituales**

| Campo | Descripción |
|-------|-------------|
| `codError` | Código de la consulta. `"0"` cuando el registro se obtuvo |
| `contract_id` | Identificador del contrato utilizado |
| `nuip` | Número único de identificación personal |
| `descripcionEstado` | Texto del estado de la cédula (por ejemplo `VIGENTE`, `CANCELADA POR MUERTE`) |
| `estadoCedula` | Código de estado de la cédula (string; ver tabla más abajo) |
| `fechaHoraConsulta` | Fecha y hora de la consulta |
| `departamentoExpedicion`, `municipioExpedicion`, `fechaExpedicion` | Datos de expedición |
| `primerNombre`, `segundoNombre`, `primerApellido`, `segundoApellido`, `particula` | Datos de nombre según registro |
| `genero` | Puede venir informado, o `null` |
| `fechaNacimiento`, `lugarNacimiento`, `estatura`, `grupoSanguineo` | Pueden ser `null` |
| `fechaDefuncion`, `fechaAfectacion`, `fechaReferencia`, `informante`, `lugarNovedad` | Relevantes en cancelaciones (por ejemplo por muerte) |
| `serial`, `identity_id`, `anoResolucion`, `numResolucion`, `oficinaExpedicion`, `lugarPreparacion`, `fechaPreparacion` | Adicionales. En standalone, `identity_id` es `null` |

**Ejemplo — cédula vigente**

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
    "segundoNombre": "ALBERTO",
    "serial": null
}
```

**Ejemplo — cédula cancelada por muerte**

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

**Ejemplo — el resto de campos de cédula en null**

`particula` es un espacio. `genero` puede venir informado.

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
    "genero": "M",
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
    "particula": " ",
    "primerApellido": "PRADO",
    "primerNombre": "JUAN",
    "segundoApellido": "GARCIA",
    "segundoNombre": "ALBERTO",
    "serial": null
}
```

#### **200 — Error de contrato**

Estos resultados no salen en 4xx. El cuerpo es solo `msg`.

**El contrato no existe para la compañía del token:**

```json
{ "msg": "No se encontro el contrato" }
```

**El token no puede usar ese `contract_id`:**

```json
{ "msg": "Acceso no autorizado para el contract_id solicitado" }
```

**El contrato no tiene habilitada esta consulta** (`can_registraduria_co_standalone` es `false` o `null`):

```json
{ "msg": "Contrato no disponible para este servicio" }
```

**Se alcanzó el máximo de validaciones** (`count_registraduria_co_standalone` llegó al tope):

```json
{ "msg": "Límite de validaciones alcanzado" }
```

#### **200 — Mantenimiento**

Si el body trae `simulate_ani_maintenance` en `true`, `"true"` o `"1"`, no se llama a la consulta:

```json
{
  "estadoConsulta": {
    "numeroControl": "0",
    "codError": "016",
    "descripcionError": "El sistema se encuentra en mantenimiento. Esta consulta no será registrada en el sistema.",
    "fechaHoraConsulta": "YYYY-MM-DD HH:MM:SS"
  },
  "datosCedulas": null,
  "simulated": true
}
```

El mismo cuerpo, sin `simulated`, sale cuando la consulta no trae `datosCedulas`. Un timeout se arma por dentro como 504 y la ruta lo publica como este 200.

#### **401 — Unauthorized**

**Token inválido o vencido.** La respuesta incluye `WWW-Authenticate: Bearer`.

```json
{ "msg": "Token invalid or expired" }
```

**El token no trae el claim de identidad:**

```json
{ "msg": "Token missing identity claim" }
```

#### **403 — Forbidden**

**No hay Bearer:**

```json
{ "msg": "Not authenticated" }
```

**Scope distinto de `full_access`, o token de cambio de contraseña:**

```json
{ "msg": "This token is not authorized for this operation" }
```

**El `grant_type` no es `standalone:govchecks` ni `standalone:ani`.** Un token sin `grant_type` sí pasa.

```json
{ "msg": "Token grant type not authorized for this endpoint" }
```

**Token `dashboard:*` con origen o IP fuera de la allowlist:**

```json
{ "msg": "Origin not allowed for dashboard token" }
```

#### **412 — Precondition Failed**

El contrato no incluye este servicio:

```json
{ "msg": "El contrato no contiene este servicio" }
```

Si faltan `document_number` o `contract_id`, o `contract_id` no es entero, la respuesta es **422**, no 412.

#### **422 — Unprocessable Entity**

`msg` cita solo el primer error. Si hay más, van todos en `detail`. `input` repite el JSON enviado. Pasa antes de llegar al servicio si faltan `document_number` o `contract_id`, si `contract_id` no es entero (`null`, texto o decimal) o si `identity_id` no es entero. `document_number` en `null` también es 422.

Body vacío. `msg` nombra solo `document_number`. `detail` trae los dos:

```json
{
  "msg": "body → document_number: Field required",
  "detail": [
    {
      "type": "missing",
      "loc": ["body", "document_number"],
      "msg": "Field required",
      "input": {},
      "url": "https://errors.pydantic.dev/2.14/v/missing"
    },
    {
      "type": "missing",
      "loc": ["body", "contract_id"],
      "msg": "Field required",
      "input": {},
      "url": "https://errors.pydantic.dev/2.14/v/missing"
    }
  ]
}
```

`document_number: true` e `identity_id: true` no son 422: responden **500** `{"msg": "Ocurrió un error inesperado"}`. `contract_id: true` pasa y queda `1`. `identity_id: 1.5` pasa y queda `1`.

La consulta también responde 422, con otro cuerpo, si la entrada o la respuesta no valida. El detalle interno no sale:

```json
{ "success": false, "msg": "<mensaje fijo>" }
```

#### **429 — Too Many Requests**

Al superar 100.000 respuestas `200` en el día. El cuerpo es `{}`.

#### **404 — Sin registro utilizable**

```json
{ "success": false, "msg": "No se pudo obtener información del documento" }
```

#### **502 — Bad Gateway**

Falló la autenticación de la consulta y la respuesta trae `msg`:

```json
{ "success": false, "msg": "<mensaje>" }
```

El mensaje puede ser `internal server error`, `Token vacio`, o el texto o código que trae la consulta.

#### **500 — Error no capturado**

```json
{ "msg": "Ocurrió un error inesperado" }
```

También responde este 500 cuando `document_number` o `identity_id` llegan en `true`, y cuando la consulta lanza una excepción. En ese último caso el cuerpo es `{"success": false, "msg": "<mensaje fijo>"}` y el detalle interno no sale.

### Proceso de validación

1. **Token** — 401 si es inválido, venció o no trae el claim de identidad. 403 si no hay Bearer, el scope no es `full_access`, el grant no está autorizado, o el origen de un token `dashboard:*` no está permitido.
2. **Schema del body** — 422 antes de llegar al servicio. `document_number: true` e `identity_id: true` salen como 500.
3. **Cupo diario** — 429 al superar 100.000 respuestas `200` en el día.
4. **Contrato** — 200 con `msg` si no existe, el token no puede usarlo, no tiene la consulta habilitada o llegó al límite. 412 si el contrato no contiene este servicio.
5. **Mantenimiento** — 200 con `codError` `"016"` si se pidió la simulación o la consulta no trae `datosCedulas`.
6. **Resultado** — 200 con el registro, 404 si no hay registro utilizable, 502 si falló la autenticación de la consulta, 422 si la consulta no valida y 500 si lanza una excepción o el error no se capturó.

### Interpretación de la respuesta

- **`codError`:** `"0"` en un registro encontrado. `"016"` en mantenimiento, dentro de `estadoConsulta`.
- **`estadoCedula` y `descripcionEstado`:** Definen el estado oficial de la cédula. Usalos como criterio principal (vigente, cancelada, pendiente).
- **`fechaHoraConsulta`:** Momento de la consulta.
- **`nuip`:** Identificador en el registro. Se alinea con el documento consultado cuando aplica.
- En **cancelación por muerte**, suelen informarse `fechaDefuncion`, `fechaAfectacion`, `fechaReferencia`, `informante` y `serial`, además del cambio en `descripcionEstado`.
- Un **200 con solo `msg`** es un resultado de contrato, no un registro.

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
