# Releasing `@applaudiq/embed-capacitor`

Published to npm under the public `@applaudiq` scope. Mirrors the React Native SDK's flow.

## One-time
- `npm login` (an `@applaudiq` org member; 2FA enabled).

## Cut a release
```bash
npm run typecheck          # sanity
npm run build              # tsup → dist/ (ESM + CJS + .d.ts)
npm publish --access public
```

Or via release-it (bumps version, tags, builds, publishes, pushes):
```bash
npm run release            # patch
npm run release -- minor   # or major
```

`.release-it.json` runs `typecheck` before and `build` after the version bump, then `npm publish` + a git tag.

## Verify
- `npm view @applaudiq/embed-capacitor version`
- A throwaway `npm install @applaudiq/embed-capacitor @capacitor/app @capacitor/browser` resolves.
