# Re-verification Results

Once the re-verification request (facial matching and optionally liveness) has been submitted, you can look up the final result using the `executionId` provided in the first-stage response.

## Results lookup process

1. **Get executionId:** Save the `executionId` returned by the `/matches` endpoint
2. **Look up result:** Use the `/matches/check` endpoint to get the final result
3. **Process response:** Interpret results according to your confidence thresholds

## Results lookup flow

```
Client → [POST /matches with image] → API
API → [Process validation] → Returns executionId
Client → [POST /matches/check with executionId] → API
API → [Find result] → Returns validation data
```

## POST `/matches/check`

**Required grant type:** `reverification:create`

Request the token with **POST** [`/auth`](auth.md) using `"grant_type": "reverification:create"` before calling this endpoint.

> **Sandbox:** Fixed `executionId` values: `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-1`, `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-2`, `5e24688f-6c7b-48ba-a144-86bb3b9667fc-TEST-3` (same as **POST** `/matches`). `contract_id` can be any value. Scenarios and flow in [Sandbox Environment](../sandbox.md). **Direct API only** — the SDK does not use this test mode.

### Service description

This endpoint looks up the result of a previously performed facial validation. It is a lookup endpoint that finds a specific validation using the `executionId` generated during the facial comparison process.

### Required headers

```http
Content-Type: application/json
Authorization: Bearer <your_jwt_token>
```

### Request parameters

```json
{
  "contract_id": 0,
  "executionId": "fd5075b1-74e4-4b23-ac94-ddd9b36ecc33"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| **contract_id** | number | ✅ | Contract ID used in the re-verification |
| **executionId** | string | ✅ | Unique identifier returned in the `/matches` endpoint response |

### Request examples

**Re-verification results lookup:**

```bash
curl --location 'https://api.svi.becomedigital.net/api/v1/matches/check' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer your_jwt_token_here' \
--data '{
    "contract_id": 1,
    "executionId": "fd5075b1-74e4-4b23-ac94-ddd9b36ecc33"
}'
```

## Security considerations

- 🔒 **Store `executionId` values securely** in your database
- ⏰ **executionId values expire** after a set time
- 🔄 **Validate** that the `executionId` belongs to your contract
- 🔑 **Never expose** `executionId` values in public URLs or logs
- 📝 **Log all lookups** for security audit

## Process flow

1. **Parameter validation** → 400 if `contract_id` or `executionId` are missing
2. **Contract search** → 404 if it does not exist
3. **Permission check** → 400 if the service is not active
4. **Validation search** → 404 if the validation is not found
5. **Identity search** → Associated `user_id` is obtained
6. **Successful response** → 200 with all validation data

### API response

#### ✅ **200 OK - Results available**

```json
{
  "company": "YOUR_COMPANY_ACCOUNT_NAME",
  "confidence": 95.4115853309631348,
  "executionId": "fd5075b1-74e4-4b23-ac94-ddd9b36ecc33",
  "liveness": 99.89696502685547,
  "result": true,
  "timestamp": "2025-01-27 14:18:58",
  "user_id": "usuario_12345_1699123456"
}
```

### Response field descriptions

| Field | Type | Description |
|-------|------|-------------|
| `company` | string | Name of the company responsible for the process |
| `confidence` | number | Confidence level (0-100) from facial comparison |
| `executionId` | string | Identifier linking this response to the initial request |
| `liveness` | number | Value indicating the liveness result (0-100) |
| `result` | boolean | `true` = Successful verification, `false` = Failed verification |
| `timestamp` | string | Date and time the verification was processed |
| `user_id` | string | Identifier of the user for whom validation was performed |

## Differences from the `/matches` endpoint

| Aspect | `/matches` | `/matches/check` |
|--------|------------|------------------|
| **Purpose** | Perform validation | Look up result |
| **Input** | Image + data | executionId + contract_id |
| **Process** | Facial comparison | DB lookup |
| **Result** | New validation | Existing validation data |
| **Content-Type** | multipart/form-data | application/json |
| **Parameters** | user_id, image, contract_id | executionId, contract_id |
| **When to use** | First verification | Later lookup |
| **Includes timestamp** | ❌ No | ✅ Yes |

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
- ✅ The person matches the record (`result: true`)
- ✅ High confidence in the comparison (`confidence: 95.5`)
- ✅ High probability of being a real person (`liveness: 99.8`)

**Recommended action:** Allow access/transaction

### ❌ Failed authentication

```json
{
  "result": false,
  "confidence": 2.4,
  "liveness": 99.8
}
```

**Interpretation:**
- ❌ The person does NOT match the record (`result: false`)
- ❌ Low confidence in the comparison (`confidence: 2.4`)
- ℹ️ Although liveness is valid, the face does not correspond

**Recommended action:** Deny access/transaction

### ⚠️ Attention case

```json
{
  "result": true,
  "confidence": 72.5,
  "liveness": 88.3
}
```

**Interpretation:**
- ⚠️ Match with medium confidence (`confidence: 72.5`)
- ⚠️ Liveness with medium score (`liveness: 88.3`)

**Recommended action:** 
- Apply additional validation
- Request a second attempt
- Escalate to manual review per risk policies

## Decision thresholds

### Conservative configuration (high security)

```javascript
if (result === true && confidence >= 90 && liveness >= 95) {
  // ✅ Approve
} else if (result === true && confidence >= 80 && liveness >= 90) {
  // ⚠️ Additional review
} else {
  // ❌ Reject
}
```

### Moderate configuration (balance)

```javascript
if (result === true && confidence >= 80 && liveness >= 90) {
  // ✅ Approve
} else if (result === true && confidence >= 70 && liveness >= 85) {
  // ⚠️ Additional review
} else {
  // ❌ Reject
}
```

### Permissive configuration (better UX)

```javascript
if (result === true && confidence >= 70 && liveness >= 85) {
  // ✅ Approve
} else if (result === true && confidence >= 60 && liveness >= 75) {
  // ⚠️ Additional review
} else {
  // ❌ Reject
}
```

> **Note:** Adjust thresholds according to your application’s risk level and applicable regulations.

## Use cases

### 1. Check the status of an asynchronous facial validation

```python
# At re-verification time
response = requests.post("/matches", data=formdata)
execution_id = response.json()["executionId"]

# Save execution_id in database...
transaction.verification_execution_id = execution_id
transaction.save()

# Later, when you need the result
result = requests.post("/matches/check", json={
    "contract_id": 1,
    "executionId": execution_id
})

if result.json()["result"]:
    print("Successful validation")
```

### 2. Get results from prior validations

```python
# Retrieve execution_id from logs or database
execution_id = get_execution_id_from_database(transaction_id)

# Look up result
result = requests.post("/matches/check", json={
    "contract_id": 1,
    "executionId": execution_id
})

# Process result
if result.json()["result"]:
    approve_transaction()
else:
    reject_transaction()
```

### 3. Verification process audit

```python
# Look up historical results for audit
execution_id = get_execution_id_from_audit_log(verification_id)

# Look up result
result = requests.post("/matches/check", json={
    "contract_id": contract_id,
    "executionId": execution_id
})

# Save to audit logs
save_audit_log({
    "execution_id": execution_id,
    "result": result.json()["result"],
    "confidence": result.json()["confidence"],
    "timestamp": result.json()["timestamp"]
})
```

### 4. Integration with systems that need to look up results

```python
# Transaction monitoring system
def check_verification_status(execution_id, contract_id):
    try:
        result = requests.post("/matches/check", json={
            "contract_id": contract_id,
            "executionId": execution_id
        }, timeout=30)
        
        return {
            "status": "completed",
            "result": result.json()["result"],
            "confidence": result.json()["confidence"]
        }
    except requests.Timeout:
        return {"status": "timeout"}
    except requests.RequestException:
        return {"status": "error"}
```

## Error handling

### **400 - Bad Request - Validation Errors**

#### 1. Missing required fields

```json
{
  "msg": "Faltan campos requeridos: contract_id, executionId"
}
```

**Cause:** Required fields were not sent.

#### 2. Inactive service

```json
{
  "msg": "El servicio de re-verificación no está activo para este contrato"
}
```

**Cause:** The contract does not have the re-verification service enabled.

### **404 - Not Found - Resource Not Found**

#### 1. Contract not found

```json
{
  "msg": "Contrato no encontrado"
}
```

**Cause:** The `contract_id` does not exist in the system.

#### 2. Validation not found

```json
{
  "msg": "Execution ID no encontrado"
}
```

**Cause:** The `executionId` does not exist or is invalid.

### **500 - Internal Server Error - Server Errors**

#### 1. General server error

```json
{
  "msg": "Error interno del servidor"
}
```

**Cause:** Technical error during processing.

### **401 - Unauthorized**

```json
{
  "msg": "Missing Authorization Header"
}
```

**Cause:** Missing or invalid token.

**Solution:** Include a valid `Authorization: Bearer <token>` header.

## Using executionId in lookups

The `executionId` must be included in the JSON request body:

```json
{
  "contract_id": 1,
  "executionId": "fd5075b1-74e4-4b23-ac94-ddd9b36ecc33"
}
```

All lookups to the `/matches/check` endpoint must include these required fields.

## Asynchronous flow

This endpoint complements `/matches` and enables an asynchronous flow where you can perform validation and then look up the result separately.

### Recommended flow:

1. **Perform validation** → POST `/matches` (with image)
2. **Get executionId** → Save in database
3. **Look up result** → POST `/matches/check` (with executionId)
4. **Process result** → Apply business logic

## Best practices

### 1. Storing execution_id

```python
# ✅ Good practice: Save execution_id related to the transaction
transaction.verification_execution_id = execution_id
transaction.save()
```

### 2. Lookup timeout

```python
# ✅ Implement timeout for lookups
try:
    response = requests.post(url, json=data, timeout=30)
except requests.Timeout:
    # Handle timeout
    log_error("Timeout looking up results")
```

### 3. Audit logs

```python
# ✅ Log all verification lookups
log_verification_check({
    "execution_id": execution_id,
    "result": result["result"],
    "confidence": result["confidence"],
    "timestamp": datetime.now(),
    "user_id": result["user_id"]
})
```

### 4. Result caching

```python
# ✅ Cache results to avoid repeated lookups
cache_key = f"verification:{execution_id}"
cached_result = cache.get(cache_key)

if cached_result:
    return cached_result

result = requests.post("/matches/check", json=data)
cache.set(cache_key, result.json(), timeout=3600)  # 1 hour
```

### 5. Robust error handling

```python
# ✅ Implement retries and error handling
def check_verification_with_retry(execution_id, contract_id, max_retries=3):
    for attempt in range(max_retries):
        try:
            result = requests.post("/matches/check", json={
                "contract_id": contract_id,
                "executionId": execution_id
            }, timeout=30)
            
            if result.status_code == 200:
                return result.json()
            elif result.status_code == 404:
                # Validation does not exist
                return None
            else:
                # Server error, retry
                if attempt < max_retries - 1:
                    time.sleep(2 ** attempt)  # Exponential backoff
                    continue
                else:
                    raise Exception(f"Error after {max_retries} attempts")
                    
        except requests.Timeout:
            if attempt < max_retries - 1:
                time.sleep(2 ** attempt)
                continue
            else:
                raise Exception("Timeout after multiple attempts")
```

## timestamp field

The `timestamp` field indicates when the verification was processed:

```json
{
  "timestamp": "2025-01-27 14:18:58"
}
```

**Format:** `YYYY-MM-DD HH:MM:SS`

**Usefulness:**
- Audit of when the verification was performed
- Validate that the verification is recent
- Detect replay attempts


## Next steps

- [Perform re-verification →](re-verification.md)
- [See full verification →](verification-add.md)
- [Look up verification results →](verification-results.md)
