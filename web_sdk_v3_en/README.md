# Integrator documentation

Guides for embedding Become KYC: which product to use, how to load it, and which attributes, methods, and events it supports.

## Choose a product

| Product | Use it when | What you load |
|---------|-------------|----------------|
| [Web SDK](web-sdk.md) | You embed verification in your own page | `become-kyc-sdk.js` and `<become-kyc-sdk>` |
| [Legacy button](button.md) | An existing integration already uses the button wrapper | `become-web-button.js` and `<become-button>` |
| [Hosted web](hosted-web.md) | You send a verification link, including from WhatsApp or email | A URL on the hosted site. No script install |

The web SDK is the product for new embeds. The button is a compatibility wrapper. The hosted page is a fullscreen link. It does not provide the custom element's browser events, including when the URL is placed in an iframe.

## Documents

| Document | Description |
|----------|-------------|
| [web-sdk.md](web-sdk.md) | `<become-kyc-sdk>`: attributes, properties, configuration, production methods |
| [button.md](button.md) | `<become-button>`: legacy attributes, configuration, and `become:*` events |
| [hosted-web.md](hosted-web.md) | Hosted URL parameters, redirect, and fullscreen behavior |
| [events.md](events.md) | Event names and payloads for all three products |

## Quick start

Embed:

```html
<script src="https://websdk.svi.becomedigital.net/sdk/become-kyc-sdk.js"></script>

<become-kyc-sdk
  mode="embed"
  contract-id="contract_123"
  access-token="eyJhbGciOiJIUzI1NiIs..."
  flow="onboarding"
  locale="es"
></become-kyc-sdk>
```

Legacy button:

```html
<script src="https://websdk.svi.becomedigital.net/sdk/become-web-button.js"></script>

<become-button
  contract-id="contract_123"
  access-token="eyJhbGciOiJIUzI1NiIs..."
  flow="onboarding"
  button-text="Start Verification"
></become-button>
```

Hosted link:

```
https://websdk.svi.becomedigital.net/?contractId=contract_123&accessToken=eyJhbGciOiJIUzI1NiIs...&flow=onboarding&locale=es
```
