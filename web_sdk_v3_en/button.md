# Become Button

The `<become-button>` custom element renders a button that opens the verification flow when it is clicked. It runs the same flow as `<become-kyc-sdk>`, onboarding and reverification, with the same contract configuration. To choose between them, see [Choose a product](README.md#choose-a-product).

> **Note**: The registered custom element name is `become-button`, not `become-web-button`.

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

For reverification, set `flow="reverification"` and `user-id`.

## Attributes

The button reads the same verification attributes as `<become-kyc-sdk>` (`contract-id`, `access-token`, `user-id`, `flow`, `locale`, `country`, `doc-type`, `redirect-url`, `webhook-url`, `theme-mode`, and the rest), plus its own:

| Attribute | Description |
|-----------|-------------|
| `mode` | `overlay` (default), `embed`, or `fullscreen`. See [Display modes](#display-modes) |
| `button-text` | Button label text (default: "Iniciar verificación") |
| `loading-text` | Loading text (default: "Cargando...") |
| `hide-icon` | Hide the default shield icon (boolean attribute) |

Values are read when the button is clicked. The full attribute list, the alternative names (`token`, `data-user-id`, `flowId`, `theme`, `language`, and others), and the JavaScript properties are in the [API reference](api.md#become-button).

## Display Modes

The flow always opens on its own layer above your page content, a direct child of `<body>`, wherever the button is placed. The page does not scroll behind it. The mode sets what is around the card:

| Mode | Behavior |
|------|----------|
| `overlay` | Centered card on a dimmed backdrop, with a close control |
| `embed` | The card over the page, without a backdrop |
| `fullscreen` | Covers the whole viewport with the theme background |

To show the flow inside a container of your own layout, use [`<become-kyc-sdk>`](web-sdk.md) in `embed` mode.

## Button Label and Icon

The button renders its own label and icon. Child content placed inside `<become-button>` is replaced when the button renders. Customize it with:

- `button-text` and `loading-text` for the label.
- `hide-icon` to remove the default shield icon.
- The `icon` property for your own SVG markup. It is not reflected as an attribute. Use only markup you control.

```javascript
const button = document.querySelector('become-button');
button.icon = '<svg width="18" height="18" viewBox="0 0 24 24" aria-hidden="true">…</svg>';
```

Colors, logo, and texts inside the flow come from your contract configuration, the same as the web SDK. See [customization.md](customization.md).

## Methods

| Method | Description |
|--------|-------------|
| `setConfig(publicConfig: Partial<PublicConfig>): void` | Update public UI configuration |
| `getConfig(): PublicConfig \| null` | Get the resolved public configuration while the flow is open |
| `click(): void` | Open the flow, as a user click does |

## Events

The button emits its own `become:*` events, different from the `become-kyc:*` events of `<become-kyc-sdk>`. They are dispatched on the button element (`bubbles: true`, `composed: true`). Payloads are in [events.md](events.md#become-button-events).

| Event | When |
|-------|------|
| `become:result` | Flow completed successfully |
| `become:userFinishedSdk` | User finished the flow |
| `become:exitedSdk` | User closed the SDK |
| `become:errorSdk` | Fatal or token errors |

## Important Notes

- The button lazy-loads the SDK when clicked.
- After `become:userFinishedSdk` or `become:exitedSdk` the flow closes and the button can open it again.
- Events are emitted on the button element, not on `document.documentElement`.
- The event names and payloads differ from the KYC Web Component (`become-kyc:*` events).
- Default mode is `overlay` (not `embed` like `<become-kyc-sdk>`).
- There is no `createBecomeButton()` function. Programmatic use is `document.createElement('become-button')`, property setters, and `click()`.
- A startup failure appears when the button is clicked. Where it is shown depends on `mode`. See [Troubleshooting](troubleshooting.md#become-button).

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
