# Validación CURP México

Endpoint para validar un CURP mexicano frente a los registros oficiales de RENAPO (Registro Nacional de Población). Existen dos versiones de respuesta (**v1** y **v2**) con el mismo request; cambia el formato de la respuesta y el tratamiento del CURP inválido.

## Auth común {#auth-comun}

**Grant type requerido:** `standalone:govchecks`

Solicite el token en **POST** [`/auth`](auth.md) con `"grant_type": "standalone:govchecks"` antes de llamar a estos endpoints.

El contrato debe tener habilitada la validación CURP (`can_mexican_curp`).

### Headers requeridos

```http
Content-Type: application/json
Authorization: Bearer <tu_jwt_token>
```

---

## Request (ambas rutas) {#request}

```json
{
  "curp": "SAGN920221MDFNML01",
  "contract_id": 123
}
```

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| `curp` | string | Sí | CURP a consultar (se normaliza a mayúsculas) |
| `contract_id` | integer | Sí | ID del contrato asignado a tu integración |

### Formato CURP esperado

18 caracteres:

| Pos. | Contenido |
|------|-----------|
| 1–4 | Letras |
| 5–10 | Dígitos (fecha `AAMMDD`) |
| 11 | Sexo: `H`, `M` o `X` |
| 12–13 | Entidad federativa válida (ej. `DF`, `AS`, `NE`; Querétaro = `QO`) |
| 14–16 | Letras |
| 17 | Letra o dígito |
| 18 | Dígito |

### Ejemplo de request (v1)

```bash
curl --location 'https://api.svi.becomedigital.net/api/v1/govchecks/mx/curp' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer <tu_jwt_token>' \
--data '{
    "curp": "SAGN920221MDFNML01",
    "contract_id": 123
}'
```

### Ejemplo de request (v2)

```bash
curl --location 'https://api.svi.becomedigital.net/api/v1/govchecks/mx/curp/v2' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer <tu_jwt_token>' \
--data '{
    "curp": "SAGN920221MDFNML01",
    "contract_id": 123
}'
```

---

## POST `/govchecks/mx/curp` (v1)

### Respuestas

#### ✅ **200 — Consulta exitosa (CURP hallado)** {#v1-200-hallado}

```json
{
  "fechaNacimiento": "1992-02-21",
  "primerApellido": "...",
  "segundoApellido": "...",
  "nombres": "...",
  "sexo": "...",
  "claveEntidad": "...",
  "curp": "SAGN920221MDFNML01",
  "docProbatorio": "",
  "statusCurp": "V",
  "nacionalidad": "MEX",
  "datosDocProbatorio": {
    "tomo": "",
    "foja": "",
    "numActa": "",
    "libro": "",
    "claveMunicipioRegistro": "",
    "anioReg": "",
    "claveEntidadRegistro": ""
  }
}
```

#### ✅ **200 — CURP no hallado** {#v1-200-no-hallado}

Datos personales en `null` (puede conservarse el `curp` enviado) y `statusCurp: "N"`.

#### **401 — Token inválido o ausente** {#v1-401}

```json
{ "detail": "Token invalid or expired" }
```

#### **403 — Grant no autorizado** {#v1-403-grant}

```json
{ "detail": "Token grant type not authorized for this endpoint" }
```

#### **403 — Contrato sin validación CURP** {#v1-403-contrato}

```json
{ "msg": "este contrato no tiene habilitada la validación de CURP" }
```

#### **422 — Body inválido** {#v1-422}

Respuesta estándar de validación (`detail` como arreglo) cuando falta un campo o el tipo es incorrecto.

#### **400 — Contrato no encontrado** {#v1-400-contrato}

```json
{
  "msg": "el contrato ingresado no exite o no cuenta con los permisos para hacer esta consulta"
}
```

#### **400 — Formato de CURP inválido** {#v1-400-formato}

```json
{
  "msg": "el CURP no tiene un formato válido",
  "invalid_curp": "ZANAHORIA",
  "reason": "length"
}
```

Valores posibles de `reason`: `empty` | `length` | `pattern` | `date` | `sex` | `entity`

#### **400 — Error interno** {#v1-400-interno}

```json
{ "msg": "Internal server error" }
```

#### **500 — Error del servicio** {#v1-500}

```json
{
  "fechaNacimiento": "",
  "primerApellido": "",
  "segundoApellido": "",
  "nombres": "",
  "sexo": "",
  "claveEntidad": "",
  "curp": "",
  "docProbatorio": "",
  "statusCurp": "N",
  "nacionalidad": "",
  "datosDocProbatorio": {
    "tomo": "",
    "foja": "",
    "numActa": "",
    "libro": "",
    "claveMunicipioRegistro": "",
    "anioReg": "",
    "claveEntidadRegistro": ""
  }
}
```

---

## POST `/govchecks/mx/curp/v2` (v2)

Misma [autenticación](#auth-comun), grant type y [body](#request) que v1.

### Respuestas

#### ✅ **200 — Consulta exitosa (CURP hallado)**

```json
{
  "adulto": true,
  "fechaNacimiento": "1992-02-21",
  "nombres": "...",
  "primerApellido": "...",
  "segundoApellido": "..."
}
```

`adulto` es `true` o `false` según mayoría de edad (≥18), o `null` si no hay fecha válida.

#### ✅ **200 — No hallado o formato inválido**

En v2 un CURP con formato inválido responde igual que un no hallado: todos los campos en `null`.

```json
{
  "adulto": null,
  "fechaNacimiento": null,
  "nombres": null,
  "primerApellido": null,
  "segundoApellido": null
}
```

#### **401 — Token inválido o ausente**

Igual que en [v1](#v1-401).

#### **403 — Grant no autorizado**

Igual que en [v1](#v1-403-grant).

#### **403 — Contrato sin validación CURP**

Igual que en [v1](#v1-403-contrato).

#### **422 — Body inválido**

Igual que en [v1](#v1-422).

#### **400 — Contrato no encontrado**

```json
{
  "msg": "el contrato ingresado no existe o no cuenta con los permisos para hacer esta consulta"
}
```

#### **400 — Error interno**

Igual que en [v1](#v1-400-interno).

#### **500 — Error del servicio**

```json
{
  "adulto": null,
  "fechaNacimiento": null,
  "nombres": null,
  "primerApellido": null,
  "segundoApellido": null
}
```
