# Web Integration

URL-based integration for WhatsApp, email, or direct links. The user receives a link and completes verification in a full-page flow hosted at `https://websdk.svi.becomedigital.net/`.

## Link Format

```
https://websdk.svi.becomedigital.net/?contractId=contract_123&accessToken=eyJ...&flow=onboarding&locale=es
```

## Parameters

All parameters are passed as URL query parameters.

### Required

| Parameter | Description |
|-----------|-------------|
| `contractId` | Contract identifier |
| `accessToken` | JWT access token |

### Optional

| Parameter | Description | Values |
|-----------|-------------|--------|
| `userId` | User identifier | Optional in onboarding, required in reverification |
| `flow` | Flow type | `onboarding` (default), `reverification` |
| `locale` | Language locale | `es` (default), `en`, `pt` |
| `country` | ISO 3166 country code | Skips country selection |
| `state` | State/province code | Used with country |
| `docType` | Document type | `national-id`, `passport`, `driving-license` |
| `nationalIdType` | National ID subtype | e.g. `CC`, `CE`, `TI` |
| `nationalIdTypeChoices` | Valid subtypes list | Comma-separated |
| `documentNumber` | Document number | |
| `redirectUrl` | URL after completion | |
| `webhookUrl` | Backend callback URL | |
| `mode` | Display mode | `fullscreen` (forced for web integration) |
| `environment` | API environment | `prod` (default), `dev` |
| `themeMode` | Which palette is active | `light` (used when omitted), `dark`, `system` |
| `externaltxid` | External transaction ID | For tracing |

### Theme Mode Rationale

The `themeMode` parameter is an **explicit client decision** because the SDK must respect each client's visual identity rather than impose a color scheme based on assumptions.

Different clients have different branding requirements:

- **Light-only:** The client's brand defines only a light visual theme.
- **Dark-only:** The client's brand defines only a dark visual theme.
- **Light and dark:** The client supports both and may select the theme according to its own user preferences or application settings.
- **Custom palettes:** A palette may contain dark-looking colors while still being the client's intended light theme. The SDK must not infer the theme from the colors themselves.

The SDK cannot reliably determine which theme is appropriate from the palette, the user's operating system, or generic assumptions about dark mode. Automatically imposing a theme could introduce colors that conflict with the client's brand identity.

Therefore, the client **must** explicitly communicate the intended theming mode through the `themeMode` query parameter.

The `system` value is an **opt-in behavior** for clients that want the SDK to follow the user's system preference. It is not the universal default for branding purposes. If a client's brand requires a specific theme regardless of the user's OS setting, the client should pass `light` or `dark` explicitly.

### Parameter Aliases

The following aliases are also accepted for backward compatibility:

| Alias | Canonical Parameter |
|-------|---------------------|
| `contract-id` | `contractId` |
| `access-token` | `accessToken` |
| `user-id` | `userId` |
| `flow-type` | `flow` |
| `theme-mode` | `themeMode` |
| `redirect-url` | `redirectUrl` |
| `webhook-url` | `webhookUrl` |

## Behavior

- The hosted page reads the query parameters and runs the verification flow in fullscreen. The card stays within the browser window.
- The `mode` parameter is **forced to `fullscreen`** regardless of what is passed.
- If startup fails, the page shows a message for the `locale` you passed. A retry control appears only when the failure can be retried. The page does not show the access token. Leaving the flow and a completed verification are not startup failures. Your application is not notified of that failure in the browser.
- When verification succeeds and `redirectUrl` is set, the hosted page navigates the browser to that URL. A startup failure, a fatal error, or the user leaving the flow does not navigate there. If the page is still open when the user finishes, it navigates to `redirectUrl` again.
- If `webhookUrl` is set, it is sent with the verification so your server can be called when the process is complete. That call is an HTTP callback to your backend. It is not a browser event, and `document.addEventListener()` on your page will not receive it.
- The person goes through the same verification flow as in the web component.
- Visual language, colors, and density come from the contract's public configuration, the same options as the web component. They are not URL parameters. See [web-sdk.md](web-sdk.md).

## Example

```
https://websdk.svi.becomedigital.net/?contractId=contract_123&accessToken=eyJhbGciOiJIUzI1NiIs...&flow=onboarding&locale=es&country=CO&redirectUrl=https://myapp.com/thank-you
```

## Notes

- `userId` is optional in `onboarding`. If not provided, the backend generates a UUID.
- `userId` is required in `reverification` for matching.
- `nationalIdType` is a subtype of `national-id` specific to each country (e.g., `CC`, `CE`, `TI`, `PPT` in Colombia).
- `redirectUrl` is the page the person's browser opens after a successful verification. It is not a JavaScript callback.
- `webhookUrl` is your server's HTTP callback. It is not a browser event.
- This page does not emit `become-kyc:ready`, `become-kyc:event`, `become-kyc:error`, `become-kyc:warning`, or `become:*` events to your application. Listening on your own page, or on a page that only frames this URL, does not receive them.
- You can show the hosted URL in an iframe. The surrounding page still does not receive those events. Use `redirectUrl` and `webhookUrl`.
- This is separate from the scripts you add to your own application. People open a link. They do not install a package.

## Relationship to Web Components

| Aspect | Web Integration | Web Components |
|--------|-----------------|----------------|
| Deployment | Hosted full-page app | Script you add to your page |
| How it is used | Open a hosted link | Add `<become-kyc-sdk>` to your page |
| Mode | Always `fullscreen` | `embed`, `overlay`, `fullscreen` |
| Configuration | Query parameters only | Attributes + `setConfig()` |
| Completion | `redirectUrl` in the browser, `webhookUrl` on your server | DOM events on `<become-kyc-sdk>`. See [events.md](events.md) |
| Use Case | WhatsApp, email, direct links | Embedded in your application |

Both options run the same verification flow. Embed it in your application with the web component. Use this hosted page only when you need to send a link, such as from WhatsApp or email.