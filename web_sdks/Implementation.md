# Implementation

## Available SDK Versions

| SDK | Script URL | Description |
| --- | ---------- | ----------- |
| **Onboarding SDK v2** (Recommended) | `https://onboarding-v2.svi.becomedigital.net/resources/button.js` | Enhanced interface with warnings in development and step analytics events. |
| **Reverification SDK** | `https://onboarding.svi.becomedigital.net/resources/reverification-button.js` | Standalone reverification flow using facial biometrics. |

## Integration Steps

### Onboarding SDK v2

Add the following script to your HTML `<head>` (recommended) or before the closing `<body>` tag:

```html
<script src="https://onboarding-v2.svi.becomedigital.net/resources/button.js"></script>
```

### Reverification SDK

```html
<script src="https://onboarding.svi.becomedigital.net/resources/reverification-button.js"></script>
```

## Usage in Your Code

### Onboarding

To include the Become KYC button in your website, simply add the following HTML tag to your page:

```html
<become-button
  userId="<YOUR_USER_ID>"
  contractId="<YOUR_CONTRACT_ID>"
  token="<YOUR_JWT_TOKEN>"
/>
```

### Reverification

To include the Become KYC button in your website, simply add the following HTML tag to your page:

```html
<become-reverification-button
  userId="<YOUR_USER_ID>"
  contractId="<YOUR_CONTRACT_ID>"
  token="<YOUR_JWT_TOKEN>"
/>
```

### How It Looks on Your Page

<img src="https://gist.githubusercontent.com/Tyg0th/15c5131ef7d2b24b9effa97eb45dedce/raw/07a5e1f3e428bd1d32bfe2940591872e1ae1ec2d/become-button-example.jpg" width="211" alt="Become web button" />

### Button Attributes

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

### Dynamic Creation with JavaScript

```js
const button = document.createElement("become-button");
button.setAttribute("userId", "user_12345");
button.setAttribute("contractId", "contract_67890");
button.setAttribute("token", "your_jwt_token_here");
document.body.appendChild(button);
```

## Events

### Onboarding SDK

| Event Name | Description |
| ---------- | ----------- |
| `become:verificationStarted` | Fired when the KYC verification process begins. |
| `become:userFinishedSdk` | Fired when the user completes the KYC verification process successfully. Payload includes `userId` and `endpoint` - you must poll the API or use webhooks to get full results. |
| `become:exitedSdk` | Fired when the user exits the KYC process manually. |

### Example Event Listener (Onboarding)

```js
document
  .querySelector("become-button")
  .addEventListener("become:userFinishedSdk", (event) => {
    console.log("User has completed the KYC process:", event.detail);
    // event.detail contains: { userId, endpoint }
    // Poll the API at endpoint or use webhooks to get full verification data
  });
```

### Reverification SDK

| Event Name | Description |
| ---------- | ----------- |
| `become:sdkLoaded` | Indicates that the SDK was loaded successfully. |
| `become:verificationStarted` | Notifies that the re-verification process has started. |
| `become:result` | Indicates the successful completion of the re-verification process; data is returned. |
| `become:userFinishedSdk` | Notifies that the user clicked the continue button to close the SDK. Only emitted when the verification process is successful. |
| `become:exitedSdk` | Indicates that the user exited the process manually using the close button. |

### Example Event Listener (Reverification)

```js
document.querySelector('become-reverification-button').addEventListener('become:result', (event) => {
    console.log('Re-verification result:', event.detail);
    // Full data is returned in the event
});
```