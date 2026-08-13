# Webhooks - Per-contract Configuration

Guide to configure the webhook URL and additional options (headers and parameters) at the contract level.

When a verification completes, the system sends a `POST` to the configured URL. Optional configuration is stored in the contract’s `webhook_config` (JSONB) column.

Webhooks let you receive automatic notifications when a verification completes, without constant polling.

> **Important:** The webhook is a **notification**. After receiving it, **you must call** [GET /identity](verification-results.md) to get the full verification data.

---

## Summary

| Concept | Description |
|---------|-------------|
| `webhook_url` | Webhook destination URL (existing column on the contract) |
| `webhook_config` | JSON object with optional headers and parameters |
| Trigger event | `verification_completed` (when a verification completes) |

Base prefix: `/api/v1`

Authentication: `Authorization: Bearer <token>`

You can only operate on contracts belonging to the **token’s company** (`company_id` from the JWT).

### Required permissions (grant types)

| Method | Grant types |
|--------|-------------|
| `GET` | `verification:get` |
| `PUT` | `verification:update` |

---

## `webhook_config` structure

```json
{
  "headers": {
    "Authorization": "Bearer mi-token",
    "user": "456789"
  },
  "params": {
    "query": {
      "tenant": "acme",
      "partner_id": "abc123"
    },
    "body": {
      "environment": "production",
      "source": "become"
    }
  }
}
```

| Section | Where it is applied on send |
|---------|----------------------------|
| `headers` | Additional HTTP headers (merged with defaults) |
| `params.query` | Query string on the URL (`?tenant=acme&partner_id=abc123`) |
| `params.body` | Extra fields merged into the JSON body |

If `webhook_config` is `null` or empty, the send uses only the base configuration (no custom headers or parameters).

### Limits and validations

- Maximum **20 key-value pairs** per section (`headers`, `params.query`, `params.body`).
- Keys and values must be **text** (`string`), maximum **256 characters** each.
- `webhook_url` must be `http` or `https`, with no embedded credentials and not pointing to internal hosts (SSRF policy).
- Blocked headers (not sent): `Host`, `Content-Length`, `Transfer-Encoding`, `Connection`, `Keep-Alive`, `Proxy-Authorization`, `TE`, `Trailer`, `Upgrade`.

### Receiver URL requirements

- `POST` method, HTTPS recommended
- Publicly accessible
- Respond with `200 OK` within 30 seconds

---

## Payload sent when verification completes

The webhook is sent **only when the process moves to `completed` status**.

### Without `webhook_config` (or `null`)

```http
POST https://hooks.client.com/become
Content-Type: application/json
User-Agent: BecomeDigital-Webhook/1.0

{
  "event": "verification_completed",
  "contract_id": 123,
  "user_id": "uuid-del-usuario"
}
```

### With example configuration

```json
{
  "headers": { "user": "456789" },
  "params": {
    "query": { "tenant": "acme" },
    "body": { "environment": "production" }
  }
}
```

Result:

```http
POST https://hooks.client.com/become?tenant=acme
Content-Type: application/json
User-Agent: BecomeDigital-Webhook/1.0
user: 456789

{
  "event": "verification_completed",
  "contract_id": 123,
  "user_id": "uuid-del-usuario",
  "environment": "production"
}
```

**Base payload fields:**

| Field | Description |
|-------|-------------|
| `event` | Always `"verification_completed"` |
| `contract_id` | Contract ID |
| `user_id` | Unique user identifier |

> **Note:** Full description of result fields in [GET /identity](verification-results.md).

---

## Flow with webhook

1. Receive webhook → Completion notification
2. Call `GET /identity/<user_id>` → Full data (**required**)
3. Process and store

**If the webhook fails:**

- There are no automatic retries
- Use [polling](verification-results.md#polling-request-periodic-lookup) as a fallback
- Implement webhook monitoring

---

## GET `/contract/<contract_id>/webhook`

Look up the webhook configuration for one of your contracts.

**Grant type:** `verification:get`

### Required headers

```http
Authorization: Bearer <your_jwt_token>
```

### Request example

```bash
curl -s -X GET "https://api.svi.becomedigital.net/api/v1/contract/100/webhook" \
  -H "Authorization: Bearer $TOKEN"
```

### 200 Response

```json
{
  "contract_id": 100,
  "webhook_url": "https://hooks.client.com/become",
  "webhook_config": {
    "headers": { "user": "456789" },
    "params": {
      "query": { "tenant": "acme" },
      "body": { "environment": "production" }
    }
  }
}
```

- If no real URL is configured (`aws` or empty), `webhook_url` comes as `null`.
- If there is no extra config, `webhook_config` comes as `null`.

### Errors

| HTTP | Cause |
|------|-------|
| 401 | Missing or invalid token |
| 403 | Missing grant type or no access to the contract |
| 404 | Contract does not exist or does not belong to the company |

---

## PUT `/contract/<contract_id>/webhook`

Update the webhook URL and/or optional configuration.

**Grant type:** `verification:update`

**Body:** at least one of `webhook_url` or `webhook_config` is required.

### Required headers

```http
Content-Type: application/json
Authorization: Bearer <your_jwt_token>
```

### Request example

```bash
curl -s -X PUT "https://api.svi.becomedigital.net/api/v1/contract/100/webhook" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "webhook_url": "https://hooks.client.com/become",
    "webhook_config": {
      "headers": {
        "Authorization": "Bearer secret-token",
        "user": "456789"
      },
      "params": {
        "query": { "tenant": "acme" },
        "body": { "environment": "production" }
      }
    }
  }'
```

### 200 Response

```json
{
  "message": "Webhook configurado correctamente"
}
```

### Partial update

If you send only part of `webhook_config`, it is **merged** with the existing config:

- Send only `headers` → updates headers, keeps prior `params`.
- Send only `params.query` → updates query, keeps prior headers and `params.body`.

### Clear configuration

```json
{
  "webhook_config": null
}
```

Removes all optional config (headers and params). `webhook_url` is not modified if you do not send it.

### Update URL only

```json
{
  "webhook_url": "https://nueva-url.client.com/hook"
}
```

### Update headers only

```json
{
  "webhook_config": {
    "headers": { "user": "456789" }
  }
}
```

### Errors

| HTTP | Cause |
|------|-------|
| 400 | Missing `webhook_url` and `webhook_config`, URL not allowed, or invalid `webhook_config` |
| 401 | Missing or invalid token |
| 403 | Missing grant type or no access to the contract |
| 404 | Contract does not exist or does not belong to the company |

---

## Best practices

- Configure a webhook instead of constant polling
- Validate webhook origin (token in header, custom parameters, etc.)
- Respond `200 OK` quickly and process asynchronously on your side
- Have polling as a fallback if the webhook fails
- Do not expose secrets in logs; use HTTPS

## Next steps

- [Create identity verification →](verification-add.md)
- [Look up verification results →](verification-results.md)
