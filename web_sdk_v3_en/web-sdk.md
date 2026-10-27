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

### Optional

| Attribute | Description | Values |
|-----------|-------------|--------|
| `user-id` | User identifier | Required for reverification |
| `flow` | Flow type | `onboarding` (default), `reverification` |
| `locale` | Language locale | `es` (default), `en`, `pt` |
| `country` | ISO 3166 country code | Skips country selection step |
| `state` | State/province code | Used with country |
| `doc-type` | Document type | `national-id`, `passport`, `driving-license` |
| `national-id-type` | National ID subtype | e.g. `CC`, `CE`, `TI` |
| `national-id-type-choices` | Valid subtypes list | Comma-separated |
| `document-number` | Document number | |
| `redirect-url` | URL after completion | |
| `webhook-url` | Backend callback URL | |
| `mode` | Display mode | `embed` (default), `overlay`, `fullscreen` |
| `environment` | API environment | `prod` (default), `dev` |
| `theme-mode` | Which palette is active | `light` (used when omitted), `dark`, `system` |
| `loading-text` | Custom loading text | |

### Theme Mode Rationale

The `theme-mode` parameter is an **explicit client decision** because the SDK must respect each client's visual identity rather than impose a color scheme based on assumptions.

Different clients have different branding requirements:

- **Light-only:** The client's brand defines only a light visual theme.
- **Dark-only:** The client's brand defines only a dark visual theme.
- **Light and dark:** The client supports both and may select the theme according to its own user preferences or application settings.
- **Custom palettes:** A palette may contain dark-looking colors while still being the client's intended light theme. The SDK must not infer the theme from the colors themselves.

The SDK cannot reliably determine which theme is appropriate from the palette, the user's operating system, or generic assumptions about dark mode. Automatically imposing a theme could introduce colors that conflict with the client's brand identity.

Therefore, the client **must** explicitly communicate the intended theming mode through the `theme-mode` attribute (or `themeMode` query parameter for web integration).

The `system` value is an **opt-in behavior** for clients that want the SDK to follow the user's system preference. It is not the universal default for branding purposes. If a client's brand requires a specific theme regardless of the user's OS setting, the client should pass `light` or `dark` explicitly.

### Attribute Aliases (Legacy Support)

The component also accepts the following legacy attribute names for backward compatibility:

| Legacy Attribute | Maps To |
|------------------|---------|
| `contractId` | `contract-id` |
| `accessToken` | `access-token` |
| `userId` | `user-id` |
| `flowType` | `flow` |
| `themeMode` | `theme-mode` |
| `redirectUrl` | `redirect-url` |
| `webhookUrl` | `webhook-url` |
| `phoneNumberEnc` | `phone-number-enc` |
| `externaltxid` | `externaltxid` |
| `docType` | `doc-type` |
| `nationalIdType` | `national-id-type` |
| `nationalIdTypeChoices` | `national-id-type-choices` |
| `documentNumber` | `document-number` |

When both kebab-case and legacy camelCase attributes are present, the kebab-case value takes precedence.

## Properties (JavaScript API)

In addition to HTML attributes, the following JavaScript properties can be set/get on the element:

| Property | Type | Description |
|----------|------|-------------|
| `contractId` | `string \| null` | Contract identifier |
| `accessToken` | `string \| null` | JWT access token |
| `userId` | `string \| null` | User identifier |
| `flow` | `string \| null` | Flow type (`onboarding` \| `reverification`) |
| `mode` | `string` | Display mode (`embed` \| `overlay` \| `fullscreen`), defaults to `embed` |
| `redirectUrl` | `string \| null` | URL after completion |
| `webhookUrl` | `string \| null` | Backend callback URL |

Setting a property updates the corresponding HTML attribute automatically.

## Methods

| Method | Description |
|--------|-------------|
| `setConfig(publicConfig: Partial<PublicConfig>): void` | Update public UI configuration after mount. **Preferred: use the Dashboard to customize your contract**; this method allows programmatic overrides for theming, branding, texts, and UI behavior. |
| `getConfig(): PublicConfig \| null` | Get the resolved public configuration |
| `getConfigContract(): ConfigContractDescriptor` | Get the public configuration schema for editors. Available before mount. |
| `getResolvedSteps(): ResolvedStepId[]` | Get the resolved step list for the active flow. Empty before mount. |

## Public Configuration

Update theming, branding, texts, and UI behavior at runtime:

```javascript
const sdk = document.querySelector('become-kyc-sdk');

// Selects the active palette. This is not a field of setConfig().
sdk.setAttribute('theme-mode', 'dark');

sdk.setConfig({
  theming: {
    colors: {
      light: { primary: '#07a8b8' },
      dark: { primary: '#07a8b8' },
    },
  },
  branding: {
    logo: {
      url: 'https://websdk.svi.becomedigital.net/sdk/logo.svg',
      position: 'top-center',
      size: 'md',
    },
  },
  texts: {
    es: {
      intro: {
        title: 'Custom title',
      },
    },
  },
  ui: {
    hideIntroStep: true,
    presentation: {
      contentAlignment: 'center',
    },
  },
});
```

### Configuration Domains

| Domain | Description | Key Options |
|--------|-------------|-------------|
| `theming` | Colors, fonts, spacing, visual language | `appearance`, `colors.light/dark`, `typography.fontFamily`, `density`, `radius`, `shadows`, `container.size` |
| `components` | Control presentation | `button.size`, `input.labelStyle` (`outside`, `static-top`, `floating`), `input.size`, `stepper.variant`, `errorBox.variant` |
| `branding` | Logo and branding assets | `logo.url`, `logo.position`, `logo.size`, `logo.opacity`, `logo.shadow`, `logo.radius` |
| `texts` | Internationalized text strings | Organized by locale (`es`, `en`, `pt`) and namespace |
| `ui` | Presentation and confirmation toggles | `confirmations`, `hideIntroHeader`, `hideIntroStep`, `presentation` |

**Note**: `theme-mode` (`light`, `dark`, or `system`) selects the active palette. It is not part of `setConfig()`. When the attribute is omitted, or the value is not one of those three, the active palette is `light`. `system` follows the operating system only when you set it. Color overrides belong under `theming.colors.light` and `theming.colors.dark`. A flat `theming.colors.primary` is not the supported shape, and `theming.mode` is ignored.

## Events

The component dispatches `CustomEvent` on `document.documentElement` (bubbles and composed).

| Event | When |
|-------|------|
| `become-kyc:ready` | SDK mounted successfully |
| `become-kyc:event` | Business logic events (see actions below) |
| `become-kyc:error` | Fatal or token errors |
| `become-kyc:warning` | Non-critical initialization warnings |

### `become-kyc:event` Actions

| Action | When Emitted | Payload |
|--------|--------------|---------|
| `stepLifecycleEvent` | Step lifecycle transition | `{ step, userId?, contractId, status, timestamp? }` |
| `stepChange` | Current step changed | `{ stepId }` |
| `result` | Flow completed successfully | `{ flow, userId, contractId, data }` |
| `atdpDeclined` | User declined data treatment terms | `{ contractId, userId, declinedAt }` |
| `exitedSdk` | SDK exited/closed | `{ data: null }` |
| `userFinished` | User finished the flow | `{ flow?, data? }` |

See [events.md](events.md) for complete event reference with payload structures.

## Lifecycle

1. **Load** — Include the SDK bundle.
2. **Mount** — Add the custom element to the DOM. It bootstraps automatically.
3. **Flow** — The SDK renders the resolved flow steps sequentially.
4. **Cleanup** — The component removes itself from the DOM when the flow ends (`result`, `exitedSdk`) or the user declines ATDP (`atdpDeclined`).

## Display Modes

| Mode | Behavior |
|------|----------|
| `embed` | Fills the host container. Scrolls inside that container when the content is taller than the space. |
| `overlay` | Centered modal with backdrop |
| `fullscreen` | Full viewport coverage |

## Layout

The component adapts to the size of the element you place it in. In `embed` mode that size is your container, not the browser window. A tall phone with the component inside a short area stays compact. A smaller window with a tall area gets the more open layout. Nothing has to be configured for this.

### Embed

Set a width and a height on the host element, or on a parent that already has a definite height. A height of `100%` does nothing useful until some ancestor has one.

The component fills that box. It will not lay out narrower than 280px or shorter than 560px. If you give it less, it scrolls inside its own element instead of overflowing your page. Size the host at least that large so the person completing verification is not scrolling inside the component.

`theming.container.size` limits how wide the card grows (`adaptive`, `sm`, `md`, or `lg`). It does not set the height. `theming.density` changes the spacing (`compact`, `comfortable`, or `spacious`).

### Appearance

`theming.appearance` selects a visual language for the whole flow. Omit it, or set `default`, and the SDK looks as it does today.

| Value | What changes |
|-------|----------------|
| `default` | Current radii, shadows, and borders |
| `flat` | Soft corners, no elevation. Separation comes from borders |
| `material` | Medium corners, raised cards and filled buttons, sunken fields |

A preset is the baseline. These fields can still be set on their own, including together with `default`:

| Field | Supported values | What follows it |
|-------|------------------|-----------------|
| `radius.button` | `xs`, `sm`, `md`, `lg`, `xl`, `full` | Buttons |
| `radius.input` | `xs`, `sm`, `md`, `lg`, `xl` | Fields and validation boxes |
| `radius.card` | `xs`, `sm`, `md`, `lg`, `xl` | Step card, option cards, dialogs, and review images |
| `radius.stepper` | `xs`, `sm`, `md`, `lg`, `xl`, `full` | Progress indicator |
| `shadows.card` | `none`, `subtle`, `medium`, `elevated` | Step card, buttons, and on `material` the document cards and country panel |
| `shadows.modal` | `none`, `subtle`, `medium`, `elevated` | Dialogs |
| `borders.width` | 0–4px | Controls and dialogs |
| `borders.style` | `solid`, `dashed`, `dotted` | Controls and dialogs |

Colors, density, font, card size, and text stay independent of the preset. The preset also sets the field label: on the border for `default`, above the field for `flat`, and floating for `material`. `components.input.labelStyle` or `components.input.size` replaces the preset when you set it.

Resolution order, from lowest to highest:

1. Shipped values.
2. The selected preset, for radius, shadow, border, and the field label and size.
3. A field you set in that same configuration.
4. Later `setConfig()` calls replace only the explicit fields they send. Changing `appearance`, including selecting `default`, preserves earlier explicit values from the contract and the host. The new preset supplies only fields that have never been overridden. Updating an open SDK and reopening it use the same precedence. To reset a customized field, send its desired value explicitly.

`appearance: "material"` with `shadows.card: "none"` keeps the material corners and dialog elevation, and removes the card and button shadow. `appearance: "default"` with `radius.card: "xs"` changes the card corner and leaves the rest of the current UI in place.

The base font size stays within 12px–24px. A value outside that range is ignored and the SDK uses 16px. Border width stays within 0–4px.

The logo corner is `branding.logo.radius` (shipped `lg`, 16px) and its elevation is `branding.logo.shadow` (`none` by default). Neither follows `radius.card` or `shadows.card`. The shadow levels are the same names the card uses.

### Overlay and fullscreen

These modes cover the viewport. The card stays inside the viewport and keeps a stable size from step to step. The host element's size is not used.

### What people see

On the opening welcome screen, and on the introductions to document capture and face verification, extra vertical space is used when the container is taller than the copy. The same screens stay compact when the copy already fills the area. The primary action stays available. If the text is longer than the area, the content scrolls and the action remains reachable. Text is not clipped to fit.

The progress indicator appears on the steps the person moves through. The opening welcome screen and the final result screen do not show it. It is also hidden while the camera is in use, and it returns when the camera step is finished.

## Startup errors

`mount()` paints the flow only after startup succeeds. Until then, `<become-kyc-sdk>` owns the visible error. It uses the mode already selected by the `mode` attribute (default `embed`):

| When | What the user sees |
|------|--------------------|
| `embed` startup fails | A message inside the element, in the embedded area. No page-level notice. |
| `overlay` or `fullscreen` startup fails before the SDK opens | The element itself becomes the viewport surface and shows an alert dialog. Close dismisses that surface. The element stays in the page. |
| Startup can be retried (`retryable`) | A retry control runs startup again. |
| The SDK is already mounted | The SDK paints the failure inside the open experience. The element does not add a second message. |
| The element is removed while startup is in flight | No error surface is left behind. |

`become-kyc:error` still carries the diagnostic (`CONFIG_FETCH_FAILED`, `message`, `status`, `retryable`). The sentence in the element is the SDK catalog sentence (`errors.startup`): it does not include tokens, status codes, or response bodies. Override it with `setConfig({ texts })` before the element starts. HTTP 401 and HTTP 403 use the authorization sentence, not a wrong end-user credential. The startup retry and close labels are `Reintentar` and `Cerrar`. Leaving the flow (`exitedSdk`, `userFinished`, `atdpDeclined`) is not a startup error.

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
  <div id="kyc-root" style="width: 100%; max-width: 480px;"></div>

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