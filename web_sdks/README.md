# Become Onboarding/Reverification Web SDK

This SDK provides a comprehensive solution for user onboarding and reverification processes, including document capture, face liveness, and identity checks. It supports data pooling for enhanced functionality.

This repository combines elements from both [Become-web-button](https://github.com/Becomedigital/Become-web-button) and [Become-reverification-web-button](https://github.com/Becomedigital/Become-reverification-web-button), along with data pooling capabilities.

If you prefer, you can compile the Become-web-button yourself, ensuring that the `signupHost` points to `https://identity.svi.becomedigital.net`.

## SDK Events and Parameters

The SDK communicates with the parent window via `postMessage` events.

### Events Emitted

The SDK emits the following events via `sendMessage`. Each event includes an action and an optional payload.

| Event              | Description                                                                 |
|--------------------|-----------------------------------------------------------------------------|
| `result`          | Emitted when the SDK has a result to share, such as after completing a step or the entire process. Payload includes: `data` (result data), `flow` (ONBOARDING or REVERIFICATION), `userId`, `endpoint`. For REVERIFICATION, additional fields: `livenessConfidence`, `auditImages`, `referenceImage`. |
| `userFinishedSdk` | Emitted when the user completes the SDK flow successfully. Payload includes the final data. |
| `exitedSdk`       | Emitted when the user exits the SDK prematurely. Payload is usually null.   |
| `initWarning`     | Emitted during initialization if there are warnings, such as missing country, state, or document type. Payload includes the key of the warning. |
| `flowDetermined`  | Emitted when the flow type (ONBOARDING or REVERIFICATION) is determined. Payload includes the flow type. |
| `identityCheckError` | Emitted if there's an error during identity check. Payload includes the error message. |
| `stepLifecycleEvent` | Emitted for various step lifecycle events, such as entering or exiting a step. Payload includes step and event details. |

### Parameters Accepted

The SDK accepts the following parameters, typically passed via query parameters or props:

| Parameter    | Type   | Required/Optional | Description                                                                 |
|--------------|--------|-------------------|-----------------------------------------------------------------------------|
| `accessToken` | string | Required         | The access token for authentication.                                        |
| `userId`     | string | Required         | The unique identifier for the user.                                          |
| `contractId` | string | Required         | The contract identifier for the session.                                    |
| `country` (ONBOARDING)     | string | Optional          | The country code (e.g., "US").                                              |
| `docType`  (ONBOARDING)    | string | Optional         | The type of document (e.g., "passport", "id").                              |
| `state`  (ONBOARDING)     | string | Optional         | The state code, only used if `country` is "US".                              |

### Result Event Payload

The `result` event sends the following data in its payload, depending on the flow type:

**Common Fields (for both ONBOARDING and REVERIFICATION):**
| Field     | Type   | Description                          |
|-----------|--------|--------------------------------------|
| `data`    | object | The result data from the API or pooling service. |
| `flow`    | string | The flow type: "ONBOARDING" or "REVERIFICATION". |
| `userId`  | string | The unique identifier for the user.  |
| `endpoint`| string | The endpoint used for the request.   |

**Additional Fields for REVERIFICATION:**
| Field              | Type   | Description                          |
|--------------------|--------|--------------------------------------|
| `livenessConfidence`| number | The confidence score from Liveness. |
| `auditImages`      | array  | Audit images from Liveness.      |
| `referenceImage`   | object | Reference image from Liveness.   |

For more detailed information on the `data` field structure:

- **ONBOARDING**: Refer to the [Postman Documentation](https://documenter.getpostman.com/view/2293906/T1DtdvBk?version=latest#283c98a6-b797-48d7-ad1e-c756efca7477).
- **REVERIFICATION**: Refer to the [Postman Documentation](https://documenter.getpostman.com/view/2293906/T1DtdvBk?version=latest#06c03291-3e2f-4f66-bf6d-a7bf179d17df).
