# Authentication

To access Become Digital’s KYC Verification API, you must authenticate with a JWT access token. The authentication endpoint validates the client credentials and the requested `grant_type`, and returns a token scoped to the indicated resource. That token must be included in requests to the corresponding endpoint.

## Authentication process

1. **Get credentials:** Request your `client_id` and `client_secret` from the Become Digital support team
2. **Identify the resource:** Check the documentation for the endpoint you will call to learn the required `grant_type`
3. **Generate a token:** Use the `/auth` endpoint, sending credentials and that resource’s specific `grant_type`
4. **Use the token:** Include the JWT in the `Authorization: Bearer <token>` header of requests to that resource

> Each endpoint’s `grant_type` is documented on that resource’s page. Request a separate token for each resource you will consume.

## Authentication flow

```
Client → [Look up endpoint grant_type] → [POST /auth with credentials + grant_type] → API
API → [Validate credentials and grant_type] → Returns JWT
Client → [Use JWT in headers] → Accesses the protected endpoint
```

## Security considerations

- ⚠️ **Never expose** the `client_secret` in client-side code
- 🔒 **Store access tokens securely**
- ⏰ **Tokens expire** by default after **1 hour**
- 🔄 **Implement automatic token renewal** when expiration is detected
- 🔑 **Password resets** must be requested from technical support

## Token expiration time

The JWT token has a default expiration of **1 hour**, but this limit can be changed on request to technical support.

> **Important note:** Once the **token** has **expired**, you must call the `/auth` endpoint again with the same credentials and the **same `grant_type`** for the resource you will consume.

## Available environments

Access credentials (`client_id` and `client_secret`) are provided by the Become Digital team for:

- **Testing environment**
- **Production environment**

Make sure you use the correct credentials for the environment you are working in.

## Using the token in requests

The token must be included in the `Authorization` header on subsequent requests:

```http
Authorization: Bearer <your_jwt_token>
```

All requests to protected endpoints (except `/auth`) must include this header. The token must have been obtained with the `grant_type` indicated in each resource’s documentation.

## Next steps

- [See authentication endpoint →](endpoints/auth.md)
- [Run first verification →](endpoints/verification-add.md)
- [Look up results →](endpoints/verification-results.md)
