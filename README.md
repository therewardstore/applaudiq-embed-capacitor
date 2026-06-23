# @applaudiq/embed-capacitor

[![npm](https://img.shields.io/npm/v/@applaudiq/embed-capacitor.svg)](https://www.npmjs.com/package/@applaudiq/embed-capacitor)

Embed the full **Applaud IQ** recognition portal inside a **Capacitor** app — plain Capacitor or
**Ionic React / Angular / Vue** — with **auto-login** (a one-time token minted by your server) or
**manual login** (the portal's own email / SSO). The portal renders in an `<iframe>`; **SSO** and the
Android **Back** button go through Capacitor's native plugins.

- Works with **plain Capacitor** and **Ionic** (React / Angular / Vue) — the API is framework-agnostic.
- **Auto + manual login**, the **HR-approval gate**, and native **SSO** (Google / Microsoft).
- Peer deps: `@capacitor/core`, `@capacitor/app`, `@capacitor/browser`.

---

## 1. Install

```bash
npm install @applaudiq/embed-capacitor @capacitor/app @capacitor/browser
npx cap sync
```

## 2. Register your SSO callback scheme

SSO opens in the system browser and the backend hands the one-time code back to **your app's** deep link.
Pick a scheme **unique to your app** (not the brand-wide `applaudiq://`) and register it natively:

```xml
<!-- iOS: ios/App/App/Info.plist -->
<key>CFBundleURLTypes</key>
<array><dict>
  <key>CFBundleURLSchemes</key><array><string>myapp</string></array>
</dict></array>
```

```xml
<!-- Android: android/app/src/main/AndroidManifest.xml — inside <activity android:name=".MainActivity"> -->
<intent-filter>
  <action android:name="android.intent.action.VIEW" />
  <category android:name="android.intent.category.DEFAULT" />
  <category android:name="android.intent.category.BROWSABLE" />
  <data android:scheme="myapp" android:host="sso-callback" />
</intent-filter>
```

Pass it as `ssoCallback` (the SDK sends it to the backend as `native_redirect`). Default `applaudiq://sso-callback`.

## 3. Render the embed

**Manual login** — just the publishable key:

```ts
import { ApplaudIQ } from '@applaudiq/embed-capacitor';

ApplaudIQ.init({ key: 'pk_live_…', ssoCallback: 'myapp://sso-callback' })
  .open({ mode: 'manual', render: 'fullscreen' });
```

**Auto-login** — a one-time token your backend minted (`POST /api/v1/embed/sessions`):

```ts
const embed = ApplaudIQ.init({ key: 'pk_live_…', ssoCallback: 'myapp://sso-callback' }).open({
  mode: 'auto',
  token: embedToken,
  render: 'fullscreen',
  onReady: () => {},        // signed in
  onAuthPending: () => {},  // awaiting HR approval
  onError: (m) => {},       // failed (incl. SSO ?error=)
  onClose: () => {},        // dismissed
});
// later: embed.close();
```

## 4. Options

| Option | |
|---|---|
| `config.key` | Publishable key (`pk_live_…`). Required, both modes. |
| `config.baseUrl` | Portal origin. Defaults to the production portal. **HTTPS** in production. |
| `config.ssoCallback` | Your app's `scheme://host` deep link (default `applaudiq://sso-callback`); sent as `native_redirect`. |
| `config.backNavigation` | `true` (default): **Android** hardware Back + **iOS** left-edge swipe step back inside the embed (`onClose` at the root). `false`: platform default, no iOS swipe. See [Back navigation](#back-navigation). |
| `mode` | `'auto'` (needs `token`) or `'manual'` (portal login). |
| `render` | `'fullscreen'` (default) · `'modal'` · `'inline'` (needs `container`). |
| callbacks | `onReady` · `onAuthPending` · `onError(message)` · `onClose` · `onSignOut`. |

## Back navigation

The portal renders in a **cross-origin iframe**, so the SDK can't read its history directly. With
`backNavigation` on (the default), it relays a back request to the portal, which steps back through its own
screens and replies only when it's already at the embed root — then the SDK tears the embed down and fires
`onClose`. Route home in `onClose` (the Angular example navigates back to its Home page there):

- **Android** — the hardware Back button.
- **iOS** — a **left-edge swipe** (iOS has no hardware Back, and WKWebView's own swipe-back can't traverse the
  iframe). The gesture lives in the leftmost ~20px of the embed.

```ts
.open({
  mode: 'manual',
  onClose: () => router.navigate(['/']),   // dismissed at the embed root → go Home
});
```

## How SSO works

`mode: 'manual'` shows the portal's email / SSO login inside the embed. SSO can't run in a WebView, so the SDK
opens the IdP in the **system browser** (`@capacitor/browser`) at
`…/auth/sso/{provider}/employee/authorize?native=1&native_redirect=<ssoCallback>`. The backend redirects the
one-time code to `<ssoCallback>?code=` (or `?error=`), `@capacitor/app`'s `appUrlOpen` catches it, and the SDK
relays it into the embed, which redeems it and reloads — signed in.

## Test integration

A runnable example ships for each framework in
[`applaudiq-sdk-example`](https://github.com/therewardstore/applaudiq-sdk-example) under
`native-integration/capacitor/` (vanilla · react · angular · vue · ionic-react · ionic-angular · ionic-vue).

See [CHANGELOG.md](./CHANGELOG.md). MIT licensed.
