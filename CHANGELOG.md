# Changelog

All notable changes to `@applaudiq/embed-capacitor` are documented here. This project follows
[Semantic Versioning](https://semver.org/).

## [Unreleased]

- **iOS back navigation** — `backNavigation` (default `true`) now adds a **left-edge swipe-back** gesture on iOS
  (which has no hardware Back). It relays the same `applaudiq:back` bridge message as the Android hardware button,
  so the portal steps back through its own history or dismisses (`onClose`) at the embed root. Implemented with
  Pointer Events in the leftmost ~20px of the embed; gated to iOS so Android keeps the hardware button.
- **SSO error message decode** — the deep-link error value is normalized for `+` (form-encoded spaces) and a
  trailing `#` fragment, so the portal's "Authentication Failed" card renders the message cleanly.

## [1.1.0]

First release of the Capacitor SDK — parity with the Web, iOS, Android, React Native, and Flutter SDKs.

- **Auto + manual login** rendered in an `<iframe>` to `<baseUrl>/embed`, using the same
  `{ source, type, payload }` postMessage bridge as the web SDK (`ready`/`authenticated` → `init-token`,
  `auth-pending`, `error`, `close`, `signout`, `resize`).
- **Native SSO** via `@capacitor/browser` (`Browser.open`) — Google/Microsoft reject WebView OAuth, so the
  authorize URL (`?native=1&client_id=&login_hint=&native_redirect=<ssoCallback>`) opens in the system browser.
  The one-time code returns on the app's per-app `ssoCallback` deep link, relayed in via `@capacitor/app`'s
  `appUrlOpen`; on `?error=` the SDK fires `onError`.
- **Per-app callback scheme** (`Config.ssoCallback`, default `applaudiq://sso-callback`) sent to the backend as
  `native_redirect` so two Applaud IQ apps never collide on a device.
- **`backNavigation`** (default `true`) — Android hardware Back (via `@capacitor/app`) dismisses the embed.
- **Security:** the portal must be HTTPS (`http` only for localhost-class dev hosts); the bridge is origin-checked;
  the publishable key is non-secret.
- Works with **plain Capacitor** and **Ionic React/Angular/Vue** — the API is framework-agnostic
  (`ApplaudIQ.init(config).open(options)`).
