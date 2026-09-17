---
name: security-compatibility
description: Security posture, framework compatibility, and browser-support matrix for the Syncfusion React Data Grid — supported defenses (XSS, CSP, CSRF, Injection), supported React versions, supported browser versions, and license-key registration. Load when designing the integration with auth/CSP, wiring older browsers, configuring peer compatibility, or registering the Syncfusion license key.
---

# Security & compatibility

The grid ships security-sensitive defaults and a tight browser/react compatibility matrix. Read this before pairing the grid with auth/CSP, before targeting non-current browsers, or before deploying to a locked-down enterprise environment.

## Supported runtime environments

### React

| React version | Syncfusion® React version |
|---|---|
| 19.0 | 29.2.4 and above |
| 18.0 | 29.2.4 and above |
| 17.0 | 29.2.4 and above |

The grid is built on **React 17+**; Node.js **14+** is required for the dev/build toolchain.

### Browsers

| Browser | Minimum version |
|---|---|
| Chrome | 109+ |
| Firefox | 115+ |
| Edge | 121+ |
| Safari | 15.4+ |

### Node / build

- Node 14+ for the development server and tooling (Vite/CRA/etc. — your choice).
- TypeScript-strict typing is supported.

## Security defenses included

The grid ships active defenses for:

- **XSS (Cross-Site Scripting)** — sanitizes cell/input content. `disableHtmlEncode` is opt-in (`false` by default).
- **CSP (Content Security Policy)** — works in standard CSP environments; restrict `script-src` and `style-src` per Syncfusion's hosting guide.
- **CSRF (Cross-Site Request Forgery)** — guard at the `DataManager`/API layer; pass tokens via `headers: [{ 'X-CSRF-Token': … }]`.
- **Injection Attacks (filter / parameter validation)** — built-in operator type-checking in `filterByColumn` and `search` calls.

Reference: [Syncfusion React Security](https://react.syncfusion.com/react-ui/common-features/security/).

## License-key registration

When a Syncfusion trial license warning banner appears, register your license key once in `src/main.tsx`:

```tsx
import { registerLicense } from '@syncfusion/react-base';

registerLicense('YOUR-SYNCFUSION-LICENSE-KEY');
```

For CI/CD secret handling, store the key as an environment variable and pass it to `registerLicense(process.env.NEXT_PUBLIC_SYNCFUSION_KEY)`. **Never commit the literal key.** (Per the project's `guardrails` skill: "Secrets scan on staged files".)

## `enableDevMode`

`<Grid enableDevMode={false} />` suppresses diagnostic console output. **Always set `false` in production.**

```tsx
<Grid enableDevMode={false} ... />
```

## XSS — `disableHtmlEncode`

Default behaviour: HTML is **encoded** for safety. Use `disableHtmlEncode={true}` to render raw HTML, but only when the input is fully trusted:

```tsx
<Column field="feedback" headerText="<strong>Feedback</strong>" disableHtmlEncode />
```

For dynamic graphical content, prefer a `template` returning `React.ReactElement` — JSX is rendered safely by React.

## CSRF — DataManager

When your API requires an anti-CSRF token, register it in `DataManager.headers`:

```ts
const dm = new DataManager({
  url: '/api/orders',
  adaptor: new UrlAdaptor(),
  headers: [{ 'X-CSRF-Token': getCSRFToken() }],
});
```

When using `onDataRequest`, add the token to your `fetch` call directly.

## CSP — recommended baseline

```html
<meta http-equiv="Content-Security-Policy"
  content="
    default-src 'self';
    script-src 'self';
    style-src 'self' 'unsafe-inline';
    img-src 'self' data:;
    font-src 'self' data:;
    connect-src 'self' https://your-api.example.com;
    frame-ancestors 'none';
    base-uri 'self';
    form-action 'self';
">
```

`'unsafe-inline'` is sometimes required for theme CSS variables. Audit your bundler output to confirm whether classes are static.

## High-risk actions (gate before executing)

- Mass deletes through the API — prefer `confirmOnDelete: true` plus server-side confirmation.
- Bulk exports of sensitive data — gates, audit logs, and access controls on the server.
- `clearFilter([])` style calls that wipe filters — guard with an `if (count > 0)` check.

## Common production deployment gaps

- Missing `@syncfusion/react-base` `registerLicense` call → noisy banner in prod.
- Missing `enableDevMode={false}` → dev console messages leak into production.
- Treating `disableHtmlEncode` the same as `template` → XSS exposure if untrusted data lands in the cell.
- Forgetting to scope data API endpoints in `connect-src` → CSP deploys break.
- Header tokens missing in `DataManager.headers` → CSRF 403 at every API call.
- Mixing `GridAllModule` with feature-specific modules (e.g., forgetting `PagerModule` after switching from prototype) → silent pagination drop.

## Constraints & guardrails

- **Browser support floor**: Chrome 109 / Firefox 115 / Edge 121 / Safari 15.4. Below this, the grid assumes missing CSS features and may not render.
- **React support**: 17.x, 18.x, 19.x; mixing with React 16 will trigger peer-dep errors.
- **License key**: required for production. Trial mode prints a banner per page load.
- **`disableHtmlEncode={true}`** + untrusted data = XSS. Treat as a config-protection item: don't change it without a security review.
- **Don't bypass `enableDevMode` protections** — they are guards, not decorations.
- **CSP**: when the page sets strict CSP, validate that the grid's classes are bundled in JS, not injected as runtime styles. Otherwise `style-src 'unsafe-inline'` will be needed.
- **Integration check**: confirm `registerLicense` is registered exactly once at boot (not per grid) — duplicate calls throw.
- **Secret handling**: never inline the Syncfusion license key in committed source. Use secret stores / env vars per the guardrails framework.