# KYC Web Component

Primary integration method for Become KYC. Embed the verification flow in your application using the `<become-kyc-sdk>` custom element.

## Include the Bundle

```html
<script src="https://websdk.svi.becomedigital.net/sdk/become-kyc-sdk.js"></script>
```

The script is self-contained. No additional installation is required.

## Basic Usage

```html
<become-kyc-sdk
  mode="embed"
  contract-id="contract_123"
  access-token="eyJhbGciOiJIUzI1NiIs..."
  user-id="USER456"
  flow="onboarding"
  locale="es"
  country="CO"
  redirect-url="https://myapp.com/thank-you"
></become-kyc-sdk>
```

## Attributes

### Required

| Attribute | Description |
|-----------|-------------|
| `contract-id` | Contract identifier |
| `access-token` | JWT access token |

### Common optional attributes

| Attribute | Description | Values |
|-----------|-------------|--------|
| `user-id` | User identifier | Required for reverification |
| `flow` | Flow type | `onboarding` (default), `reverification` |
| `locale` | Language locale | `es` (default). See [Texts and locales](customization.md#texts-and-locales) |
| `country` | ISO 3166 country code | Skips country selection step |
| `doc-type` | Document type | `national-id`, `passport`, `driving-license` |
| `redirect-url` | URL after completion | |
| `mode` | Display mode | `embed` (default), `overlay`, `fullscreen` |
| `theme-mode` | Which palette is active | `light` (used when omitted), `dark`, `system`. See [Theme mode](customization.md#theme-mode) |

Every attribute, its alternative names, and the rules for each value are in the [API reference](api.md#become-kyc-sdk).

Set values with attributes; the element has no JavaScript properties for them. Changing `theme-mode` switches the palette in place. Changing any other attribute restarts the verification from the first step.

## Configure

Configure colors, logo, texts, and UI options for your contract in the Dashboard. To override them on one page, call `setConfig()`:

```javascript
const sdk = document.querySelector('become-kyc-sdk');
sdk.setConfig({ ui: { showStatusIcons: true } });
```

All options are described in [customization.md](customization.md). The methods are listed in the [API reference](api.md#methods).

## Handle the Outcome

The element dispatches `become-kyc:*` events. They bubble to `document`.

```javascript
sdk.addEventListener('become-kyc:event', (event) => {
  const { action, payload } = event.detail;
  if (action === 'result') console.log('Verified:', payload.data);
  if (action === 'exitedSdk') console.log('The person left the flow');
});

sdk.addEventListener('become-kyc:error', (event) => {
  console.error('SDK error:', event.detail.code);
});
```

See [events.md](events.md) for every event, payload, and sequence.

## Lifecycle

1. **Load** — Include the SDK bundle.
2. **Mount** — Add the custom element to the DOM. It shows `loading-text` while it loads the contract configuration, then renders the flow and emits `become-kyc:ready`.
3. **Flow** — The SDK renders the resolved flow steps sequentially.
4. **Finish** — On success the SDK emits `result`, then `userFinished` when the person closes the result screen. If `redirect-url` is set, the browser then opens it.
5. **Cleanup** — The component removes itself from the DOM after `userFinished` or `exitedSdk`. Declining the data treatment terms emits `atdpDeclined` followed by `exitedSdk`.

If startup fails, see [Troubleshooting](troubleshooting.md#startup-failures).

## Display Modes

| Mode | Behavior |
|------|----------|
| `embed` | Fills the host container. Scrolls inside that container when the content is taller than the space. |
| `overlay` | Centered modal with backdrop. Shows a close control; closing emits `exitedSdk` |
| `fullscreen` | Covers the whole viewport with the theme background. No close control |

## Layout

The component adapts to the size of the element you place it in. In `embed` mode that size is your container, not the browser window. A tall phone with the component inside a short area stays compact. A smaller window with a tall area gets the more open layout. Nothing has to be configured for this.

### Embed

Set a width and a height on the host element, or on a parent that already has a definite height. A height of `100%` does nothing useful until some ancestor has one.

The component fills that box. It will not lay out narrower than 280px or shorter than 560px. If you give it less, it scrolls inside its own element instead of overflowing your page. Size the host at least that large so the person completing verification is not scrolling inside the component.

The card width and spacing are configuration: see [Card size and density](customization.md#card-size-and-density).

### Overlay and fullscreen

These modes cover the viewport. The flow is painted on its own layer, a direct child of `<body>` above your page content, so where you place the tag, its size, and the styles of its ancestors do not affect it. While the flow is open the page does not scroll behind it, and focus returns to where it was when the flow closes. The card stays inside the viewport and keeps a stable size from step to step.

### What people see

On the opening welcome screen, and on the introductions to document capture and face verification, extra vertical space is used when the container is taller than the copy. The same screens stay compact when the copy already fills the area. The primary action stays available. If the text is longer than the area, the content scrolls and the action remains reachable. Text is not clipped to fit.

The progress indicator appears on the steps the person moves through. The opening welcome screen and the final result screen do not show it. It is also hidden while the camera is in use, and it returns when the camera step is finished.

## Browser Requirements

- Modern browser with Custom Elements and Shadow DOM support
- ES2020+ JavaScript target
- Network access to the SDK bundle and API endpoints

## Minimal Example

```html
<!DOCTYPE html>
<html>
<head>
  <title>KYC Verification</title>
</head>
<body>
  <div id="kyc-root" style="width: 100%; max-width: 480px; height: 640px;"></div>

  <script src="https://websdk.svi.becomedigital.net/sdk/become-kyc-sdk.js"></script>
  <script>
    const container = document.getElementById('kyc-root');
    const sdk = document.createElement('become-kyc-sdk');
    sdk.setAttribute('mode', 'embed');
    sdk.setAttribute('contract-id', 'contract_123');
    sdk.setAttribute('access-token', 'eyJhbGciOiJIUzI1NiIs...');
    sdk.setAttribute('flow', 'onboarding');
    sdk.setAttribute('locale', 'es');
    container.appendChild(sdk);

    document.addEventListener('become-kyc:event', (e) => {
      console.log('SDK event:', e.detail);
    });
  </script>
</body>
</html>
```

## Next Steps

- [Customization](customization.md) — colors, appearance, logo, texts, and UI options.
- [Events](events.md) — payloads and event sequences.
- [API reference](api.md) — every attribute, method, and type.
- [Troubleshooting](troubleshooting.md) — startup failures and common issues.
