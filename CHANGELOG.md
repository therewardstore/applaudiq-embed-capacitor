# Changelog

All notable changes to `@applaudiq/embed-capacitor` are documented here. This project follows
[Semantic Versioning](https://semver.org/).

## [Unreleased]

_Nothing yet._

## [1.3.0] — LTS

**Reward-store voucher download.** Adds the `applaudiq:save-file` bridge message (payload
`{ base64, filename, mime }`): the embedded reward store streams gift-card voucher bytes when a blob
download can't reach disk in a WebView. The SDK writes the file and opens the OS share sheet via the
**optional** `@capacitor/filesystem` + `@capacitor/share` plugins — install them (`npx cap sync`) to enable
it; without them it's a silent no-op. Backward-compatible.

## [1.2.0] — LTS

**Reward-store downloads / external links.** The SDK now handles the `applaudiq:open-external` bridge message
(payload `{ url }`) and opens the URL in the device's system browser — used by the embedded portal for file
downloads, payment pages, and OAuth handoffs. Backward-compatible; no changes to the public API surface.

## [1.1.1] — LTS

**Long-Term Support (LTS) release.** Unified 1.1.1 across the Applaud IQ embed SDK family (Web · iOS · Android ·
React Native · Flutter). Maintenance / version-alignment release — **no public API changes** since 1.1.0.

## [1.1.0]

First release of the Capacitor SDK — parity with the Web, iOS, Android, React Native, and Flutter SDKs.

- **Auto + manual login** rendered in an `<iframe>` to `<baseUrl>/embed`, using the same
  `{ source, type, payload }` postMessage bridge as the web SDK (`ready`/`authenticated` → `init-token`,
  `auth-pending`, `error`, `close`, `signout`, `resize`).
- **Native SSO** via `@capacitor/browser` (`Browser.open`) — Google/Microsoft reject WebView OAuth, so the
  authorize URL (`?native=1&client_id=&login_hint=&native_redirect=<ssoCallback>`) opens in the system browser.
  The one-time code returns on the app's per-app `ssoCallback` deep link, relayed in via `@capacitor/app`'s
  `appUrlOpen`; on `?error=` the SDK fires `onError`. The error message is normalized for `+` (form-encoded
  spaces) and a trailing `#` fragment so the portal's "Authentication Failed" card renders cleanly.
- **Per-app callback scheme** (`Config.ssoCallback`, default `applaudiq://sso-callback`) sent to the backend as
  `native_redirect` so two Applaud IQ apps never collide on a device.
- **`backNavigation`** (default `true`) — step back inside the embed: **Android** hardware Back (via
  `@capacitor/app`) **and** an **iOS left-edge swipe-back** gesture (iOS has no hardware Back). Both relay the
  `applaudiq:back` bridge message, so the portal navigates its own history or dismisses (`onClose`) at the embed
  root. The iOS gesture uses Pointer Events in the leftmost ~20px of the embed.
- **Security:** the portal must be HTTPS (`http` only for localhost-class dev hosts); the bridge is origin-checked;
  the publishable key is non-secret.
- Works with **plain Capacitor** and **Ionic React/Angular/Vue** — the API is framework-agnostic
  (`ApplaudIQ.init(config).open(options)`).
