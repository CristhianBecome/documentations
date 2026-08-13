# Endpoints

Become Digital’s KYC Verification API provides the following endpoint groups:

## Authentication

- **POST** `/auth` - Obtain a JWT access token

## KYC - Identity Verification

- **POST** `/newIdentity` - Create a new identity verification (`grant_type`: `verification:create`)
- **GET** `/identity/<user_id>` - Look up verification result by user_id (`grant_type`: `verification:get`)
- **POST** `/matches` - Re-verification via facial matching (`grant_type`: `reverification:create`)
- **POST** `/matches/check` - Look up re-verification results (`grant_type`: `reverification:create`)
- **GET** `/contract/<contract_id>/webhook` - Look up webhook configuration (`grant_type`: `verification:get`)
- **PUT** `/contract/<contract_id>/webhook` - Update webhook URL and configuration (`grant_type`: `verification:update`)

## Gov Checks - Government Verifications

### Colombia
- **POST** `/aniCompliance` - Verify national ID against ANI records

## Complementary Services

### Telco
- **POST** `/telcoValidation` - Validate phone number

### Email
- **POST** `/email` - Validate email address

## Base URL

`https://api.svi.becomedigital.net/api/v1`

Production and sandbox share this same base URL and routes (`/auth`, `/newIdentity`, `/identity/…`, `/matches`, `/matches/check`). In sandbox you use **test credentials** and `-TEST-*` suffixes on `user_id` or `executionId`. Sandbox scenarios for `/newIdentity` and `/matches` are **direct REST API only** (not SDK). Details in [Sandbox Environment](../sandbox.md).

## Required headers

All requests (except `/auth`) must include:

```http
Content-Type: application/json
Authorization: Bearer <your_jwt_token>
```

For endpoints that send files (such as `/newIdentity` and `/matches`), use:

```http
Content-Type: multipart/form-data
Authorization: Bearer <your_jwt_token>
```

## Common HTTP response codes

| Code | Description |
|------|-------------|
| **200** | Successful request |
| **201** | Resource created successfully |
| **202** | Request accepted, processing pending |
| **400** | Request error (missing or invalid parameters) |
| **401** | Unauthorized (missing or invalid token) |
| **404** | Resource not found |
| **412** | Precondition failed (data validation) |
| **500** | Internal server error |

## Navigation

### Getting started
1. [JWT Authentication](auth.md)
2. [Sandbox Environment](../sandbox.md) (testing with `-TEST-1` / `-TEST-2` / `-TEST-3`)
3. [Create identity verification](verification-add.md)
4. [Look up results](verification-results.md)

### KYC Verification
- [Create verification](verification-add.md)
- [Look up individual result](verification-results.md)
- [Re-verification (authentication)](re-verification.md)
- [Re-verification results](re-verification-results.md)
- [Webhooks - Per-contract configuration](webhooks.md)

### Additional services
- [ANI Colombia](ani-compliance.md)
- [Telco validation](telco.md)
- [Email validation](email.md)
