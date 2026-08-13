# KYC Verification API - Become Digital

**Welcome to the official documentation for Become Digital’s KYC Verification API.** This API lets you verify your users’ identity digitally and remotely in an easy, fully interoperable way. It is designed so your company can meet international and local regulations and everything related to KYC (Know Your Customer) / AML (Anti-Money Laundering).

## Key features

- ✅ Identity document verification (national IDs, passports, driver’s licenses)
- ✅ Liveness detection and facial matching
- ✅ Document alteration and fraud detection
- ✅ Secure authentication with JWT tokens
- ✅ Asynchronous verification process
- ✅ Integration via SDK or direct API

## Base URLs

- **API:** `https://api.svi.becomedigital.net/api/v1` — same URL for **production** and **sandbox** (testing).
- In sandbox you use **test credentials** and the `-TEST-1` / `-TEST-2` / `-TEST-3` suffix on `user_id` (or `executionId` on `/matches/check`). The `/newIdentity` and `/matches` scenarios are **direct API only**; with the SDK, liveness is required in production. Details in [Sandbox Environment](sandbox.md).

## Verification methods

We offer two ways to run the verification flow:

1. **SDKs** for mobile or web device integration
2. **API** for direct calls to the service

## Verification in 3 simple steps

1. **Authentication:** Secure access to the service with credentials
2. **Submission:** Upload the required documents or information
3. **Results lookup:** Get validation results asynchronously

> **Important note:** Validation is **asynchronous**, which means the response from the results lookup endpoint will change as validation progresses.

## Supported identification types

- **National ID** (cédulas, DNI, INE, etc.)
- **Passport**
- **Driver’s license**

## Use cases

Our API is designed to help companies:

- ✅ Comply with KYC/AML rules and regulations
- ✅ Enable digital identity validation for their users
- ✅ Prevent fraud and identity spoofing
- ✅ Automate customer onboarding
- ✅ Re-authenticate users without requiring documents again

## Getting started

1. [Get access credentials](authentication.md)
2. [Authenticate and obtain a JWT token](endpoints/auth.md)
3. *(Optional for integration)* [Try Sandbox](sandbox.md) with `user_id` and `-TEST-1` / `-TEST-2` / `-TEST-3` suffix
4. [Run your first identity verification](endpoints/verification-add.md)
5. [Look up verification results](endpoints/verification-results.md)
6. *(Optional)* [Configure per-contract webhooks](endpoints/webhooks.md)

## Support

For access credentials or technical assistance, contact the Become Digital support team.

Visit our website: [https://becomedigital.ai/](https://becomedigital.ai/)
