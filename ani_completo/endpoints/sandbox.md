# Sandbox — `aniValidationSB`

Ruta aparte de [`/aniValidation`](verification.md). Responde con un catálogo fijo de 999 documentos. No consulta registros en vivo.

## POST `/aniValidationSB`

Mismo Bearer que la consulta de producción: scope `full_access` y, si el token trae `grant_type`, `standalone:ani` o `standalone:govchecks`.

```http
Content-Type: application/json
Authorization: Bearer <tu_jwt_token>
```

`document_number` tiene que ir como **texto**, igual que la clave del catálogo. Un número JSON no coincide con el catálogo y responde 412.

```json
{
  "document_number": "13",
  "contract_id": 1
}
```

```bash
curl --location 'https://api.svi.becomedigital.net/api/v1/aniValidationSB' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer tu_jwt_token_aqui' \
--data '{
    "document_number": "13",
    "contract_id": 1
}'
```

### 200 — Documento del catálogo

Devuelve el registro guardado para ese número: nombre, `particula`, lugar y fecha de expedición, `estadoCedula`, `descripcionEstado` y el resto de campos de cédula. `identity_id` es `null`. `codError` es `"0"`.

**Ejemplo — `13`, vigente:**

```json
{
  "identity_id": null,
  "fechaHoraConsulta": "2022-11-29 13:28:46",
  "nuip": "13",
  "codError": "0",
  "primerApellido": "AMIGO",
  "particula": "VIUDA DE",
  "segundoApellido": "GUZMÁN",
  "primerNombre": "BRAIS",
  "segundoNombre": "GERARDO",
  "municipioExpedicion": "PIENDAMO",
  "departamentoExpedicion": "CAUCA",
  "fechaExpedicion": "21/09/1939",
  "estadoCedula": "0",
  "numResolucion": null,
  "anoResolucion": null,
  "genero": null,
  "fechaNacimiento": null,
  "lugarNacimiento": "TERLIZZI-BARI - ITALIA",
  "informante": null,
  "serial": null,
  "fechaDefuncion": null,
  "lugarNovedad": null,
  "lugarPreparacion": null,
  "grupoSanguineo": null,
  "estatura": null,
  "fechaReferencia": null,
  "fechaAfectacion": null,
  "oficinaExpedicion": null,
  "fechaPreparacion": null,
  "descripcionEstado": "VIGENTE"
}
```

### 412 — El documento no está en el catálogo

```json
{ "msg": "documento no valido para sandbox" }
```

También responde 412, antes del catálogo, si falta un campo o `contract_id` no es entero:

```json
{ "msg": "debes indicar document_number asociado" }
```

```json
{ "msg": "debes indicar contract_id asociado" }
```

```json
{ "msg": "el contract_id debe ser un valor de tipo entero" }
```

Si el contrato no pasa la revisión, la ruta responde **403** con el `msg` de esa revisión.

## Números vigentes de ejemplo

967 documentos del catálogo están vigentes (`estadoCedula` `"0"`), cada uno con nombre y datos de expedición. Estos diez también:

| Número | Nombre |
|--------|--------|
| 13 | BRAIS AMIGO |
| 14 | HUGO CORDERA |
| 28159 | ABRAHAM GIONO |
| 123456 | SALOMÉ MENDIOLEA |
| 226821 | IZAN Y TORRES |
| 272839 | BORIS GARCÍA-MANZO |
| 332377 | GINA PEÑALOZA |
| 355191 | ABRIL ÁNGELES |
| 406640 | KATHERINE ESTRADA |
| 454089 | BLAKE MAULÉN |

## Números que no están vigentes

Estos 32 no están vigentes:

| Número | Estado | Descripción |
|--------|--------|-------------|
| 78 | 88 | Pendiente solicitud en reproceso |
| 145, 649, 1995, 9000, 27106, 20000809, 20053653, 39723854, 80831015, 1111111111, 1233489185 | 21 | Cancelada por muerte |
| 2389294, 21676619, 52411271, 79961457, 80831012 | 12 | Vigente con pérdida o suspensión de los derechos políticos |
| 16265309, 31395498, 66724723 | 14 | Vigente con interdicción judicial |
| 391234, 1069128, 29710551, 9999999901 | 22 | Cancelada por doble cedulación |
| 80831016, 80831018 | 27 | Cancelada con reasignación de cupo numérico |
| 1052415629 | 23 | Cancelada por suplantación |
| 80831011, 80831019, 1015393002, 1047362038, 1100549082 | 53 | Cancelada por falsa identidad |
