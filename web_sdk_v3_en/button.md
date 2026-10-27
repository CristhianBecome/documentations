# Web Button (Legacy)

**Legacy integration.**

The `<become-button>` custom element provides an optional lightweight button that lazy-loads the SDK on click. It opens the SDK in an overlay or embedded container.

> **Note**: The registered custom element name is `become-button`, not `become-web-button`. This is a legacy wrapper. For new integrations, use `<become-kyc-sdk>` directly.

## Include the Bundle

```html
<script src="https://websdk.svi.becomedigital.net/sdk/become-web-button.js"></script>
```

The `<become-button>` bundle is self-contained. It does not require `<become-kyc-sdk>`.

## Basic Usage

```html
<become-button
  contract-id="contract_123"
  access-token="eyJhbGciOiJIUzI1NiIs..."
  flow="onboarding"
  button-text="Start Verification"
></become-button>
```

## Attributes

| Attribute | Description |
|-----------|-------------|
| `contract-id` | Contract identifier |
| `access-token` | JWT access token |
| `user-id` | User identifier |
| `flow` | Flow type: `onboarding` or `reverification` |
| `locale` | Language locale |
| `country` | ISO 3166 country code |
| `state` | State/province code |
| `doc-type` | Document type |
| `national-id-type` | National ID subtype |
| `national-id-type-choices` | Valid subtypes list (comma-separated) |
| `document-number` | Document number |
| `redirect-url` | URL after completion |
| `webhook-url` | Backend callback URL |
| `mode` | `embed` or `overlay` (default: `overlay`) |
| `button-text` | Button label text (default: "Iniciar verificación") |
| `loading-text` | Loading text (default: "Cargando...") |
| `hide-icon` | Hide the default shield icon (boolean attribute) |

### Attribute Aliases (Legacy V2 Support)

The component accepts the following legacy attribute names for backward compatibility with V2 integrations:

| Legacy Attribute | Maps To |
|------------------|---------|
| `contractId` | `contract-id` |
| `accessToken` / `token` / `data-access-token` | `access-token` |
| `userId` / `data-user-id` / `userid` | `user-id` |
| `flowId` | `flow` |
| `theme` | `theme-mode` |
| `language` | `locale` |
| `redirectUrl` / `redirecturl` | `redirect-url` |
| `webhookUrl` / `webhookurl` | `webhook-url` |
| `docType` | `doc-type` |
| `nationalIdType` | `national-id-type` |

When both kebab-case and legacy attributes are present, the kebab-case value takes precedence.

## Properties (JavaScript API)

In addition to HTML attributes, the following JavaScript properties can be set/get on the element:

| Property | Type | Description |
|----------|------|-------------|
| `contractId` | `string \| null` | Contract identifier |
| `accessToken` | `string \| null` | JWT access token |
| `userId` | `string \| null` | User identifier |
| `flow` | `string \| null` | Flow type |
| `mode` | `string` | Display mode (`embed` \| `overlay`), defaults to `overlay` |
| `redirectUrl` | `string \| null` | URL after completion |
| `webhookUrl` | `string \| null` | Backend callback URL |
| `icon` | `string \| null` | Custom SVG icon HTML for the button |

Setting a property updates the corresponding HTML attribute. `icon` is the exception: it is not reflected as an attribute.

## Methods

| Method | Description |
|--------|-------------|
| `setConfig(publicConfig: Partial<PublicConfig>): void` | Update public UI configuration |
| `getConfig(): PublicConfig \| null` | Get the resolved public configuration |

## Custom Button Content

The component supports custom button content via child elements. When child elements are present, they replace the default button label and icon:

```html
<become-button contract-id="contract_123" access-token="eyJhbGciOiJIUzI1NiIs...">
  <span>Custom button content</span>
  <svg>...</svg>
</become-button>
```

**Note**: Custom content is cleared before the button renders its own label. A startup error replaces that label area with the safe message.

## Events

The button emits different events from `<become-kyc-sdk>`. These legacy events are dispatched on the button element (`bubbles: true`, `composed: true`). Payloads are also listed in [events.md](events.md). The button does not emit preview or editor events.

| Event | When |
|-------|------|
| `become:result` | Flow completed successfully |
| `become:userFinishedSdk` | User finished the flow |
| `become:exitedSdk` | User closed the SDK |
| `become:errorSdk` | Fatal or token errors |

### Event Payloads

```javascript
// become:result
{
  flow: 'onboarding' | 'reverification',
  data: unknown
}

// become:userFinishedSdk
{
  flow: 'onboarding' | 'reverification',
  data: unknown
}

// become:exitedSdk
{
  payload: unknown
}

// become:errorSdk
{
  code: string | number,
  message?: string,
  status?: number,
  retryable?: boolean,
  data?: unknown,
  flow?: string
}
```

## Important Notes

- The Web Button lazy-loads the SDK when clicked.
- It creates an overlay or embedded container automatically (fixed position, z-index: 9999).
- Events are emitted on the button element, not on `document.documentElement`.
- The event names and payloads differ from the KYC Web Component (`become-kyc:*` events).
- The component supports custom content via child elements/slots.
- Default mode is `overlay` (not `embed` like `<become-kyc-sdk>`).
- There is no `createBecomeButton()` function. Programmatic use is `document.createElement('become-button')`, property setters, and `click()`.
- A startup failure is shown under the button and emitted as `become:errorSdk` (`code`, `message`, `status`, `retryable`). The user sentence is the SDK catalog sentence (`errors.startup`) and does not include the diagnostic text. Override it with `setConfig({ texts })` before opening the SDK. The SDK also emits `become-kyc:error` for the same failure. Listen to one contract.
- `embed` and `overlay` do not change that startup surface. The SDK is not mounted yet, so the button cannot paint the error inside an opened experience.
- If the button is hidden (`hidden`, or `display: none` / `visibility: hidden` on the button or an ancestor), the inline message cannot be seen. A non-blocking page notice shows the same safe sentence, with retry only when `retryable` is true. Closing or removing the button removes the notice. A visible button does not also show the notice.
- After the SDK has opened, an error is forwarded once as `become:errorSdk`. The button does not paint that error again. User exit is not a startup failure.

## Minimal Example

```html
<!DOCTYPE html>
<html>
<head>
  <title>KYC Verification Button</title>
</head>
<body>
  <script src="https://websdk.svi.becomedigital.net/sdk/become-web-button.js"></script>
  <become-button
    contract-id="contract_123"
    access-token="eyJhbGciOiJIUzI1NiIs..."
    flow="onboarding"
    button-text="Iniciar verificación"
  ></become-button>
  <script>
    const button = document.querySelector('become-button');
    button.addEventListener('become:result', (e) => {
      console.log('Verification completed:', e.detail);
    });
    button.addEventListener('become:errorSdk', (e) => {
      console.error('Verification error:', e.detail);
    });
  </script>
</body>
</html>
```