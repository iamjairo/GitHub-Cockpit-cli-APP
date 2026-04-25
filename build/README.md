# Build Resources

This directory contains static assets used by `electron-builder` when packaging the app.

## Required: App Icon

Before packaging, place the appropriate icon file(s) here:

| File | Platform | Notes |
|------|----------|-------|
| `icon.icns` | macOS | Required for `.dmg` / `.app` packaging |
| `icon.png` | Linux | 1024×1024 PNG — required for AppImage and `.deb` packaging |

> **Tip:** Start with a single **1024×1024 `icon.png`**. electron-builder can auto-convert it to `.icns` for macOS when `icon.icns` is absent, but supplying both is recommended for best results.

### Generating `icon.icns` from a PNG (macOS only)

```bash
# Using macOS built-in iconutil
mkdir icon.iconset
sips -z 16 16     icon.png --out icon.iconset/icon_16x16.png
sips -z 32 32     icon.png --out icon.iconset/icon_16x16@2x.png
sips -z 32 32     icon.png --out icon.iconset/icon_32x32.png
sips -z 64 64     icon.png --out icon.iconset/icon_32x32@2x.png
sips -z 128 128   icon.png --out icon.iconset/icon_128x128.png
sips -z 256 256   icon.png --out icon.iconset/icon_128x128@2x.png
sips -z 256 256   icon.png --out icon.iconset/icon_256x256.png
sips -z 512 512   icon.png --out icon.iconset/icon_256x256@2x.png
sips -z 512 512   icon.png --out icon.iconset/icon_512x512.png
cp icon.png         icon.iconset/icon_512x512@2x.png
iconutil -c icns icon.iconset
```

## Other Files

- `entitlements.mac.plist` — macOS Hardened Runtime entitlements (required for notarisation)
