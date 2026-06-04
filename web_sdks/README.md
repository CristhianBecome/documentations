# Become Onboarding/Reverification Web SDK

This SDK provides a comprehensive solution for user onboarding and reverification processes, including document capture, face liveness, and identity checks. It supports data pooling for enhanced functionality.

This repository combines elements from both [Become-web-button](https://github.com/Becomedigital/Become-web-button) and [Become-reverification-web-button](https://github.com/Becomedigital/Become-reverification-web-button).

## Available SDK Versions

| SDK | Script URL | Description |
| --- | ---------- | ----------- |
| **Onboarding SDK v2** (Recommended) | `https://onboarding-v2.svi.becomedigital.net/resources/button.js` | Enhanced interface with warnings in development and step analytics events. |
| **Reverification SDK** | `https://onboarding.svi.becomedigital.net/resources/reverification-button.js` | Standalone reverification flow using facial biometrics. |

## SDK Events and Parameters

The SDK communicates with the parent window via `postMessage` events.

### Events Emitted

| Event              | Description |
|--------------------|-------------------------------------------------------------------------------------------------------------------------------------------------|
| `result`           | Emitted when the SDK has a result to share. For REVERIFICATION: includes full data. For ONBOARDING: includes `msg`, `userId` and `endpoint`. |
| `userFinishedSdk`  | Emitted when the user completes the SDK flow successfully. For ONBOARDING: payload includes `userId` and `endpoint`. Client must poll API or use webhooks. |
| `exitedSdk`        | Emitted when the user exits the SDK prematurely. Payload is usually null. |
| `initWarning`      | Emitted during initialization if there are warnings (missing country, state, or document type). Payload includes the key of the warning. |
| `stepLifecycleEvent` | Emitted for step lifecycle events (enter/exit). v2 has enhanced support for detailed step analytics. |

### Parameters Accepted

| Parameter     | Type   | Required | Description                                                                  |
| ------------- | ------ | -------- | ---------------------------------------------------------------------------- |
| `accessToken` | string | Yes      | The access token for authentication (can also be `token`).                   |
| `userId`      | string | Yes      | The unique identifier for the user.                                          |
| `contractId`  | string | Yes      | The contract identifier for the session.                                     |
| `country`     | string | No       | Country code (e.g., "US", "CO").                                                   |
| `docType`     | string | No       | Document type (e.g., "passport", "national-id", "driving-license"). Single value or comma-separated list. |
| `state`       | string | No       | State code, only used if `country` is "US".                                  |

### Result Event Payload

**Onboarding SDK v2:**

- The `userFinishedSdk` event returns `{ msg, userId, endpoint }`
- Clients must poll the API at the endpoint or configure webhooks to retrieve full verification data

**Reverification SDK:**

- The `result` event returns full data including `livenessConfidence`, `auditImages`, `referenceImage`, and `data`

For more detailed information on the `data` field structure:

- **ONBOARDING**: [Postman Documentation](https://documenter.getpostman.com/view/2293906/T1DtdvBk?version=latest#283c98a6-b797-48d7-ad1e-c756efca7477)
- **REVERIFICATION**: [Postman Documentation](https://documenter.getpostman.com/view/2293906/T1DtdvBk?version=latest#06c03291-3e2f-4f66-bf6d-a7bf179d17df)