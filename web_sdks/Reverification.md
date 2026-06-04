# Reverification

## Overview

The reverification flow is designed for existing users to re-verify their identity, typically for security purposes or periodic compliance checks. This process focuses on biometric verification and liveness detection.

### How It Looks on Your Page

<img src="https://gist.githubusercontent.com/Tyg0th/15c5131ef7d2b24b9effa97eb45dedce/raw/07a5e1f3e428bd1d32bfe2940591872e1ae1ec2d/become-button-example.jpg" width="211" alt="Become reverification button" />

## Integration Steps

Add the following script to your HTML `<head>` or before the closing `<body>` tag:

```html
<script src="https://onboarding.svi.becomedigital.net/resources/reverification-button.js"></script>
```

## Flow Details

During reverification, the user will be guided through the following steps:

1. Face liveness verification with challenge-response
2. Biometric matching against stored reference data
3. Liveness confidence scoring
4. Audit trail generation

## Usage in Your Code

```html
<become-reverification-button
  userId="<YOUR_USER_ID>"
  contractId="<YOUR_CONTRACT_ID>"
  token="<YOUR_JWT_TOKEN>"
/>
```

### Dynamic Creation with JavaScript

```js
const button = document.createElement('become-reverification-button');
button.setAttribute('userId', 'user_12345');
button.setAttribute('contractId', 'contract_67890');
button.setAttribute('token', 'your_jwt_token_here');
document.body.appendChild(button);
```

## Button Attributes

| Attribute Name | Description |
| -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `userId` | The unique identifier assigned during the previous validation process. |
| `contractId` | Contract identifier for validation type. |
| `token` | The JWT token generated for API requests, created using company credentials. |

## Events

| Event Name | Description |
| ---------- | ----------- |
| `become:sdkLoaded` | Indicates that the SDK was loaded successfully. |
| `become:verificationStarted` | Notifies that the re-verification process has started. |
| `become:result` | Indicates the successful completion of the re-verification process; full data is returned. |
| `become:userFinishedSdk` | Notifies that the user clicked the continue button to close the SDK. Only emitted when the verification process is successful. |
| `become:exitedSdk` | Indicates that the user exited the process manually using the close button. |

### Example Event Listeners

```js
// Listen for result
document.querySelector('become-reverification-button').addEventListener('become:result', (event) => {
    console.log('Re-verification result:', event.detail);
    // Full data is returned in the event
});

// Handle multiple events
const btn = document.querySelector('become-reverification-button');
btn.addEventListener('become:result', (event) => {
    console.log('Re-verification result:', event.detail);
});
btn.addEventListener('become:userFinishedSdk', (event) => {
    console.log('User finished SDK', event.detail);
});
```

## Result Event Payload

When the `result` event is triggered, the following data is returned:

```json
{
  "userId": "test_1",
  "livenessConfidence": 99.91615295410156,
  "auditImages": [
    {
      "BoundingBox": {
        "Height": 283.93048095703125,
        "Left": 218.52137756347656,
        "Top": 105.67168426513672,
        "Width": 193.6444854736328
      },
      "Bytes": ""
    }
  ],
  "referenceImage": {
    "BoundingBox": {
      "Height": 287.81195068359375,
      "Left": 206.05030822753906,
      "Top": 76.31394958496094,
      "Width": 201.10231018066406
    },
    "Bytes": ""
  },
  "data": {
    "company": "<YOUR_COMPANY_ACCOUNT_NAME>",
    "user_id": "<USER_ID>",
    "result": false,
    "confidence": 0.20414,
    "liveness": 99.91615295410156,
    "executionId": "UUID"
  }
}
```

## Result Object

| Key | Type | Description |
| --- | ---- | ----------- |
| `livenessConfidence` | Number (Decimal) | The percentage of confidence in the user's liveness during the challenge. If the score is >= 75, it indicates the user is likely a real person. |
| `auditImages` | Array | An array containing the images used as base images for liveness verification. |
| `referenceImage` | Object | The reference image used to compare and verify if the user is live. |
| `data` | Object | The result of the comparison between the most recent selfie and the one attached to the userId. |
| `userId` | String | The unique identifier used to initiate the re-verification process. |

## Data Information

`data` contains the response from our service, indicating whether the user is who they claim to be.

| Key | Type | Description |
| --- | ---- | ----------- |
| `company` | String | The company from where the user was onboarded (your company). |
| `user_id` | String | The user ID of the individual (same as provided). |
| `liveness` | Number (Decimal) | The percentage of confidence in the user's liveness during the challenge. |
| `result` | Boolean | Indicates whether the user passed re-verification. `true` means verified; `false` means not verified. |
| `confidence` | Number (Decimal) | The comparison score between the sent selfie and the one attached to the userId. |
| `executionId` | UUID (String) | The unique identifier for this transaction. |

## Notes

- `auditImages.Bytes` and `referenceImage.Bytes` are Base64-encoded images. These images are not saved on our end, so if you need them, please ensure you handle them appropriately.