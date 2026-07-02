# climbing-guide-ota

**Public** distribution repo for Climbing Guide over-the-air (OTA / live) web updates.

This repo contains **build artifacts only — no source code**. The private app is
built elsewhere; the resulting web bundle zip is uploaded here as a GitHub
Release asset, and `latest.json` records what's current. GitHub serves the zips
over its CDN, so no bandwidth is spent on the app's own servers.

## How it works

1. In the private `climbing-guide` repo: bump `ui/ota.version`, then run
   `make ota-release`. That builds the prod SPA, zips it, uploads
   `bundle-<version>.zip` to a Release tagged `ota-v<version>` here, and
   regenerates `latest.json`.
2. Commit the updated `latest.json` to `main`.
3. The .NET API (`GET /api/app-update-check`) reads
   `https://raw.githubusercontent.com/asammitte/climbing-guide-ota/main/latest.json`,
   applies native-version compatibility rules, and tells devices what to download.
4. Installed apps download the zip directly from the Release asset and swap the
   WebView to the new bundle.

## `latest.json` format

```json
{
  "updatedAt": "2026-07-02T10:00:00.000Z",
  "bundles": [
    {
      "platform": "android",
      "version": "1.0.1",
      "url": "https://github.com/asammitte/climbing-guide-ota/releases/download/ota-v1.0.1/bundle-1.0.1.zip",
      "checksum": "<sha256 of the zip>",
      "nativeVersion": "1.0",
      "minNativeVersion": "1.0",
      "notes": "Bug fixes",
      "mandatory": false
    },
    {
      "platform": "ios",
      "version": "1.0.1",
      "url": "https://github.com/asammitte/climbing-guide-ota/releases/download/ota-v1.0.1/bundle-1.0.1.zip",
      "checksum": "<sha256 of the zip>",
      "nativeVersion": "1.0",
      "minNativeVersion": "1.0",
      "notes": "Bug fixes",
      "mandatory": false
    }
  ]
}
```

- `version` — web bundle semver (independent of the native app version).
- `checksum` — sha256 of the zip; the client verifies it before applying.
- `minNativeVersion` — the API will **not** offer a bundle to a device whose
  native `versionName` is lower than this. Raise it whenever a bundle depends on
  native changes (new plugin, `capacitor.config.ts` edits) shipped in a newer
  store binary. This is the safety gate that prevents OTA crashes.

## ⚠️ Never OTA native changes

Live updates replace **web assets only** (HTML/CSS/JS/images). New/updated
Capacitor plugins or `capacitor.config.ts` changes require a new store binary —
ship those through Google Play / App Store and bump `minNativeVersion` here.
# climbing-guide-ota
