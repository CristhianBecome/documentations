# Onboarding

## Overview

The onboarding flow is designed for new users to complete their identity verification process. This includes document capture, face liveness checks, and identity validation.

### How It Looks on Your Page

<img src="https://gist.githubusercontent.com/Tyg0th/15c5131ef7d2b24b9effa97eb45dedce/raw/07a5e1f3e428bd1d32bfe2940591872e1ae1ec2d/become-button-example.jpg" width="211" alt="Become web button" />

## Integration Steps

Add the following script to your HTML `<head>` (recommended) or before the closing `<body>` tag:

```html
<script src="https://onboarding-v2.svi.becomedigital.net/resources/button.js"></script>
```

## Flow Details

During onboarding, the user will be guided through the following steps:

1. Document capture (ID, passport, etc.)
2. Face liveness verification
3. Identity checks against government databases
4. Data pooling for enhanced verification

## Button Attributes

| Attribute Name | Description |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `userId` | A unique identifier for the user undergoing KYC verification. |
| `contractId` | The contract identifier associated with the KYC process. |
| `token` | A JWT token created using company credentials, used for secure API requests. |
| `country` | A 2-character country code (default: "CO" for Colombia). Example: "FR" for France. |
| `state` | A 2-character state code (only required for users from the US when the country is set to "US"). |
| `docType` | Defines the type(s) of documents that can be used for KYC verification. Can be a single string or comma-separated list. Valid options: `national-id`, `passport`, `driving-license`. |

### Example of docType Usage

Single Document Type:
```html
<become-button
  userId="<YOUR_USER_ID>"
  contractId="<YOUR_CONTRACT_ID>"
  token="<YOUR_JWT_TOKEN>"
  docType="passport"
/>
```

Multiple Document Types:
```html
<become-button
  userId="<YOUR_USER_ID>"
  contractId="<YOUR_CONTRACT_ID>"
  token="<YOUR_JWT_TOKEN>"
  docType="passport,national-id,driving-license"
/>
```

## Events

| Event Name | Description |
| ---------- | ----------- |
| `become:verificationStarted` | Fired when the KYC verification process begins. |
| `become:userFinishedSdk` | Fired when the user completes the KYC verification process successfully. Payload includes `userId` and `endpoint` - you must poll the API or use webhooks to get full results. |
| `become:exitedSdk` | Fired when the user exits the KYC process manually. |

### Example Event Listener

```js
document
  .querySelector("become-button")
  .addEventListener("become:userFinishedSdk", (event) => {
    console.log("User has completed the KYC process:", event.detail);
    // event.detail contains: { userId, endpoint }
    // Poll the API at endpoint or use webhooks to get full verification data
  });
```

## Getting Results

After `userFinishedSdk` is emitted, you must retrieve the verification data:

1. **Polling**: Call `{endpoint}/api/v1/identity/{userId}` to get the full result
2. **Webhooks**: Configure a webhook URL to receive notification when verification is ready

## Result Event Payload

The onboarding data structure when retrieved from the API:

```json
{
  "data": {
    "id": 0,
    "created_at": "Sat, 25 Jul 2020 15:40:00 GMT",
    "company": "Acc_Demo",
    "fullname": "CLIENT_NAME",
    "birth": "Fri, 03 Jul 1998 00:00:00 GMT",
    "document_type": "national-id",
    "document_number": "CLIENT_DOCUMENT_NUMBER",
    "verification": {
      "alteration": true,
      "estimated_age": true,
      "face_match": true,
      "ip_validation": true,
      "liveness": true,
      "liveness_doc": null,
      "one_to_many_result": true,
      "template": true,
      "ubica": null,
      "verification_status": "completed",
      "watch_list": true
    },
    "blacklist": {
      "result": false,
      "score": 0.8766211271286011,
      "id": 556885
    },
    "comply_advantage": {
      "comply_advantage_result": null,
      "comply_advantage_url": null
    }
  },
  "userId": "user_12345",
  "endpoint": "{API_HOST}/api/v1/identity/{userId}"
}
```

For more detailed information on the `data` field structure, refer to the [Postman Documentation](https://documenter.getpostman.com/view/2293906/T1DtdvBk?version=latest#283c98a6-b797-48d7-ad1e-c756efca7477).