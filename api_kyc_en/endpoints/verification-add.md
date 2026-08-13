# Identity Verification - Create

Main endpoint to create and process identity verifications. This service validates identity documents and performs facial comparison.

> **Note:** This step can be done with the web SDK, which handles the full document capture, facial liveness, and validation process. The SDK only requires an access token (`accessToken`) and `userId`. For implementation details, see the [full SDK documentation](https://becomedigital.gitbook.io/web_sdks/).

## POST `/newIdentity`

**Required grant type:** `verification:create`

Request the token with **POST** [`/auth`](auth.md) using `"grant_type": "verification:create"` before calling this endpoint.

> **Sandbox (direct API only):** In the test environment, files are not processed and the result depends on the `-TEST-1`, `-TEST-2`, or `-TEST-3` suffix on `user_id`. Requires sandbox credentials and JWT token. **Not available from the SDK** — with the SDK, liveness is required and real (production). See [Sandbox Environment](../sandbox.md).

### Required headers

```http
Content-Type: multipart/form-data
Authorization: Bearer <your_jwt_token>
```

### Request parameters

The request must be sent as `multipart/form-data` and include text fields and files.

#### ✅ REQUIRED fields

| Field | Type | Description |
|-------|------|-------------|
| `user_id` | string | Unique user identifier (no special characters or spaces). Maximum 50 characters |
| `contract_id` | string | Active company contract ID |
| `file_type` | string | Document type: `national-id`, `passport`, `driving-license`, `proof-of-residency` |
| `country` | string | ISO-2 country code (e.g.: CO, MX, US, AR) |

#### 📍 CONDITIONAL fields

| Field | Type | Required if | Description |
|-------|------|-------------|-------------|
| `state` | string | `country = "US"` | US state code (e.g.: NY, CA) |

**⚠️ Important restriction:**
- If `country = "US"`, the `state` field **IS REQUIRED**
- If `country != "US"`, **do not send** the `state` field

#### 📱 OPTIONAL fields

| Field | Type | Description |
|-------|------|-------------|
| `city` | string | User city |
| `street` | string | Address/street |
| `zip` | string | Postal code |
| `phone_number` | string | Phone number (E.164 format recommended: +573001234567) |
| `email_address` | string | User email address |
| `nit` | string | NIT (for companies in Colombia) |
| `nationalIdType` | string | Specific national ID type (e.g.: CC, TI, CE) |
| `specific_document_type` | string | Specific document subtype |
| `data_policy_consent` | string | Data policy consent (S/N) |

#### 📁 REQUIRED files

**For `national-id` or `driving-license`:**

| Field | Description | Format | Required |
|-------|-------------|--------|----------|
| `document1` | Front image of the document | `.jpg`, `.jpeg`, `.png` | ✅ Yes |
| `document2` | Back image of the document | `.jpg`, `.jpeg`, `.png` | ✅ Yes |
| `video` | Face video/photo (selfie) | `.mp4`, `.mov`, `.wmv`, `.jpg`, `.png` | ⚠️ Per contract |

**For `passport`:**

| Field | Description | Format | Required |
|-------|-------------|--------|----------|
| `document1` | Passport image | `.jpg`, `.jpeg`, `.png` | ✅ Yes |
| `video` | Face video/photo (selfie) | `.mp4`, `.mov`, `.wmv`, `.jpg`, `.png` | ⚠️ Per contract |

**⚠️ IMPORTANT:**
- For passports, **do not send `document2`**
- The `video` field may be an image or video depending on contract configuration

### Quality requirements

#### Images (document1, document2)
- **Formats:** `.jpg`, `.jpeg`, `.png`
- **Resolution:** Minimum 800x600 pixels
- **Max size:** 5 MB
- **Full document visible, good lighting, sharp image**

#### Videos/Selfie (video)
- **Formats:** `.mp4`, `.mov`, `.wmv` (video) or `.jpg`, `.png` (image)
- **Max size:** 10 MB
- **Duration:** 3-7 seconds (if video)
- **Centered face, good lighting, only one person**

#### Recommended user_id
- **No special characters:** Only letters, numbers, hyphens (`-`), and underscores (`_`)
- **Max length:** 50 characters
- **Valid examples:** `user-12345-abc`, `550e8400-e29b-41d4`, `usuario_789_1699123456`
- **Invalid examples:** `user@123#invalid!`, `juan.perez@gmail.com`

### Request examples

**Colombian national ID verification:**

```bash
curl --location 'https://api.svi.becomedigital.net/api/v1/newIdentity' \
--header 'Authorization: Bearer your_jwt_token_here' \
--form 'user_id="col-user-789xyz"' \
--form 'contract_id="23"' \
--form 'file_type="national-id"' \
--form 'country="CO"' \
--form 'city="Bogotá"' \
--form 'phone_number="+573001234567"' \
--form 'email_address="juan.perez@example.com"' \
--form 'nationalIdType="CC"' \
--form 'data_policy_consent="S"' \
--form 'document1=@"cedula_frente.jpg"' \
--form 'document2=@"cedula_atras.jpg"' \
--form 'video=@"selfie_video.mp4"'
```

**USA driver’s license verification:**

```bash
curl --location 'https://api.svi.becomedigital.net/api/v1/newIdentity' \
--header 'Authorization: Bearer your_jwt_token_here' \
--form 'user_id="usa-driver-456abc"' \
--form 'contract_id="5"' \
--form 'file_type="driving-license"' \
--form 'country="US"' \
--form 'state="NY"' \
--form 'city="New York"' \
--form 'phone_number="+12125551234"' \
--form 'document1=@"license_front.jpg"' \
--form 'document2=@"license_back.jpg"' \
--form 'video=@"selfie.jpg"'
```

**Passport verification:**

```bash
curl --location 'https://api.svi.becomedigital.net/api/v1/newIdentity' \
--header 'Authorization: Bearer your_jwt_token_here' \
--form 'user_id="mex-passport-321def"' \
--form 'contract_id="26"' \
--form 'file_type="passport"' \
--form 'country="MX"' \
--form 'phone_number="+525512345678"' \
--form 'document1=@"passport.jpg"' \
--form 'video=@"selfie_video.mp4"'
```

**⚠️ Note:** For passports, do NOT send `document2`

### API responses

#### ✅ **201 - Resource created successfully**

```json
{
  "code": 201,
  "message": "El recurso fue creado",
  "url_resource": "https://api.svi.becomedigital.net/api/v1/identity/usuario_12345_1699123456",
  "user_id": "usuario_12345_1699123456"
}
```

#### ℹ️ **200 - Record already exists**

```json
{
  "code": 200,
  "message": "Ya existe un registro con el mismo ID para esta compañia",
  "url_resource": "https://api.svi.becomedigital.net/api/v1/identity/usuario_12345_1699123456"
}
```

**Explanation:** If a verification with the same `user_id` already exists for the company, the existing resource is returned without creating a new one

### Common errors

**400 - Invalid fields:**
- `"Falta el 'user_id'"` - Missing required field
- `"El user_id no debe contener caracteres especiales"` - Use only letters, numbers, `-`, and `_`
- `"Debe enviar el estado si selecciona US"` - Missing `state` field for country=US
- `"El file_type no es válido"` - Use: national-id, passport, driving-license, proof-of-residency

**400 - Files:**
- `"Hubo un problema con el envío de los archivos"` - Document not detected or incorrect format
- `documentError: 1` - Front document not detected
- `documentError: 2` - Back document not detected

**401 - Authentication:**
- `"Missing Authorization Header"` - Missing JWT token
- `"Token has expired"` - Token expired, renew

**Contract errors:**
- `"No se encontró el contrato"` - Invalid contract_id
- `"Verifica tu cupo"` - Verification limit reached

## Asynchronous flow

> **Important:** The verification process is **asynchronous**. A 201 response means the verification was created successfully and is being validated. Final results are available once all validations complete.

### Obtaining results

Once the verification is created, you have two options to get results:

**Option 1: Manual lookup (Polling)**
- Periodically check status using [GET /identity/\<user_id\>](verification-results.md)
- See processing times and full documentation at: [Look up Results →](verification-results.md)

**Option 2: Webhook (Recommended)**
- Configure a webhook to receive automatic notifications when verification completes
- See detailed configuration at: [Webhooks - Per-contract configuration →](webhooks.md)

## Best practices

**File quality:**
- Better image quality = Faster processing
- Compress before sending (max 5MB per file)
- Check lighting and sharpness before uploading

**Security:**
- Renew JWT tokens regularly
- Do not expose tokens in logs or URLs
- Transmit only over HTTPS
- Validate webhook origin

**Performance:**
- Configure a webhook instead of constant polling
- Reuse the same JWT token for multiple requests
- Implement adequate timeouts (60-90 seconds)
- Validate data before sending to avoid errors

**Error handling:**
- Implement automatic retries for 500 errors
- Log error responses for debugging
- Have a fallback process if the webhook fails

## Next steps

- [Look up verification results →](verification-results.md)
- [Configure webhooks →](webhooks.md)
- [Re-verification (authentication) →](re-verification.md)
