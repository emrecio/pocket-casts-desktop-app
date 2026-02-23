# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

This is a personal fork of the original [Pocket Casts Desktop App](https://github.com/felicianotech/pocket-casts-desktop-app), maintained at `github.com/emrecio/pocket-casts-desktop-app`.

## Commands

**Run locally:**
```
npx electron main.js
```

**Build distributable packages (deb, rpm, zip):**
```
npx electron-forge make -- --arch=x64,arm64
```
Output goes to `out/make/`.

**Publish to GitHub releases:**
```
npx electron-forge publish --arch="x64,arm64"
```

**Build Flatpak locally:**
```
npm run dist-flatpak
```

No lint or test commands are configured.

## Git Configuration

Always use local git config for any changes in this repository — never modify the global git config. Use `git config` (without `--global`) to keep all config changes scoped to this repo.

## Architecture

This is a minimal Electron wrapper (~166 lines) for the official Pocket Casts web app. All application logic lives in a single file: `main.js`.

The app loads `https://play.pocketcasts.com/podcasts` in a `BrowserWindow`. There is no renderer-process code — the renderer is the external Pocket Casts webapp. The main process interacts with the webapp via `mainWindow.webContents.executeJavaScript()` to simulate button clicks for media controls.

**Key behaviors in `main.js`:**
- Single-instance lock (prevents multiple windows)
- Window size/maximization state persisted via a custom `Store` class writing JSON to the user data directory
- System tray icon with context menu (Show/Hide, Play/Pause, Skip back/forward, Quit)
- Global media key shortcuts (`MediaPlayPause`, `MediaPreviousTrack`, `MediaNextTrack`)
- Navigation guard: only `play.pocketcasts.com` is allowed in-window; external links open in the system browser
- `nodeIntegration: false` in webPreferences for security

## Version Management

When bumping the version, update all three of these:
1. `package.json` — `version` field
2. `snap/snapcraft.yaml` — `version` field
3. `flatpak/tech.feliciano.pocket-casts.appdata.xml` — add a `<release>` entry with changelog

## Release Workflow

See `development.md` for the full publishing steps. The high-level flow is:
1. Merge version bump PR to `trunk`
2. `npx electron-forge publish --arch="x64,arm64"` — creates GitHub release with `.deb`/`.rpm`/`.zip` assets
3. `snapcraft remote-build` — builds Snap packages from the `.deb` artifacts
4. Upload Snap builds via `snapcraft upload --release=stable`
5. Update Flatpak manifest PR with changelog from the GitHub release

CI/CD runs on CircleCI and is triggered by version tags matching `v\d+\.\d+\.\d+`.
