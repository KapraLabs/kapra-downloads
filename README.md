# Kapra Downloads

Public distribution hub for Kapra desktop apps, browser installers, and extension packages. Source code stays in private repositories; **binaries and this manifest live here** so anyone can download without GitHub access to those repos.

## For users

- **Web4 Browser** (Windows / macOS / Linux): use the download buttons on [kaprachain.com/web4/download](https://kaprachain.com/web4/download) or open the [Releases](https://github.com/KapraLabs/kapra-downloads/releases) page.
- **Web4 Chrome extension**: same site and releases; use the `.zip` where provided for “Load unpacked” installs.

## How it works

1. [`downloads-manifest.json`](./downloads-manifest.json) on the default branch lists **direct URLs** to current installers (usually GitHub Release asset links in *this* repo).
2. The Kapra website fetches that JSON (with caching) and redirects download clicks to the right file.
3. If a manifest field is empty, the site may fall back to the **latest Release** in this repo and pick a matching asset by file type.

## Repository layout

| File | Purpose |
|------|---------|
| `downloads-manifest.json` | Versioned map of public download URLs (edit when you ship). |
| `MAINTAINERS.md` | Step-by-step release process for the team. |

## Security

Only publish artifacts you have built from trusted CI or signed release processes. Do not commit private keys or tokens.
