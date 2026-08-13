# JWT Authentication

Endpoint to obtain the JWT access token required to consume an API resource. You must provide the `grant_type` for the endpoint you will call; the exact value is documented on each resource’s page.

## POST `/auth`

### Required headers

```http
Content-Type: application/json
```

### Request parameters

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `client_id` | string | Yes | Client identifier provided by Become Digital |
| `client_secret` | string | Yes | Client secret provided by Become Digital |
| `grant_type` | string | Yes | Token scope based on the resource to consume. See the target endpoint documentation |

```json
{
  "client_id": "your_client_id",
  "client_secret": "your_client_secret",
  "grant_type": "<resource_grant_type>"
}
```

### Request example

```bash
curl --location 'https://api.svi.becomedigital.net/api/v1/auth' \
--header 'Content-Type: application/json' \
--data '{
    "client_id": "demo_client_123",
    "client_secret": "demo_secret_456_ABCDEFGH",
    "grant_type": "<resource_grant_type>"
}'
```

### API responses

#### ✅ **200 - Successful authentication**
```json
{
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...",
  "exp": 1703123456,
  "iat": 1703119856
}
```

#### ⚠️ **200 - Inactive credentials**
```json
{
  "msg": "credential inactive"
}
```

### Error handling

#### **400 - Bad Request**

**Request is not JSON:**
```json
{ "msg": "Missing JSON in request" }
```

**Missing client_id:**
```json
{ "msg": "Missing client_id parameter" }
```

**Missing client_secret:**
```json
{ "msg": "Missing client_secret parameter" }
```

**Invalid grant_type:**
```json
{ "msg": "Invalid grant_type" }
```

#### **401 - Unauthorized**

**Invalid credentials:**
```json
{ "msg": "Invalid login details" }
```

#### **404 - Not Found**

**Company not found:**
```json
{ "error": "Not Found" }
```

### Validation process

The API follows this validation flow:

1. **JSON format validation** → Error 400 if the request is not valid JSON
2. **Parameter validation** → Error 400 if `client_id`, `client_secret`, or `grant_type` are missing
3. **grant_type validation** → Error 400 if the `grant_type` is not recognized
4. **Client validation** → Error 401 if the client_id does not exist
5. **Active status validation** → 200 response if credentials are inactive
6. **Company validation** → Error 404 if the company does not exist
7. **Password validation** → Error 401 if the client_secret is incorrect
8. **Successful authentication** → 200 response with a JWT token scoped to the requested `grant_type`

### Important information

- The JWT token contains client, company, and authorized scope (`grant_type`) information
- The `exp` and `iat` fields are Unix timestamps for expiration and issuance dates
- The valid `grant_type` for each resource is indicated in that endpoint’s documentation
- Request a separate token for each resource; a token issued for one `grant_type` does not work for another
- Save the `access_token` to include it in requests to the endpoint associated with that `grant_type`
- If the requested `grant_type` does not work or the token does not authorize the operation, notify the support team to enable the operation for the requested user
