# Re-verification (Biometric Authentication)

Once a user has completed the **Onboarding** process (initial identity verification), they can authenticate quickly and securely through **facial matching** and **liveness detection**. This removes the need to present an identity document again on every transaction.

> **Note:** This step can be done with the web SDK, which handles the full selfie capture, facial liveness, and validation process. The SDK only requires an access token (`accessToken`) and `userId`. For implementation details, see the [full SDK documentation](https://becomedigital.gitbook.io/web_sdks/).

## Re-verification process

1. **Get selfie:** Capture a new user selfie
2. **Submit validation:** Use the `/matches` endpoint with the image and user data
3. **Process result:** Interpret the response according to your confidence thresholds

## Re-verification flow

```
Client → [Capture selfie] → [POST /matches with image] → API
API → [Compare with registered selfie] → [Validate liveness] → Returns result
Client → [Process result] → [Apply business logic]
```

## POST `/matches`

**Required grant type:** `reverification:create`

Request the token with **POST** [`/auth`](auth.md) using `"grant_type": "reverification:create"` before calling this endpoint.

> **Sandbox (direct API only):** Only `user_id` is sent (no `image` or `contract_id`). Fixed IDs: `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-1` (successful facial match), `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-2` (no facial match), `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-3` (resource not found). Requires sandbox credentials and JWT token. **Not available from the SDK** — with the SDK, selfie capture and liveness are required and real (production). See [Sandbox Environment](../sandbox.md).

### Service description

This endpoint verifies the identity of a previously registered user by comparing a new selfie with the one stored during onboarding.

#### Two main components:

1. **Facial Matching**
   - Compares the selfie taken in the current transaction with the selfie registered at Onboarding
   - Verifies that the person performing the operation is the same one originally enrolled

2. **Liveness Detection**
   - Performed with the SDK in **passive** mode (no forced movements)
   - Ensures the selfie comes from a real person and not a photograph or pre-recorded video

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
| `user_id` | string | Identifier of the user registered at onboarding |
| `contract_id` | string | Contract ID to use |
| `image` | file | Current user selfie photo |

#### 📱 OPTIONAL fields

| Field | Type | Description |
|-------|------|-------------|
| `liveness` | object | Liveness data provided by the SDK |

### Request examples

**Basic re-verification (facial matching only):**

```bash
curl --location 'https://api.svi.becomedigital.net/api/v1/matches' \
--header 'Authorization: Bearer your_jwt_token_here' \
--form 'user_id="usuario_12345_1699123456"' \
--form 'contract_id="1"' \
--form 'image=@"selfie_actual.jpg"'
```

**Re-verification with liveness (facial matching + liveness):**

```bash
curl --location 'https://api.svi.becomedigital.net/api/v1/matches' \
--header 'Authorization: Bearer your_jwt_token_here' \
--form 'user_id="usuario_12345_1699123456"' \
--form 'contract_id="1"' \
--form 'image=@"selfie_actual.jpg"' \
--form 'liveness="{\"score\": 0.95, \"passed\": true}"'
```

### API responses

#### ✅ **200 OK - Re-verification completed**

```json
{
  "company": "YOUR_COMPANY_ACCOUNT_NAME",
  "confidence": 95.4115853309631348,
  "executionId": "f850eaa8-db4e-468f-a5a3-14cffbf975c0",
  "liveness": 99.89696502685547,
  "result": true,
  "user_id": "usuario_12345_1699123456"
}
```

### Response field descriptions

| Field | Type | Description |
|-------|------|-------------|
| `company` | string | Name of the company responsible for the process or the client integrating the service |
| `confidence` | number | Confidence level from facial comparison (0-100) |
| `executionId` | string | Unique identifier linking this response to the request. Use it to look up results |
| `liveness` | number | Liveness result (0-100). Higher value = higher confidence it is a real person |
| `result` | boolean | `true` = Successful verification, `false` = Failed verification |
| `user_id` | string | Identifier of the user for whom validation was performed |

## Interpreting results

### ✅ Successful authentication

```json
{
  "result": true,
  "confidence": 95.5,
  "liveness": 99.8
}
```

**Interpretation:**
- The person in the selfie matches the record (`result: true`)
- High confidence in the comparison (`confidence: 95.5`)
- High probability of being a real person (`liveness: 99.8`)

### ❌ Failed authentication

```json
{
  "result": false,
  "confidence": 2.4115853309631348,
  "liveness": 99.89696502685547
}
```

**Interpretation:**
- The person does NOT match the record (`result: false`)
- Low confidence in the comparison (`confidence: 2.4`)
- Although liveness is valid, the face does not correspond

## Optional liveness parameter

If the `liveness` parameter is included with SDK data:

- **Both** identity and liveness will be validated
- The result will include the `liveness` field with the score

If the `liveness` parameter is omitted:

- **Only** identity (facial matching) will be validated
- The `liveness` field may be `null` or absent

## Recommended thresholds

### Confidence (Facial match confidence)

| Range | Interpretation | Recommendation |
|-------|----------------|----------------|
| **90-100** | Very high match | ✅ Approve automatically |
| **70-89** | Medium-high match | ⚠️ Review or request a second validation |
| **50-69** | Low match | ❌ Reject or escalate to manual review |
| **0-49** | No match | ❌ Reject |

### Liveness

| Range | Interpretation | Recommendation |
|-------|----------------|----------------|
| **95-100** | High probability of real person | ✅ Approve |
| **85-94** | Medium-high probability | ⚠️ Review |
| **0-84** | Low probability or possible fraud | ❌ Reject |

> **Note:** Exact thresholds can be adjusted based on your application’s security needs.

## Security considerations

- 🔒 **Store selfies and biometric data securely**
- ⏰ **executionId values expire** after a set time
- 🔄 **Validate** that the `user_id` belongs to your contract
- 🔑 **Never expose** selfies in logs or public URLs
- 📝 **Log all re-verifications** for security audit
- 🛡️ **Validate confidence thresholds** according to risk policies

## Using the image in requests

The image must be sent as a file in the `image` field of the multipart/form-data request:

```bash
--form 'image=@"selfie_actual.jpg"'
```

All requests to the `/matches` endpoint must include this required file.

## Asynchronous process

Although this endpoint returns an immediate response, you can also look up the result later using the `executionId`:

**See:** [Re-verification Results →](re-verification-results.md)

### Quality requirements

#### Images (image)
- **Formats:** `.jpg`, `.jpeg`, `.png`
- **Resolution:** Minimum 480x480 pixels (recommended: 720x720 or higher)
- **Max size:** 15 MB
- **Centered face, good lighting, only one person**
- **No obstructions:** No sunglasses, mask, etc.

#### Recommended user_id
- **No special characters:** Only letters, numbers, hyphens (`-`), and underscores (`_`)
- **Max length:** 50 characters
- **Valid examples:** `user-12345-abc`, `550e8400-e29b-41d4`, `usuario_789_1699123456`
- **Invalid examples:** `user@123#invalid!`, `juan.perez@gmail.com`

## Use cases

### 1. Authentication in transactions

Validate the user’s identity before approving a sensitive financial transaction.

### 2. Biometric login

Allow application access via facial recognition instead of a password.

### 3. Identity confirmation

Verify that the person performing an operation is the account holder.

### 4. Periodic re-validation

Request biometric authentication periodically to keep the session active.

### Common errors

**400 - Invalid fields:**
- `"Falta el 'user_id'"` - Missing required field
- `"Falta el 'contract_id'"` - Missing required field
- `"Falta el archivo 'image'"` - Selfie image required
- `"user_id no encontrado"` - No prior onboarding for that user_id

**400 - Files:**
- `"No se detectó rostro en la imagen"` - Image does not contain a visible face
- `"Múltiples rostros detectados"` - More than one person in the image
- `"Imagen de baja calidad"` - Blurry or pixelated image
- `"Formato de imagen no válido"` - Use JPG, JPEG, or PNG

**401 - Authentication:**
- `"Missing Authorization Header"` - Missing JWT token
- `"Token has expired"` - Token expired, renew

**Contract errors:**
- `"No se encontró el contrato"` - Invalid contract_id
- `"Usuario no registrado"` - user_id does not exist in the system

## Best practices

**File quality:**
- Better image quality = Faster processing
- Compress before sending (max 15MB per file)
- Check lighting and sharpness before uploading

**Security:**
- Renew JWT tokens regularly
- Do not expose tokens in logs or URLs
- Transmit only over HTTPS
- Validate confidence thresholds according to security policies

**Performance:**
- Reuse the same JWT token for multiple requests
- Implement adequate timeouts (60-90 seconds)
- Validate data before sending to avoid errors
- Configure appropriate confidence thresholds

**Error handling:**
- Implement automatic retries for 500 errors
- Log error responses for debugging
- Handle low-confidence cases with alternative flows
- Implement failure notifications for the user

## Advantages of re-verification

- ⚡ **Fast:** No need to send documents
- 🔒 **Secure:** Combines facial matching and liveness
- 👍 **Better UX:** Smoother experience for the user
- 💰 **Cost-effective:** Lower cost than a full verification

## Next steps

- [Look up re-verification results →](re-verification-results.md)
- [See initial verification (onboarding) →](verification-add.md)
- [Look up verification results →](verification-results.md)
