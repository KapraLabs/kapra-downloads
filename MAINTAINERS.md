# Maintainer guide: publishing to Kapra Downloads

## Principles

- **`kapra-downloads` is public.** Do not reference private URLs that require auth.
- Prefer attaching binaries to a **GitHub Release in this repo**, then pointing `downloads-manifest.json` at the resulting asset URLs.
- **End-user links** in README and marketing should point to **kaprachain.com** (`/download`, `/web4/download`, `/web4/all-releases`), not the GitHub Releases tab (which lists source archives). See **`kapra-website/docs/PUBLIC_DOWNLOADS_AND_RELEASES.md`** for the audited site map.

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

## Commits on `main` are not a Release

Pushing commits (for example updates to `downloads-manifest.json` or `kapra-web4-browser-update.json`) **does not** create a new row on the **Releases** tab and **does not** upload installers. The Releases page only changes when someone **Drafts a new release**, picks or creates a **tag** (e.g. `v0.1.1`), **attaches binaries**, and clicks **Publish**. Until then, **Latest** stays on the previous tag (e.g. `v0.1.0`) even if `main` has moved forward.

## Automated Windows release (kapra-web4-browser)

The private **`KapraLabs/kapra-web4-browser`** repo includes **Release to kapra-downloads** (GitHub Actions). When you **push to `main`** and **`package.version`** in **`tauri.conf.json`** changed vs the previous commit, **or** you **push a `v*` tag** after aligning versions, the workflow builds Windows artifacts, **creates or updates** the matching **Release** on **this** repo (`kapra-downloads`), and **pushes manifest commits** to `main` here.

Setup and operator steps: **`kapra-web4-browser` → `docs/RELEASE_AUTOMATION.md`**. You still attach **extension zip** (and optional macOS/Linux assets) manually to the new release if needed.

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

The Kapra **website** download API routes use: **env → `downloads-manifest.json` → latest GitHub Release on this repo**. Keep `downloads-manifest.json` on `main` aligned with the installer URLs you want users to get (e.g. after bumping the browser version); GitHub “latest” is the fallback when a platform entry is empty. Pin a URL with env when you need to override.

## 6. Icons (Web4 Browser)

If you change branding, from `kapra-web4-browser` regenerate icons from the 1024×1024 PNG:

```bash
npm run tauri icon src-tauri/icons/source-icon.png
```

## 7. Web4 Browser auto-updates (Tauri updater)

The desktop app checks **`kapra-web4-browser-update.json`** on launch (see `kapra-web4-browser` `src-tauri/tauri.conf.json` → `tauri.updater.endpoints`).

1. **Signing key:** The **public** key is embedded in the app as base64 (Tauri `pubkey` field). The **private** key must be available only in maintainer/CI secrets as `TAURI_PRIVATE_KEY` (see [Tauri updater](https://v1.tauri.app/v1/guides/distribution/updater/)). If you rotate keys, update `pubkey` in `tauri.conf.json` and ship a new installer before publishing updates signed with the new key.

2. **Release build:** From `kapra-web4-browser`, with `TAURI_PRIVATE_KEY` set:

   ```bash
   npm run tauri:build
   ```

   Upload the **NSIS updater zip** (not only the `.exe` installer), e.g. `KapraWeb4Browser_<version>_x64-setup.nsis.zip`, from `src-tauri/target/release/bundle/nsis/` to the **KapraLabs/kapra-downloads** GitHub Release for that version. The website’s `downloads-manifest.json` must use **Release tags that actually exist** (e.g. point Windows at `KapraWeb4Browser_0.1.0_…` on `v0.1.0` until you publish `v0.1.1` with new assets). If the updater zip is missing from a release, in-app updates to that URL will fail until you attach it.

3. **Update manifest:** Edit **`kapra-web4-browser-update.json`** on `main`:

   - Set `"version"` to the new semver (must be **greater** than the build baked into users’ apps for them to be prompted).
   - Set `"platforms"."windows-x86_64"."url"` to the **direct** asset URL of the `.nsis.zip`.
   - Set `"signature"` to the **base64 encoding** of the full minisign `.sig` file text (UTF-8) for that zip. Tauri’s bundler may emit a `.sig` next to the zip when signing succeeds; otherwise sign the same zip with the same private key and base64-encode the signature file contents.

4. **macOS / Linux:** Add `darwin-aarch64`, `darwin-x86_64`, and/or `linux-x86_64` entries when you publish signed updater bundles for those targets (see Tauri docs for bundle file names).
