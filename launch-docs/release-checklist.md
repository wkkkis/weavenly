# Releasing Weavenly: owner checklist

How a desktop release is built and published, and the one-time setup it needs. The pipeline is live:
the tags `v0.3.5` … `v0.3.13` went out through it.

Already set up:
- `weavenly.app` DNS is at Cloudflare;
- the R2 bucket `weavenly-downloads`, public at `https://downloads.weavenly.app`, is in `frontend/vercel.json`;
- the site's download buttons are on (`RELEASES_LIVE = true` in `frontend/src/components/home/Download.astro`).

## How downloads work

- **Build and upload.** `release.yml` builds macOS (signed and notarized; Apple silicon and Intel as two
  builds, not one universal app), Windows (NSIS, unsigned) and Linux (AppImage, unsigned). Each build uploads
  its files to **Cloudflare R2** under `releases/<tag>/`.
- **Publish.** Once all three builds succeed, `publish-release.yml` copies them to the bucket's root:
  - `Weavenly-arm64.dmg`, `Weavenly-arm64.zip`, `Weavenly-x64.dmg`, `Weavenly-x64.zip`, and
    `Weavenly.dmg` (a copy of the Apple silicon one, for links made before the split);
  - `Weavenly-Setup.exe`;
  - `Weavenly.AppImage`;
  - the update feeds `latest-mac.yml`, `latest.yml` and `latest-linux.yml`.

  Installers are uploaded first and the feeds last, so an updater never reads a feed that points at a
  missing file. Publishing can be re-run by hand (Actions → Publish release → tag) for a tag whose files
  are already in `releases/<tag>/`.
- **The site** links to `https://weavenly.app/download/<file>`. `frontend/vercel.json` redirects that
  to `https://downloads.weavenly.app/<file>`.
- **The app's updater** reads `https://weavenly.app/download/latest*.yml` (`app/package.json` →
  `build.publish`, generic provider) and follows the same redirect.

## 1. Cloudflare R2 (one time, done)

1. **Bucket.** R2 → create a bucket (`weavenly-downloads`).
2. **Public access.** Bucket → Settings → Custom Domains → add `downloads.weavenly.app` (needs the DNS at
   Cloudflare).
3. **API token.** R2 → Manage API tokens → create one with Object Read & Write on that bucket. Keep the
   Access Key ID and the Secret Access Key.
4. **The site.** `frontend/vercel.json` points `/download/:file` at the bucket's public URL. With
   `RELEASES_LIVE` on, the site build fails if that URL (or the Chrome Web Store link) is missing.

The public job index has its **own** bucket and key (served at `https://index.weavenly.app/v1/`); see
`.github/workflows/job-index.yml`.

## 2. Actions secrets (root repo → Settings → Secrets and variables → Actions)

| Secret | Used by | What |
|---|---|---|
| `SUBMODULE_PAT` | every workflow | Fine-grained token with **Contents: read** on **all five** repos: `weavenly`, `weavenly-app`, `weavenly-extension`, `weavenly-frontend`, `weavenly-backend`. The root is public; the four submodules are private, and checkout fetches them with this token. |
| `R2_ACCOUNT_ID` | release, publish, job index | Cloudflare account ID (dashboard → R2 → right column) |
| `R2_ACCESS_KEY_ID` / `R2_SECRET_ACCESS_KEY` | release, publish | the downloads bucket's R2 API token from §1.3 |
| `R2_BUCKET` | release, publish | the downloads bucket's name |
| `INDEX_R2_ACCESS_KEY_ID` / `INDEX_R2_SECRET_ACCESS_KEY` / `INDEX_R2_BUCKET` | job index | the job index bucket and its own token |
| `APPLE_ID` | release (macOS) | Apple ID email of the developer account |
| `APPLE_APP_SPECIFIC_PASSWORD` | release (macOS) | app-specific password from appleid.apple.com |
| `APPLE_TEAM_ID` | release (macOS) | Team ID (developer.apple.com → Membership); electron-builder reads it to notarize |
| `CSC_LINK` | release (macOS) | base64 of the Developer ID Application `.p12` |
| `CSC_KEY_PASSWORD` | release (macOS) | the `.p12` password |

Paste secrets without a trailing newline.

## 3. Releasing the desktop app

1. Bump `version` in `app/package.json`, merge, and bump the root's `app` pointer (and update the root
   docs if the release changes what they say; see `CLAUDE.md`).
2. Push `main` to GitHub. The checks run on the push.
3. `git tag v0.x.y && git push origin v0.x.y` in the root repo.
4. `release.yml` runs: three builds, then the upload to `releases/v0.x.y/`, then `publish-release.yml`
   copies them to the bucket's root.
5. **Check.** `https://weavenly.app/download/Weavenly-arm64.dmg` (and `Weavenly-x64.dmg`, `.exe`,
   `.AppImage`) downloads, and `…/download/latest-mac.yml` shows the new version.

A manual run (Actions → Release Desktop App → Run workflow, not on a tag) builds all three and keeps
them as workflow artifacts. It uploads nothing, so you can use it as a dry run.

## 4. Releasing the extension

The extension ships through the Chrome Web Store, not this pipeline: bump `version` in
`extension/manifest.json` and `extension/package.json`, run `npm run package` in `extension/`, and upload
`weavenly-extension.zip` to the existing store item. The listing texts and the review notes are in
`extension/docs/chrome-web-store-listing.md`.
