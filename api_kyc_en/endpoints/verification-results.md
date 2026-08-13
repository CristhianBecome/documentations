# KYC - Results Lookup

Look up the status and full result of a previously created identity verification.

> **Sandbox:** Responses may be simulated based on the `-TEST-1`, `-TEST-2`, or `-TEST-3` suffix on `user_id` (no real verification data). See [Sandbox Environment](../sandbox.md).

## GET `/identity/<user_id>`

**Required grant type:** `verification:get`

Request the token with **POST** [`/auth`](auth.md) using `"grant_type": "verification:get"` before calling this endpoint.

### Required headers

```http
Authorization: Bearer <your_jwt_token>
```

### Parameters

| Parameter | Location | Type   | Description                                                       |
| --------- | -------- | ------ | ----------------------------------------------------------------- |
| `user_id` | URL      | string | Unique user identifier defined when creating the verification |

### Request example

```bash
curl --location 'https://api.svi.becomedigital.net/api/v1/identity/usuario_demo_20250115' \
--header 'Authorization: Bearer <your_jwt_token>'
```

## Processing times

**Typical times:**

* Standard verifications: 10-30 seconds
* With additional validations: 30-60 seconds
* With risk alerts: Up to 2 minutes

**Factors that affect timing:**

* Image quality (resolution, sharpness, lighting)
* Fraud detection (screens, photocopies, alterations)
* Additional services (ANI, Telco, Email)

**Recommendation:** Wait 15-30 seconds before the first lookup. If status is `pending`, poll every 10-15 seconds until complete (maximum 2 minutes).

## Responses

### ✅ **200 - Verification completed**

```json
{
  "id": 12345,
  "contract_id": 100,
  "company": "Empresa Demo",
  "created_at": "Sat, 25 Jul 2020 15:40:00 GMT",
  "fullname": "MARIA ELENA RODRIGUEZ SANTOS",
  "first_name": "MARIA ELENA",
  "last_name": "RODRIGUEZ SANTOS",
  "birth": "1985-12-03",
  "birth_place": "MEDELLIN (ANTIOQUIA)",
  "document_type": "national-id",
  "document_number": "9876543210",
  "dni_number": "9876543210",
  "gender": "F",
  "emission_date": "Tue, 22 Aug 2018 00:00:00 GMT",
  "expiration_date": "Wed, 22 Aug 2033 00:00:00 GMT",
  "type_id": "TYPE_ID",
  "face_match_score": 95.5,
  "estimated_age": 38,
  "processingStatus": "SUCCESS",
  "frontProcessingStatus": "SUCCESS",
  "backProcessingStatus": "SUCCESS",
  "error_code": "1000 - OK",
  "verification": {
    "verification_status": "completed",
    "face_match": true,
    "liveness": true,
    "alteration": true,
    "template": true,
    "estimated_age": true,
    "one_to_many_result": true,
    "watch_list": false,
    "liveness_doc": null,
    "ubica": null
  },
  "one_to_many_score": 0.0,
  "one_to_many_user_id_matched": null,
  "media": {
    "selfiImageUrl": "https://api.svi.becomedigital.net/api/v1/media/abc123-def456-ghi789/selfieImg",
    "frontImgUrl": "https://api.svi.becomedigital.net/api/v1/media/abc123-def456-ghi789/frontImg",
    "backImgUrl": "https://api.svi.becomedigital.net/api/v1/media/abc123-def456-ghi789/backImg"
  },
  "userAgent": {
    "browser_name": "Chrome",
    "browser_version": "118.0.0",
    "os_name": "Windows",
    "os_version": "10",
    "device_type": "pc",
    "device_vendor": "Unknown",
    "device_model": "Unknown"
  },
  "dataMatchResult": {
    "dataMatchResult": "SUCCESS",
    "dateOfBirth": "SUCCESS",
    "documentNumber": "SUCCESS"
  },
  "ip": "203.45.67.123",
  "data_policy_consent": "S",
  "status": "1",
  "uuid": "a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0u1v2w3x4y5z6a7b8c9d0e1f2"
}
```

### ⏳ **202 - Verification pending**

```json
{
  "id": 12346,
  "contract_id": "23",
  "company": "Mi Empresa",
  "created_at": "2025-01-15T09:22:00Z",
  "fullname": null,
  "document_type": "national-id",
  "document_number": null,
  "verification": {
    "verification_status": "pending",
    "face_match": null,
    "liveness": null,
    "alteration": null,
    "template": null
  }
}
```

**Explanation:** When status is `pending`, most fields are `null` because processing has not finished. Check again in 10-15 seconds.

## Interpreting results

### Main fields

**Basic information:**

* `id` - Internal verification ID
* `contract_id` - Contract ID used
* `company` - Company name
* `created_at` - Creation date and time
* `uuid` - Unique hash of the verification state (changes with each update)

**Data extracted from the document (OCR):**

* `fullname` - Full name
* `first_name` - First name
* `last_name` - Last names
* `document_type` - Type: `national-id`, `passport`, `driving-license`
* `document_number` - Document number
* `dni_number` - DNI/NUIP number
* `birth` - Date of birth
* `birth_place` - Place of birth
* `gender` - Gender (M/F)
* `emission_date` - Document issue date
* `expiration_date` - Document expiration date

**Biometric results:**

* `face_match_score` - Facial match score (0-100)
* `estimated_age` - Estimated age of the face

**OCR processing statuses (informational):**

* `processingStatus` - Overall OCR status
* `frontProcessingStatus` - OCR status for front image
* `backProcessingStatus` - OCR status for back image
* `error_code` - Error code (e.g.: "1000 - OK")

**Possible values for processingStatus:**

* `SUCCESS` - Successful processing
* `INVALID_CHARACTERS_FOUND` - Invalid characters found
* `Exception` - General processing error
* `AWAITING_OTHER_SIDE` - Waiting to process the other side of the document
* `Recognition process failed to complete in time.` - Recognition process did not complete in time
* `SCANNING_WRONG_SIDE` - Scanning the wrong side of the document
* `RequestException` - Request error
* `UNSUPPORTED_CLASS` - Unsupported document type
* `Unsupported image type` - Unsupported image type
* `LOW_CONFIDENCE` - Low recognition confidence
* `BARCODE_RECOGNITION_FAILED` - Barcode reading failed
* `MRZ_PARSING_FAILED` - Machine-readable zone parsing failed
* `FIELD_IDENTIFICATION_FAILED` - Field identification failed
* `MANDATORY_FIELD_MISSING` - Mandatory field missing
* `IMAGE_PREPROCESSING_FAILED` - Image preprocessing failed

**Other fields:**

* `type_id` - Detected ID type (see [Detected document types](verification-results.md#detected-document-types))
* `status` - Numeric status
* `ip` - User IP address
* `data_policy_consent` - Policy consent (S/N)

### `verification` object (Validation summary)

**⚠️ Important:** This object is a **quick summary** of the main validations.

| Field                 | Description                             | Values                          |
| --------------------- | --------------------------------------- | ------------------------------- |
| `verification_status` | Overall status                          | `pending`, `completed`, `error` |
| `face_match`          | Facial match selfie vs document         | `true`, `false`, `null`         |
| `liveness`            | Liveness check (SDK only)               | `true`, `false`, `null`         |
| `alteration`          | Alteration detection                    | `true`, `false`, `null`         |
| `template`            | Template validation                     | `true`, `false`, `null`         |
| `estimated_age`       | Estimated age matches                   | `true`, `false`, `null`         |
| `one_to_many_result`  | 1:N match (duplicate found)             | `true`, `false`, `null`         |
| `watch_list`          | On watch lists                          | `true`, `false`, `null`         |
| `ip_validation`       | IP validation                           | `true`, `false`, `null`         |

**Interpreting values:**

* ✅ **`true`** = Successful validation, all clear
* ⚠️ **`false`** = A problem or alert was detected
* ℹ️ **`null`** = Not applicable or not configured in the contract

### Alteration and template reasons

**When `alteration: false` or `template: false`**, the system has detected specific issues that may include:

**Alteration reasons (`alteration: false`):**

* **Civil registry (ANI) issues:** Invalid status, missing or inconsistent names
* **Blacklist:** User detected on watch lists
* **Document validation:** Failures validating the document number
* **Document status:** Photocopies, screen photos, or manipulated documents
* **AI model:** High-probability alteration detection (90%+)
* **Data Match:** Inconsistencies between front and back document information

**Template reasons (`template: false`):**

* **OCR issues:** Empty data or failure identifying fields
* **Image quality:** Document heavily damaged, blurry, or unreadable
* **Country issues:** Country does not match, not in the client’s accepted country list
* **Type issues:** Incorrect or unrecognized document type
* **Format issues:** Unrecognized template, invalid format, or processing error
* **Validation issues:** Expired document or inconsistent information

**Important:**

* **You only see the flag:** The API only returns `alteration: false` or `template: false` when it detects problems
* **Specific reason:** Knowing the exact reason (specific code) requires **manual review** by the Become Digital team
* **Purpose:** The flag is an immediate alert so the client knows there is a problem and can take action (reject, escalate, etc.)
* **Later review:** If you need the specific reason for legal or audit cases, contact support with the corresponding `user_id`

**Examples:**

* `alteration: true` → Alterations were **not** detected ✅
* `alteration: false` → Alterations **were** detected ⚠️
* `watch_list: true` → **Not** on watch lists ✅
* `watch_list: false` → **Is** on watch lists ⚠️
* `one_to_many_result: false` → A duplicate **was** found in the account ⚠️
* `one_to_many_result: true` → No duplicate was found in the account ✅

### Note on Liveness

**⚠️ Liveness is an exclusive feature of the Become Digital SDK.**

* **SDK requests:** The `liveness` flag will be `true` or `false`
* **Direct API requests:** The `liveness` flag will be `null`

If you integrate directly via REST API without the SDK, liveness validation is not performed and this field always returns `null`.

### `media` object

URLs to access processed images:

* `selfiImageUrl` - User selfie/video
* `frontImgUrl` - Front document image
* `backImgUrl` - Back image (not included for passports)

### 1:N match (One-to-Many)

**⚠️ Important:** This service compares the current face with **all prior validations for the same client account**.

If the 1:N service is enabled and finds a match:

* `one_to_many_result` - `false` if the face matches a prior validation in the same account (duplicate found)
* `one_to_many_user_id_matched` - User ID of the prior validation it matched
* `one_to_many_score` - Similarity score (0-1, where 1 is identical)

**How it works:**

* ✅ Compares the current face with prior validations **in the same account**
* ✅ If it matches a different `user_id` in the same account → `one_to_many_result: false` (duplicate detected)
* ❌ Does **NOT** cross information with other internal accounts/clients
* ❌ Does **NOT** access external databases

### `userAgent` object

Device information used:

```json
{
  "browser_name": "Chrome Mobile",
  "browser_version": "140.0.0",
  "os_name": "Android",
  "os_version": "10",
  "device_type": "mobile",
  "device_vendor": "Generic_Android",
  "device_model": "K"
}
```

### `dataMatchResult` object

Document data match results:

* Possible statuses: `SUCCESS`, `FAILED`, `NOT_PERFORMED`
* Validates: `dateOfBirth`, `dateOfExpiry`, `documentNumber`, etc.

## Additional conditional fields

Depending on contract configuration and enabled services, the response may include:

### Email validation (if `email_address` was sent)

```json
{
  "email_validation": {
    "recent_abuse": false,
    "fraud_score": 15,
    "first_seen": "1 month",
    "domain_age": "10 years",
    "deliverability": "high",
    "dns_valid": true,
    "spf_record": true,
    "dmarc_record": true,
    "honeypot": false,
    "frequent_complainer": false,
    "suspect": false,
    "spam_trap_score": "none",
    "risky_tld": false,
    "sanitized_email": "maria.rodriguez@empresa.com"
  },
  "verification": {
    "email_deliverable": true,
    "email_safe": true
  }
}
```

### Phone validation (if `phone_number` was sent)

```json
{
  "phone_id_validation": {
    "carrier": "Comcel",
    "contact_first_name": null,
    "contact_last_name": null,
    "sim_swap_date": null,
    "sim_swap_score": 1
  },
  "phone_number": "573002345678",
  "phone_score_validation": {
    "phone_score": 301
  }
}
```

### IP validation

```json
{
  "ip_validation": {
    "country": "Colombia",
    "region": "Antioquia",
    "city": "Medellín",
    "is_safe": true
  }
}
```

### Government records (Colombia)

**`registry` object (ANI/Registraduría/Migración):**

```json
{
  "registry": {
    "status": "ACTIVA",
    "nuip": "9876543210"
  }
}
```

**Note:** `status` values in the `registry` object are the same as those used in the ANI Compliance documentation. See that documentation for all available status codes: [See ANI Compliance status codes](ani-compliance.md)

**`ubica_response` object:** Includes address, phone, and email history:

```json
{
  "ubica_response": {
    "full_name": "RODRIGUEZ SANTOS MARIA ELENA",
    "document_number": "9876543210",
    "document_status": "VIGENTE",
    "age_range": "35-40",
    "addreses": [
      {
        "addresses": "CR 45 # 78 - 90",
        "city": "MEDELLIN (ANTIOQUIA)",
        "location_type": "RES",
        "first_report": "Mon, 15 Jan 2020 00:00:00 GMT",
        "last_report": "Tue, 30 Sep 2024 00:00:00 GMT"
      }
    ],
    "cellphones": [
      {
        "cellphone_number": "3009876543",
        "is_activate": "SI",
        "first_report": "Wed, 20 Mar 2021 00:00:00 GMT",
        "last_report": "Mon, 15 Oct 2024 00:00:00 GMT"
      }
    ],
    "mails": [
      {
        "email": "MARIA.RODRIGUEZ@EMPRESA.COM",
        "first_report": "Fri, 10 Jun 2022 00:00:00 GMT",
        "last_report": "Wed, 01 Oct 2024 00:00:00 GMT"
      }
    ],
    "phones": [
      {
        "phone_number": "6045678901",
        "prefix": "604",
        "city": "MEDELLIN (ANTIOQUIA)",
        "is_activate": "SI"
      }
    ]
  }
}
```

## Detected document types

The `type_id` field indicates the specific document type detected by the OCR engine:

### National identity documents (national-id)

| Type                              | Description                                          |
| --------------------------------- | ---------------------------------------------------- |
| `TYPE_ID`                         | National identity card or document                   |
| `TYPE_MINORS_ID`                  | Identity document for minors                         |
| `TYPE_ALIEN_ID`                   | Identity document for resident foreigners            |
| `TYPE_RESIDENT_ID`                | Identification document for legal residents          |
| `TYPE_RESIDENCE_PERMIT`           | Residence permit                                     |
| `TYPE_TEMPORARY_RESIDENCE_PERMIT` | Temporary residence permit                           |
| `TYPE_CITIZENSHIP_CERTIFICATE`    | Citizenship certificate                              |
| `TYPE_MULTIPURPOSE_ID`            | Multipurpose document (access to several services)   |
| `TYPE_VOTER_ID`                   | Voting document                                      |
| `TYPE_PROOF_OF_AGE_CARD`          | Card to prove legal age                              |
| `TYPE_GREEN_CARD`                 | Permanent residence (Green Card)                     |

### Driver’s licenses (driving-license)

| Type                           | Description                                             |
| ------------------------------ | ------------------------------------------------------- |
| `TYPE_DRIVER_CARD`             | Professional driver card                                |
| `TYPE_DRIVING_PRIVILEGE_CARD`  | Driving privilege card                                  |
| `TYPE_DL`                      | Standard driver’s license                               |
| `TYPE_DL_PUBLIC_SERVICES_CARD` | License for public transport or special services        |

### Passports (passport)

| Type                     | Description                                       |
| ------------------------ | ------------------------------------------------- |
| `TYPE_PASSPORT_CARD`     | Passport card (reduced format)                    |
| `TYPE_CONSULAR_PASSPORT` | Passport issued by consulates                     |
| `TYPE_MINORS_PASSPORT`   | Special passport for minors                       |
| `TYPE_ALIEN_PASSPORT`    | Passport for stateless foreigners or refugees     |
| `TYPE_PASSPORT`          | Standard travel passport                          |

This value is informational and helps automatically identify which document type was processed.

## Important: Information engine, not decision engine

**⚠️ Become Digital is NOT a decision engine.**

We provide:

* ✅ Detailed verification information
* ✅ Technical validation results (OCR, biometrics, liveness, etc.)
* ✅ Scores and risk flags
* ✅ Data extracted from official records

**Your company must:**

* 🎯 Analyze the results provided
* 🎯 Define your own thresholds and business rules
* 🎯 Make approve/reject decisions based on your risk level
* 🎯 Implement decision logic according to your internal policies

### Commercial team support

Our commercial team can help you:

* 📊 Define result interpretation guidelines
* 🎯 Set thresholds for your industry and risk level
* 📋 Create customized decision flows
* 🔍 Analyze specific cases and patterns

**Contact:** Request assistance from the Become Digital support team.

## Interpretation examples

### ✅ Successful verification

```json
{
  "verification": {
    "verification_status": "completed",
    "face_match": true,
    "liveness": true,
    "alteration": true,
    "template": true,
    "watch_list": false,
    "one_to_many_result": true
  },
  "face_match_score": 95.5
}
```

**Interpretation:**

* ✅ Successful facial match (`face_match: true`)
* ✅ Liveness check passed (`liveness: true`)
* ✅ No alterations detected (`alteration: true`)
* ✅ Not on watch lists (`watch_list: false`)
* ✅ No duplicate found in the account (`one_to_many_result: true`)

### ⚠️ Failed facial match

```json
{
  "verification": {
    "verification_status": "completed",
    "face_match": false,
    "liveness": true,
    "alteration": true
  },
  "face_match_score": 45.2
}
```

**Interpretation:**

* ❌ The person in the selfie does NOT match the document photo (`face_match: false`)
* Although liveness is valid, there is an identity discrepancy

### ❌ Altered document detected

```json
{
  "verification": {
    "verification_status": "completed",
    "face_match": true,
    "alteration": false,
    "template": true
  }
}
```

**Interpretation:**

* ❌ Alterations were detected in the document (`alteration: false`)
* Possible fraud or manipulated document
* Although the facial match succeeds, the document is not reliable

### ⚠️ On watch list

```json
{
  "verification": {
    "verification_status": "completed",
    "face_match": true,
    "liveness": true,
    "watch_list": false
  }
}
```

**Interpretation:**

* ⚠️ The person is on watch lists (`watch_list: false`)
* Requires additional review or escalation per your policies

## Common errors

**400 - Verification error:**

* `"La verificacion tuvo un error"` - Processing failed (OCR, liveness, external services)

**401 - Authentication:**

* `"Missing Authorization Header"` - Missing JWT token
* `"Token has expired"` - Token expired, renew

**404 - Not found:**

* `"No se encontró el usuario con ID <user_id>"` - user\_id does not exist for the company

**HTTP codes:**

* `200` - Verification completed
* `202` - Verification in progress (pending)
* `400` - Processing error
* `401` - Invalid token
* `404` - user\_id not found

## Polling Request (Periodic lookup)

If you do not use a [webhook](webhooks.md), check status periodically:

**Strategy:**

1. Wait 15-30 seconds after creating the verification
2. Call this endpoint
3. If 202 (pending), wait 10-15 seconds and check again
4. Repeat until you receive 200 (completed) or 400 (error)
5. Maximum timeout: 2 minutes

**Example (Python):**

```python
import time, requests

def wait_for_result(user_id, token, max_attempts=12):
    url = f"https://api.svi.becomedigital.net/api/v1/identity/{user_id}"
    headers = {"Authorization": f"Bearer {token}"}
    
    for i in range(max_attempts):
        response = requests.get(url, headers=headers)
        
        if response.status_code == 200:
            return response.json()  # Completed
        elif response.status_code == 202:
            time.sleep(10)  # Wait 10 seconds
        else:
            raise Exception(f"Error: {response.status_code}")
    
    raise Exception("Timeout")
```

## Next steps

* [Re-verification (authentication) →](re-verification.md)
* [Configure webhooks →](webhooks.md)
* [Create new verification →](verification-add.md)
