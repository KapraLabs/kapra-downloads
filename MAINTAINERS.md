# Maintainer guide: publishing to Kapra Downloads

## Principles

- **`kapra-downloads` is public.** Do not reference private URLs that require auth.
- Prefer attaching binaries to a **GitHub Release in this repo**, then pointing `downloads-manifest.json` at the resulting asset URLs.

## 1. Web4 Browser installers

Build from the private `kapra-web4-browser` repo (bundling enabled in `tauri.conf.json`):

```bash
cd kapra-web4-browser
npm install
npm run tauri:build
```

Collect outputs from `src-tauri/target/release/bundle/` (e.g. `.msi` / `-setup.exe` on Windows, `.dmg` on macOS, `.deb` / `.AppImage` on Linux).

**Linux:** Tauri **1.x** + **wry 0.24** currently fails to compile on Linux with **Rust ≥ ~1.79** (upstream `SettingsExt` / rustc interaction). The `build-linux` job in `kapra-web4-browser` is disabled until you migrate to **Tauri 2** or upstream fixes land. If you produce a `.deb` / `.AppImage` locally, attach it here and set `web4Browser.linux` in the manifest.

**macOS DMG in CI** is **Apple Silicon (`aarch64`)** by default. Add an `macos-13` (Intel) job later if you need `x86_64` DMGs.

Bundled app id / filenames use **`KapraWeb4Browser`** (no spaces) so macOS `bundle_dmg.sh` is reliable in GitHub Actions.

## 2. Web4 extension zip

From `kapra-web4-extension`, produce a zip of the packaged extension (e.g. `dist/` or Chrome Web Store–ready folder). Name it clearly, e.g. `kapra-web4-extension-0.1.0.zip`.

## 3. Create a GitHub Release here

1. In **KapraLabs/kapra-downloads**, open **Releases** → **Draft a new release**.
2. Choose a tag (e.g. `web4-2025.03.23` or semantic `v0.2.0`).
3. Upload all new binaries and the extension zip as **release assets**.
4. Publish the release.

Copy each asset’s **direct** link (right-click → copy link). They look like:

`https://github.com/KapraLabs/kapra-downloads/releases/download/<tag>/<filename>`

## 4. Update `downloads-manifest.json`

Edit the file on `main`:

- Set `web4Browser.windows` / `mac` / `linux` to the matching installer URLs.
- Set `web4Extension.zip` to the extension archive URL.
- Bump `schemaVersion` only when you change the JSON shape (website reads `schemaVersion === 1`).
- Update `updated` (ISO date).

Commit and push. The marketing site will pick up changes within its cache window (a few minutes) or on next deploy if you bake env overrides.

## 5. Optional website overrides

On Vercel (or similar) you can still set:

- `NEXT_PUBLIC_WEB4_BROWSER_WINDOWS_URL` / `_MAC_` / `_LINUX_`
- `NEXT_PUBLIC_WEB4_EXTENSION_ZIP_URL`
- `KAPRA_DOWNLOADS_MANIFEST_URL` — alternate manifest URL (fork/staging)

Env wins over manifest; manifest wins over “guess from latest release.”

## 6. Icons (Web4 Browser)

If you change branding, from `kapra-web4-browser` regenerate icons from the 1024×1024 PNG:

```bash
npm run tauri icon src-tauri/icons/source-icon.png
```
