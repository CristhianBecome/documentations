# Sandbox Environment (KYC API)

The **Sandbox** lets you test the integration without querying real databases or processing uploaded files. Behavior is controlled only by the **`-TEST-N` suffix** on `user_id`.

## Requirements

- **Sandbox credentials** issued by Become Digital (production credentials will not work).
- A **real JWT token** obtained with those credentials through the usual authentication flow ([Authentication](authentication.md), [POST `/auth`](endpoints/auth.md)).

## Base test identifier

Use the following UUID as a prefix and append `-TEST-1`, `-TEST-2`, or `-TEST-3` depending on the scenario:

`5e24688f-6c7b-48ba-a144-86bb3b9667fc`

**Example:** `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-1`

## Base URL (same as production)

Routes and host are **the same** as production. What distinguishes sandbox are **test credentials** and the **`-TEST-N`** suffix on `user_id` (no different host or version).

**Base URL:** `https://api.svi.becomedigital.net/api/v1`

### Endpoint routes

The full definition of each route is in the links in the **Route** column.

| Method | Route (definition) | Full example URL |
|--------|-------------------|------------------|
| **POST** | [`/auth`](endpoints/auth.md) | `https://api.svi.becomedigital.net/api/v1/auth` |
| **POST** | [`/newIdentity`](endpoints/verification-add.md) | `https://api.svi.becomedigital.net/api/v1/newIdentity` |
| **GET** | [`/identity/<user_id>`](endpoints/verification-results.md) | `https://api.svi.becomedigital.net/api/v1/identity/<user_id>` |
| **POST** | [`/matches`](endpoints/re-verification.md) | `https://api.svi.becomedigital.net/api/v1/matches` |
| **POST** | [`/matches/check`](endpoints/re-verification-results.md) | `https://api.svi.becomedigital.net/api/v1/matches/check` |

Authenticate with **POST** [`/auth`](endpoints/auth.md) using **sandbox** credentials when testing `-TEST-*` scenarios. Do not mix production credentials with test flows. Access flow: [Authentication](authentication.md).

### Availability: direct API vs SDK

Sandbox scenarios for **POST** [`/newIdentity`](endpoints/verification-add.md) and **POST** [`/matches`](endpoints/re-verification.md) are intended **only for REST API integration** (direct calls with `curl`, Postman, or your backend).

| Channel | Sandbox (`-TEST-*`) | Liveness |
|---------|---------------------|----------|
| **Direct REST API** | ✅ Available with sandbox credentials and token | Not applicable in sandbox (simulated responses; `liveness` may be `null`) |
| **Web / mobile SDK** | ❌ Not available | **Required** — the SDK runs real selfie capture and liveness in production |

If you integrate with the **SDK**, you must use **production** credentials and environment; the SDK does not accept test `user_id` values and does not skip liveness. To test the full flow with real liveness, use the SDK against production with real users who completed onboarding.

---

## POST `/newIdentity`

**Endpoint definition (parameters, files, errors):** [Identity verification — create →](endpoints/verification-add.md)

**Full URL:** `POST https://api.svi.becomedigital.net/api/v1/newIdentity`

**Grant type (`grant_type`):** `verification:create`

Send `user_id` as **multipart/form-data** (same as production). Files may be dummy: **they are not processed**; the response depends only on the `-TEST-N` suffix.

> **Direct API only:** This sandbox flow does not apply to the SDK. See [Availability: direct API vs SDK](#availability-direct-api-vs-sdk).

### Scenarios

| `user_id` (ends with) | HTTP | Description |
|------------------------|------|-------------|
| `…-TEST-1` | **201** | Identity created — returns `url_resource` and `user_id` |
| `…-TEST-2` | **200** | A record with that ID already exists for the company |
| `…-TEST-3` | **400** | File upload error (simulated) |

### Suffix `…-TEST-1` — 201 Created

```json
{
  "code": 201,
  "message": "El recurso fue creado",
  "url_resource": "https://api.svi.becomedigital.net/api/v1/identity/5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-1",
  "user_id": "5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-1"
}
```

### Suffix `…-TEST-2` — 200 Already exists

```json
{
  "code": 200,
  "message": "Ya existe un registro con el mismo ID para esta compañia",
  "url_resource": "https://api.svi.becomedigital.net/api/v1/identity/5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-2"
}
```

### Suffix `…-TEST-3` — 400 Error

```json
{
  "code": 400,
  "message": "Hubo un problema con el envio de los archivos"
}
```

---

## GET `/identity/<user_id>`

**Endpoint definition (responses, response fields, errors):** [Results lookup →](endpoints/verification-results.md)

**Full URL:** `GET https://api.svi.becomedigital.net/api/v1/identity/<user_id>`

**Grant type (`grant_type`):** `verification:get`

Same contract as production: `Authorization: Bearer <token>` header. The result is determined by the `user_id` suffix.

### Scenarios

| `user_id` (ends with) | HTTP | Description |
|------------------------|------|-------------|
| `…-TEST-1` | **200** | Verification `completed` — **random** data on each call |
| `…-TEST-2` | **202** | Verification `pending` — `verification` flags are `null` (simulated flow not “finished” yet) |
| `…-TEST-3` | **404** | User not found |

### Dynamic blocks (`TEST-1`)

In the **TEST-1** scenario, each call to `GET /identity/<user_id>` may vary (approximate probabilities):

| Block / behavior | Approximate probability |
|------------------|-------------------------|
| `registry` object | ~90 % |
| `ip_validation` | ~50 % |
| `one_to_many` (result and score) | ~40 % |
| `ubica` flag in `verification` | ~40 % |
| `estimated_age` | ~50 % |
| `ubica_response` | ~30 % (if `ubica` is active) |
| `selfiImageUrl` in `media` | ~70 % |

The simulated person and numeric values (`face_match_score`, etc.) also **rotate randomly** between calls.

### Suffix `…-TEST-1` — 200 Completed (example with dynamic blocks active)

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

### Suffix `…-TEST-2` — 202 Pending

This scenario **simulates** a verification still in progress: in the `verification` object, indicators (`face_match`, `template`, `alteration`, etc.) are `null` because in production those checks would not have returned yet; when the status becomes `completed`, those same fields return `true` or `false`.

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

### Suffix `…-TEST-3` — 404 Not found

```json
{
  "msg": "No se encontró el usuario con ID 5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-3 para la compañía 1"
}
```

> The message text may include the `user_id` and company identifier depending on the environment.

---

## POST `/matches`

**Endpoint definition (parameters, files, errors):** [Re-verification (biometric authentication) →](endpoints/re-verification.md)

**Full URL:** `POST https://api.svi.becomedigital.net/api/v1/matches`

**Grant type (`grant_type`):** `reverification:create`

In sandbox you only need `user_id` as **multipart/form-data**. No `image`, `contract_id`, active contract, or DB record is required; the response depends on the `-TEST-N` suffix (same pattern as `/newIdentity` and `/identity`).

> **Direct API only:** This sandbox flow does not apply to the SDK. With the SDK, liveness is required and real; see [Availability: direct API vs SDK](#availability-direct-api-vs-sdk).

### Sandbox IDs (`/matches` and `/matches/check`)

Use these **full** (fixed) identifiers. In sandbox, the `executionId` returned by **POST** `/matches` matches the `user_id` sent; in **POST** `/matches/check` use the same value as `executionId`.

| ID | Scenario |
|----|----------|
| `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-1` | Successful facial match |
| `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-2` | No facial match |
| `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-3` | Resource not found |

### Scenarios (`POST /matches`)

| ID | HTTP | Description |
|----|------|-------------|
| `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-1` | **200** | Successful facial match — `result: true`, `confidence: 99.87` |
| `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-2` | **200** | No facial match — `result: false`, `confidence: 32.15` |
| `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-3` | **404** | `No se encontró una identidad válida para este usuario` |

The `company` field reflects the `client_id` from the JWT token.

### `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-1` — 200 Successful match

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

### `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-2` — 200 No match

Same request with `user_id=5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-2`:

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

### `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-3` — 404 Resource not found

```json
{
  "msg": "No se encontró una identidad válida para este usuario"
}
```

### Production (any other `user_id`)

Outside sandbox, **POST** `/matches` requires `user_id`, `contract_id`, `image`, and a completed identity in the DB. Typical errors: **400** without required parameters, **404** without a prior identity.

---

## POST `/matches/check`

**Endpoint definition (responses, errors):** [Re-verification results →](endpoints/re-verification-results.md)

**Full URL:** `POST https://api.svi.becomedigital.net/api/v1/matches/check`

**Grant type (`grant_type`):** `reverification:create`

**Content-Type:** `application/json`. Fields: `contract_id`, `executionId`. In sandbox `contract_id` can be any value (e.g. `"1"`).

### Scenarios (`POST /matches/check`)

Same IDs as in [Sandbox IDs](#sandbox-ids-matches-and-matchescheck). `contract_id` can be any value (e.g. `"1"`).

| `executionId` | HTTP | Description |
|---------------|------|-------------|
| `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-1` | **200** | Validation found — `result: true`, `confidence: 99.87`, `timestamp` = server time |
| `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-2` | **200** | Validation found — `result: false`, `confidence: 32.15` |
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

Response with `result: true`, `confidence: 99.87`, and `timestamp` at the current server time.

### `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-2` — 200

Same body with `executionId`: `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-2` → `result: false`, `confidence: 32.15`.

### `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-3` — 404

```json
{
  "msg": "No se encontró una validación con el executionId proporcionado"
}
```

### Recommended end-to-end flow (sandbox)

1. **POST** `/matches` with `user_id=5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-1`
2. Take the `executionId` from the response (in sandbox: `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-1`)
3. **POST** `/matches/check` with that `executionId` and an arbitrary `contract_id` (e.g. `"1"`)

---

## References

- [POST `/auth` — JWT Authentication](endpoints/auth.md)
- [POST `/newIdentity` — Create verification](endpoints/verification-add.md)
- [GET `/identity/<user_id>` — Look up results](endpoints/verification-results.md)
- [POST `/matches` — Re-verification (facial matching)](endpoints/re-verification.md)
- [POST `/matches/check` — Look up re-verification](endpoints/re-verification-results.md)
- [Credentials and access](authentication.md)
