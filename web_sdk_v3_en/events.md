# Events

Browser events for the two embeddable components. Every event below is a `CustomEvent` with `bubbles: true` and `composed: true`.

The hosted page is not in this list. It does not give your application these events. Completion for that product is `redirectUrl` and `webhookUrl`, documented in [hosted-web.md](hosted-web.md).

## Where to listen

| Product | Listen here | Events |
|---------|-------------|--------|
| `<become-kyc-sdk>` | The custom element, or `document` on the same page | `become-kyc:ready`, `become-kyc:event`, `become-kyc:error`, `become-kyc:warning`, and, in preview, `become-kyc:preview-selection` |
| `<become-button>` | The button element | `become:result`, `become:userFinishedSdk`, `become:exitedSdk`, `become:errorSdk` |

```javascript
const sdk = document.querySelector('become-kyc-sdk');

sdk.addEventListener('become-kyc:event', (event) => {
  const { action, payload } = event.detail;
});
```

That listener applies only to a page that includes `<become-kyc-sdk>`. It does not receive anything from the hosted verification URL, including when that URL is shown in an iframe.

`<become-button>` does not emit `become-kyc:preview-selection`. Use the `become:*` events on the button.

## `<become-kyc-sdk>` events

### Event Names

| Name | When |
|------|------|
| `become-kyc:ready` | SDK mounted successfully |
| `become-kyc:event` | Business logic events (see actions below) |
| `become-kyc:error` | Fatal or token errors |
| `become-kyc:warning` | Non-critical initialization warnings |


---

### `become-kyc:ready`

Emitted after the SDK has resolved configuration and mounted.

```typescript
{
  version: string;
  contractId: string;
  flow: 'onboarding' | 'reverification';
}
```

---

### `become-kyc:event`

Emitted for business logic events. The `action` field identifies the event type.

#### Actions

| Action | When Emitted |
|--------|--------------|
| `stepLifecycleEvent` | Step lifecycle transition |
| `stepChange` | Current step changed |
| `result` | Flow completed successfully |
| `atdpDeclined` | User declined data treatment terms |
| `exitedSdk` | SDK exited/closed |
| `userFinished` | User finished the flow |

#### `stepLifecycleEvent` Payload

```typescript
{
  action: 'stepLifecycleEvent';
  payload: {
    step: string;
    userId?: string;
    contractId: string;
    status: 'initiated' | 'completed' | 'retried' | 'error';
    timestamp?: string; // ISO 8601
  };
}
```

**Status values:**
- `initiated` — Step started (component mounted)
- `completed` — Step finished successfully
- `retried` — Step is being retried (user clicked retry)
- `error` — Step failed with non-retryable error

**Steps that emit lifecycle events:**
`intro`, `atdp`, `contactValidation`, `documentType`, `countrySelection`, `capture`, `preview` (retried on hologram retry/retake), `liveness` (retried on liveness error, selfie validation retry, selfie preview retry), `result` (retried on 5xx error, error on non-retryable).

#### `stepChange` Payload

```typescript
{
  action: 'stepChange';
  stepId: string;
}
```

#### `result` Payload

```typescript
{
  action: 'result';
  payload: {
    flow: 'onboarding' | 'reverification';
    userId: string;
    contractId: string;
    data: unknown; // Full backend response
  };
}
```

#### `atdpDeclined` Payload

```typescript
{
  action: 'atdpDeclined';
  payload: {
    contractId: string;
    userId: string;
    declinedAt: string; // ISO 8601
  };
}
```

#### `exitedSdk` Payload

```typescript
{
  action: 'exitedSdk';
  payload: {
    data: null;
  };
}
```

#### `userFinished` Payload

```typescript
{
  action: 'userFinished';
  payload: {
    flow?: 'onboarding' | 'reverification';
    data?: unknown;
  };
}
```

---

### `become-kyc:error`

Emitted for fatal or blocking errors. Dispatched as a separate `CustomEvent` (not inside `become-kyc:event`).

```typescript
{
  code: string | number;
  message?: string;
  status?: number;
  retryable?: boolean;
  data?: unknown;
  flow?: string;
}
```

**Common codes:**
- `CONFIG_FETCH_FAILED` — Startup failed. This includes a private-config HTTP failure and a rejected startup check (missing `contractId`, missing or malformed `accessToken`, or missing `userId` for reverification). `message` is the diagnostic (`HTTP 401`, `Missing contractId`, `Missing accessToken`, `Invalid or malformed accessToken`, `Request timeout`, and so on). `status` is the HTTP status when the failure was an HTTP response. `retryable` is true for timeouts, network failures, HTTP 429, and HTTP 5xx.
- `PUBLIC_CONFIG_FETCH_FAILED` — Public config fetch failed
- `401` — The result request was not authorized (HTTP 401). This does not by itself mean the end user typed a wrong credential. `data` is the response body when the transport provided one.
- `RESULT_FAILED` — Non-retryable flow failure (user clicked finish on terminal error)

The end-user sentence is not this payload. `<become-kyc-sdk>` shows the catalog sentence (`errors.startup`). HTTP 401 and HTTP 403 use the authorization sentence. The status number and response body stay on this event. Override the sentence with `setConfig({ texts })` before the element starts.

---

### `become-kyc:warning`

Emitted for non-critical initialization warnings.

```typescript
{
  warnings: string[];
}
```

---

### `become-kyc:preview-selection`

Emitted only while preview is on, when the user selects a semantic element. It is not a production integration event and `<become-button>` does not translate it.

```typescript
{
  semanticId: string;
  configPath: string;
  configPaths: string[];
  editableProperties: string[];
  currentValue: unknown;
  stepId: string;
  substate: string | null;
  errorScenario: string | null;
  isShared: boolean;
  bounds?: DOMRect;
  locale: string;
  kinds: string[];
}
```

---

## Web Button (`<become-button>`) Events

These are **legacy events** emitted by the `<become-button>` element itself. They have different names and payloads from the KYC Web Component events.

| Event | When |
|-------|------|
| `become:result` | Flow completed successfully |
| `become:userFinishedSdk` | User finished the flow |
| `become:exitedSdk` | User closed the SDK |
| `become:errorSdk` | Fatal or token errors |

### `become:result` Payload

```typescript
{
  flow: 'onboarding' | 'reverification';
  data: unknown;
}
```

### `become:userFinishedSdk` Payload

```typescript
{
  flow: 'onboarding' | 'reverification';
  data: unknown;
}
```

### `become:exitedSdk` Payload

```typescript
{
  payload: unknown;
}
```

### `become:errorSdk` Payload

```typescript
{
  code: string | number;
  message?: string;
  status?: number;
  retryable?: boolean;
  data?: unknown;
  flow?: string;
}
```

Startup failures include `code: 'CONFIG_FETCH_FAILED'`, the diagnostic `message`, `status` when the failure was HTTP, and `retryable`. The same failure also emits `become-kyc:error` from the SDK. Listen to one of those contracts. The button does not paint a second message for an error that happens after the SDK has opened.

---

## Listening to Events

### `<become-kyc-sdk>`

Use these listeners on the page that contains the custom element. They do not apply to the hosted verification URL.

```javascript
// All business logic events
document.addEventListener('become-kyc:event', (event) => {
  const { action, payload } = event.detail;
  
  switch (action) {
    case 'stepLifecycleEvent':
      console.log('Step:', payload.step, 'Status:', payload.status);
      break;
    case 'result':
      console.log('Flow completed:', payload.flow, payload.data);
      break;
    case 'atdpDeclined':
      console.log('User declined ATDP');
      break;
    case 'exitedSdk':
      console.log('User exited SDK');
      break;
    case 'userFinished':
      console.log('User finished:', payload.flow);
      break;
    case 'stepChange':
      console.log('Step changed to:', payload.stepId);
      break;
  }
});

// Errors
document.addEventListener('become-kyc:error', (event) => {
  console.error('SDK error:', event.detail);
});

// Ready
document.addEventListener('become-kyc:ready', (event) => {
  console.log('SDK ready:', event.detail);
});

// Warnings
document.addEventListener('become-kyc:warning', (event) => {
  console.warn('SDK warnings:', event.detail.warnings);
});
```

### Web Button

```javascript
const button = document.querySelector('become-button');

button.addEventListener('become:result', (event) => {
  console.log('Verification completed:', event.detail);
});

button.addEventListener('become:errorSdk', (event) => {
  console.error('Verification error:', event.detail);
});

button.addEventListener('become:userFinishedSdk', (event) => {
  console.log('User finished:', event.detail);
});

button.addEventListener('become:exitedSdk', (event) => {
  console.log('User exited:', event.detail);
});
```

---

## Event Flow Summary

These sequences are for `<become-kyc-sdk>`. The button uses the `become:*` names in the table above. The hosted page does not expose this sequence to your application.

### Successful Onboarding Flow

1. `become-kyc:ready` — SDK mounted
2. `become-kyc:event` (stepLifecycleEvent: `intro`, `initiated`)
3. `become-kyc:event` (stepLifecycleEvent: `intro`, `completed`)
4. `become-kyc:event` (stepLifecycleEvent: `atdp`, `initiated`)
5. `become-kyc:event` (stepLifecycleEvent: `atdp`, `completed`)
6. `become-kyc:event` (stepLifecycleEvent: `contactValidation`, `initiated`)
7. `become-kyc:event` (stepLifecycleEvent: `contactValidation`, `completed`)
8. `become-kyc:event` (stepLifecycleEvent: `documentType`, `initiated`)
9. `become-kyc:event` (stepLifecycleEvent: `documentType`, `completed`)
10. `become-kyc:event` (stepLifecycleEvent: `countrySelection`, `initiated`)
11. `become-kyc:event` (stepLifecycleEvent: `countrySelection`, `completed`)
12. `become-kyc:event` (stepLifecycleEvent: `capture`, `initiated`)
13. `become-kyc:event` (stepLifecycleEvent: `capture`, `completed`)
14. `become-kyc:event` (stepLifecycleEvent: `liveness`, `initiated`)
15. `become-kyc:event` (stepLifecycleEvent: `liveness`, `completed`)
16. `become-kyc:event` (stepLifecycleEvent: `result`, `initiated`)
17. `become-kyc:event` (stepLifecycleEvent: `result`, `completed`)
18. `become-kyc:event` (action: `result`, payload with flow data)
19. `become-kyc:event` (action: `userFinished`)

### User Exits Early

1. `become-kyc:event` (action: `exitedSdk`)
2. Component removes itself from DOM

### User Declines ATDP

1. `become-kyc:event` (action: `atdpDeclined`)
2. Component removes itself from DOM

### Token Expired / Fatal Error

1. `become-kyc:error` (code: `401` or `RESULT_FAILED`)

---

## Distinguishing Success, Cancellation, and Failure

| Outcome | Events |
|---------|--------|
| **Success** | `become-kyc:event` with `action: 'result'` → `action: 'userFinished'` |
| **User cancelled** | `become-kyc:event` with `action: 'exitedSdk'` |
| **ATDP declined** | `become-kyc:event` with `action: 'atdpDeclined'` |
| **Fatal error** | `become-kyc:error` (check `code` and `retryable`) |

For Web Button, check `become:result` (success), `become:userFinishedSdk`/`become:exitedSdk` (cancelled), `become:errorSdk` (error).