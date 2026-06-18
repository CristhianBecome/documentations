# Ambiente Sandbox (API KYC)

El **Sandbox** permite probar la integración sin consultar bases de datos reales ni procesar archivos subidos. El comportamiento se controla únicamente mediante el **sufijo `-TEST-N`** en el `user_id`.

## Requisitos

- **Credenciales de sandbox** emitidas por Become Digital (no sirven credenciales de producción).
- **Token JWT real** obtenido con esas credenciales mediante el flujo de autenticación habitual ([Autenticación](authentication.md), [POST `/auth`](endpoints/auth.md)).

## Identificador base de prueba

Use el siguiente UUID como prefijo y añada `-TEST-1`, `-TEST-2` o `-TEST-3` según el escenario:

`5e24688f-6c7b-48ba-a144-86bb3b9667fc`

**Ejemplo:** `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-1`

## URL base (misma que producción)

Las rutas y el host son **los mismos** que en producción. Lo que distingue el sandbox son las **credenciales de prueba** y el sufijo **`-TEST-N`** en el `user_id` (sin otro host ni versión distinta).

**URL base:** `https://api.svi.becomedigital.net/api/v1`

### Rutas de los endpoints

La definición completa de cada ruta está en los enlaces de la columna **Ruta**.

| Método | Ruta (definición) | URL completa de ejemplo |
|--------|-------------------|-------------------------|
| **POST** | [`/auth`](endpoints/auth.md) | `https://api.svi.becomedigital.net/api/v1/auth` |
| **POST** | [`/newIdentity`](endpoints/verification-add.md) | `https://api.svi.becomedigital.net/api/v1/newIdentity` |
| **GET** | [`/identity/<user_id>`](endpoints/verification-results.md) | `https://api.svi.becomedigital.net/api/v1/identity/<user_id>` |
| **POST** | [`/matches`](endpoints/re-verification.md) | `https://api.svi.becomedigital.net/api/v1/matches` |
| **POST** | [`/matches/check`](endpoints/re-verification-results.md) | `https://api.svi.becomedigital.net/api/v1/matches/check` |

Autentíquese con **POST** [`/auth`](endpoints/auth.md) usando las credenciales **de sandbox** cuando pruebe escenarios `-TEST-*`. No mezcle credenciales de producción con flujos de prueba. Flujo de acceso: [Autenticación](authentication.md).

### Disponibilidad: API directa vs SDK

Los escenarios sandbox de **POST** [`/newIdentity`](endpoints/verification-add.md) y **POST** [`/matches`](endpoints/re-verification.md) están pensados **solo para integración vía API REST** (consulta directa con `curl`, Postman o tu backend).

| Canal | Sandbox (`-TEST-*`) | Prueba de vida (liveness) |
|-------|---------------------|---------------------------|
| **API REST directa** | ✅ Disponible con credenciales y token de sandbox | No aplica en sandbox (respuestas simuladas; `liveness` puede ser `null`) |
| **SDK web / móvil** | ❌ No disponible | **Obligatoria** — el SDK ejecuta captura real de selfie y liveness en producción |

Si integras con el **SDK**, debes usar credenciales y entorno de **producción**; el SDK no admite los `user_id` de prueba ni omite la prueba de vida. Para probar el flujo completo con liveness real, use el SDK contra producción con usuarios reales completados en onboarding.

---

## POST `/newIdentity`

**Definición del endpoint (parámetros, archivos, errores):** [Verificación de identidad — crear →](endpoints/verification-add.md)

**URL completa:** `POST https://api.svi.becomedigital.net/api/v1/newIdentity`

**Grant type (`grant_type`):** `verification:create`

Envíe `user_id` en **multipart/form-data** (igual que en producción). Los archivos pueden ser ficticios: **no se procesan**; la respuesta depende solo del sufijo `-TEST-N`.

> **Solo API directa:** Este flujo sandbox no aplica al SDK. Ver [Disponibilidad: API directa vs SDK](#disponibilidad-api-directa-vs-sdk).

### Escenarios

| `user_id` (termina en) | HTTP | Descripción |
|------------------------|------|-------------|
| `…-TEST-1` | **201** | Identidad creada — devuelve `url_resource` y `user_id` |
| `…-TEST-2` | **200** | Ya existe un registro con ese ID para la compañía |
| `…-TEST-3` | **400** | Error en el envío de archivos (simulado) |

### Sufijo `…-TEST-1` — 201 Creado

```json
{
  "code": 201,
  "message": "El recurso fue creado",
  "url_resource": "https://api.svi.becomedigital.net/api/v1/identity/5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-1",
  "user_id": "5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-1"
}
```

### Sufijo `…-TEST-2` — 200 Ya existe

```json
{
  "code": 200,
  "message": "Ya existe un registro con el mismo ID para esta compañia",
  "url_resource": "https://api.svi.becomedigital.net/api/v1/identity/5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-2"
}
```

### Sufijo `…-TEST-3` — 400 Error

```json
{
  "code": 400,
  "message": "Hubo un problema con el envio de los archivos"
}
```

---

## GET `/identity/<user_id>`

**Definición del endpoint (respuestas, campos de la respuesta, errores):** [Consulta de resultados →](endpoints/verification-results.md)

**URL completa:** `GET https://api.svi.becomedigital.net/api/v1/identity/<user_id>`

**Grant type (`grant_type`):** `verification:get`

Mismo contrato que producción: header `Authorization: Bearer <token>`. El resultado está determinado por el sufijo del `user_id`.

### Escenarios

| `user_id` (termina en) | HTTP | Descripción |
|------------------------|------|-------------|
| `…-TEST-1` | **200** | Verificación `completed` — datos **aleatorios** en cada llamada |
| `…-TEST-2` | **202** | Verificación `pending` — flags de `verification` en `null` (aún no “terminó” el flujo simulado) |
| `…-TEST-3` | **404** | Usuario no encontrado |

### Bloques dinámicos (`TEST-1`)

En el escenario **TEST-1**, cada llamada a `GET /identity/<user_id>` puede variar (probabilidades orientativas):

| Bloque / comportamiento | Probabilidad aproximada |
|-------------------------|-------------------------|
| Objeto `registry` | ~90 % |
| `ip_validation` | ~50 % |
| `one_to_many` (resultado y score) | ~40 % |
| Flag `ubica` en `verification` | ~40 % |
| `estimated_age` | ~50 % |
| `ubica_response` | ~30 % (si `ubica` está activo) |
| `selfiImageUrl` en `media` | ~70 % |

La persona simulada y los valores numéricos (`face_match_score`, etc.) también **rotan de forma aleatoria** entre llamadas.

### Sufijo `…-TEST-1` — 200 Completada (ejemplo con bloques dinámicos activos)

```json
{
  "id": 45231,
  "contract_id": 12,
  "company": "TEST",
  "created_at": "Wed, 15 Dec 2021 20:26:14 GMT",
  "fullname": "JULIAN ALFONSO HERRERA ZAPATA",
  "first_name": "JULIAN ALFONSO",
  "last_name": "HERRERA ZAPATA",
  "birth": "1986-08-03",
  "document_type": "national-id",
  "document_number": "18940234",
  "data_policy_consent": "S",
  "verification": {
    "face_match": true,
    "template": true,
    "alteration": true,
    "watch_list": true,
    "verification_status": "completed",
    "liveness": true,
    "ubica": true,
    "estimated_age": true,
    "one_to_many_result": true,
    "ip_validation": true
  },
  "face_match_score": 97.43,
  "estimated_age": 36,
  "one_to_many_user_id_matched": "user-1toN-342",
  "one_to_many_score": 91.28,
  "ip_validation": {
    "country": "Colombia",
    "region": "Atlantico",
    "city": "Barranquilla",
    "latitude": 11.0071,
    "longitude": -74.8092,
    "is_safe": true
  },
  "registry": {
    "fullName": "HERRERA ZAPATA JULIAN ALFONSO",
    "issuePlace": "BARRANQUILLA",
    "issueDepartment": "ATLANTICO",
    "emissionDate": "Wed, 06 Aug 2004 12:00:00",
    "ageRange": "35-40",
    "name": "JULIAN ALFONSO",
    "middleName": null,
    "surname": "HERRERA",
    "secondSurname": "ZAPATA",
    "gender": "M",
    "status": "VIGENTE",
    "nuip": "18940234",
    "savingAccountsCount": 3,
    "bankAccountsCount": 2,
    "financialIndustryDebtsCount": 1,
    "solidarityIndustryDebtsCount": 0,
    "serviceIndustryDebtsCount": 2,
    "commercialIndustryDebtsCount": 0
  },
  "inactive_status": null,
  "ubica_response": {
    "full_name": "JULIAN ALFONSO HERRERA ZAPATA",
    "document_number": "18940234",
    "emission_date": "2004-08-06",
    "age_range": "35-40",
    "document_type": "CEDULA DE CIUDADANIA",
    "expedition_place": "BARRANQUILLA",
    "code_department": "08",
    "code_municipality": "001",
    "document_status": "VIGENTE",
    "application_id": "UBP-20240115-001234",
    "cellphones": [
      {
        "cellphone_number": "3001234567",
        "is_activate": true,
        "last_report": "2024-01-15",
        "first_report": "2018-03-22",
        "economic_sector": "TELECOMUNICACIONES"
      }
    ],
    "addreses": [
      {
        "addresses": "CRA 50 # 72-10",
        "city": "BARRANQUILLA",
        "last_report": "2023-11-10",
        "first_report": "2015-06-01",
        "economic_sector": "SERVICIOS",
        "location_type": "RESIDENCIA"
      }
    ],
    "mails": [
      {
        "email": "julian.herrera@example.com",
        "last_report": "2024-01-10",
        "first_report": "2019-07-15"
      }
    ],
    "phones": [
      {
        "phone_number": "5791234",
        "city": "BARRANQUILLA",
        "last_report": "2022-08-20",
        "first_report": "2010-01-05",
        "is_activate": false,
        "economic_sector": "SERVICIOS",
        "location_type": "RESIDENCIA",
        "prefix": "05"
      }
    ]
  },
  "ip": "181.134.87.23",
  "media": {
    "frontImgUrl": "https://api.svi.becomedigital.net/api/v1/media/5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-1/frontImg",
    "backImgUrl": "https://api.svi.becomedigital.net/api/v1/media/5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-1/backImg",
    "selfiImageUrl": "https://api.svi.becomedigital.net/api/v1/media/5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-1/selfieImg"
  },
  "uuid": "a3f8c2d1e9b047..."
}
```

### Sufijo `…-TEST-2` — 202 Pendiente

Este escenario **simula** una verificación aún en curso: en el objeto `verification`, los indicadores (`face_match`, `template`, `alteration`, etc.) van en `null` porque en producción aún no habrían resultado los chequeos; cuando el estado pasa a `completed`, esos mismos campos traen `true` o `false`.

```json
{
  "id": 32891,
  "contract_id": 7,
  "company": "TEST",
  "created_at": "Wed, 15 Dec 2021 20:26:14 GMT",
  "fullname": "MARIA ELENA TORRES RINCON",
  "first_name": "MARIA ELENA",
  "last_name": "TORRES RINCON",
  "birth": "1992-03-15",
  "document_type": "national-id",
  "document_number": "52741836",
  "data_policy_consent": "S",
  "verification": {
    "face_match": null,
    "template": null,
    "alteration": null,
    "watch_list": null,
    "verification_status": "pending",
    "liveness": null,
    "ubica": null
  },
  "inactive_status": null,
  "media": {
    "frontImgUrl": "https://api.svi.becomedigital.net/api/v1/media/5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-2/frontImg",
    "backImgUrl": "https://api.svi.becomedigital.net/api/v1/media/5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-2/backImg",
    "selfiImageUrl": "https://api.svi.becomedigital.net/api/v1/media/5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-2/selfieImg"
  },
  "uuid": "b7e1d4f2c8a093..."
}
```

### Sufijo `…-TEST-3` — 404 No encontrado

```json
{
  "msg": "No se encontró el usuario con ID 5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-3 para la compañía 1"
}
```

> El texto del mensaje puede incluir el `user_id` y el identificador de compañía según el entorno.

---

## POST `/matches`

**Definición del endpoint (parámetros, archivos, errores):** [Re-verificación (autenticación biométrica) →](endpoints/re-verification.md)

**URL completa:** `POST https://api.svi.becomedigital.net/api/v1/matches`

**Grant type (`grant_type`):** `reverification:create`

En sandbox solo necesitas `user_id` en **multipart/form-data**. No hace falta `image`, `contract_id`, contrato activo ni registro en BD; la respuesta depende del sufijo `-TEST-N` (mismo patrón que `/newIdentity` y `/identity`).

> **Solo API directa:** Este flujo sandbox no aplica al SDK. Con el SDK la prueba de vida es obligatoria y real; ver [Disponibilidad: API directa vs SDK](#disponibilidad-api-directa-vs-sdk).

### IDs sandbox (`/matches` y `/matches/check`)

Use estos identificadores **completos** (fijos). En sandbox, `executionId` devuelto por **POST** `/matches` coincide con el `user_id` enviado; en **POST** `/matches/check` use el mismo valor en `executionId`.

| ID | Escenario |
|----|-----------|
| `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-1` | Coincidencia facial exitosa |
| `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-2` | Sin coincidencia facial |
| `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-3` | Recurso no encontrado |

### Escenarios (`POST /matches`)

| ID | HTTP | Descripción |
|----|------|-------------|
| `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-1` | **200** | Coincidencia facial exitosa — `result: true`, `confidence: 99.87` |
| `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-2` | **200** | Sin coincidencia facial — `result: false`, `confidence: 32.15` |
| `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-3` | **404** | `No se encontró una identidad válida para este usuario` |

El campo `company` refleja el `client_id` del token JWT.

### `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-1` — 200 Match exitoso

```bash
curl -X POST "https://api.svi.becomedigital.net/api/v1/matches" \
  -H "Authorization: Bearer <JWT>" \
  -F "user_id=5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-1"
```

```json
{
  "company": "<client_id_del_token>",
  "user_id": "5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-1",
  "result": true,
  "confidence": 99.87,
  "liveness": null,
  "executionId": "5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-1"
}
```

### `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-2` — 200 Sin match

Mismo request con `user_id=5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-2`:

```json
{
  "company": "<client_id_del_token>",
  "user_id": "5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-2",
  "result": false,
  "confidence": 32.15,
  "liveness": null,
  "executionId": "5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-2"
}
```

### `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-3` — 404 Recurso no encontrado

```json
{
  "msg": "No se encontró una identidad válida para este usuario"
}
```

### Producción (cualquier otro `user_id`)

Fuera de sandbox, **POST** `/matches` exige `user_id`, `contract_id`, `image` y una identidad completada en BD. Errores típicos: **400** sin parámetros obligatorios, **404** sin identidad previa.

---

## POST `/matches/check`

**Definición del endpoint (respuestas, errores):** [Resultados de re-verificación →](endpoints/re-verification-results.md)

**URL completa:** `POST https://api.svi.becomedigital.net/api/v1/matches/check`

**Grant type (`grant_type`):** `reverification:create`

**Content-Type:** `application/json`. Campos: `contract_id`, `executionId`. En sandbox `contract_id` puede ser cualquier valor (p. ej. `"1"`).

### Escenarios (`POST /matches/check`)

Mismos IDs que en [IDs sandbox](#ids-sandbox-matches-y-matchescheck). El `contract_id` puede ser cualquier valor (p. ej. `"1"`).

| `executionId` | HTTP | Descripción |
|---------------|------|-------------|
| `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-1` | **200** | Validación encontrada — `result: true`, `confidence: 99.87`, `timestamp` = hora del servidor |
| `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-2` | **200** | Validación encontrada — `result: false`, `confidence: 32.15` |
| `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-3` | **404** | `No se encontró una validación con el executionId proporcionado` |

### `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-1` — 200

```bash
curl -X POST "https://api.svi.becomedigital.net/api/v1/matches/check" \
  -H "Authorization: Bearer <JWT>" \
  -H "Content-Type: application/json" \
  -d '{
    "contract_id": "1",
    "executionId": "5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-1"
  }'
```

Respuesta con `result: true`, `confidence: 99.87` y `timestamp` en la hora actual del servidor.

### `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-2` — 200

Mismo body con `executionId`: `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-2` → `result: false`, `confidence: 32.15`.

### `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-3` — 404

```json
{
  "msg": "No se encontró una validación con el executionId proporcionado"
}
```

### Flujo end-to-end recomendado (sandbox)

1. **POST** `/matches` con `user_id=5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-1`
2. Tomar el `executionId` de la respuesta (en sandbox: `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-1`)
3. **POST** `/matches/check` con ese `executionId` y `contract_id` arbitrario (p. ej. `"1"`)

---

## Referencias

- [POST `/auth` — Autenticación JWT](endpoints/auth.md)
- [POST `/newIdentity` — Crear verificación](endpoints/verification-add.md)
- [GET `/identity/<user_id>` — Consultar resultados](endpoints/verification-results.md)
- [POST `/matches` — Re-verificación (cotejo facial)](endpoints/re-verification.md)
- [POST `/matches/check` — Consultar re-verificación](endpoints/re-verification-results.md)
- [Credenciales y acceso](authentication.md)
