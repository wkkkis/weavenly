# Releasing Weavenly: owner checklist

How a desktop release is built and published, and the one-time setup it needs. The pipeline is live:
the tags `v0.3.5` … `v0.3.23` went out through it.

Already set up:
- `weavenly.app` DNS is at Cloudflare;
- the R2 bucket `weavenly-downloads`, public at `https://downloads.weavenly.app`, is in `frontend/vercel.json`;
- the site's download buttons are on (`RELEASES_LIVE = true` in `frontend/src/components/home/Download.astro`).

## How downloads work

- **Build and upload.** `release.yml` builds macOS (signed and notarized; Apple silicon and Intel as two
  builds, not one universal app), Windows (NSIS, unsigned) and Linux (AppImage, unsigned). The Windows and
  Linux builds sign their update feeds with Weavenly's own key (`latest.yml.sig`, `latest-linux.yml.sig`; §2a).
  Each build uploads its files to **Cloudflare R2** under `releases/<tag>/`.
- **Publish.** Once all three builds succeed, `publish-release.yml` copies them to the bucket's root:
  - `Weavenly-arm64.dmg`, `Weavenly-arm64.zip`, `Weavenly-x64.dmg`, `Weavenly-x64.zip`, and
    `Weavenly.dmg` (a copy of the Apple silicon one, for links made before the split);
  - `Weavenly-Setup.exe`;
  - `Weavenly.AppImage`;
  - the signatures `latest.yml.sig` and `latest-linux.yml.sig` (both or neither);
  - the update feeds `latest-mac.yml`, `latest.yml` and `latest-linux.yml`.

  Installers are uploaded first, then the signatures, and the feeds last, so an updater never reads a feed
  that points at a missing file. Publishing can be re-run by hand (Actions → Publish release → tag) for a tag whose files
  are already in `releases/<tag>/`.
- **The site** links to `https://weavenly.app/download/<file>`. `frontend/vercel.json` redirects that
  to `https://downloads.weavenly.app/<file>`.
- **The app's updater** reads `https://weavenly.app/download/latest*.yml` (`app/package.json` →
  `build.publish`, generic provider) and follows the same redirect. On Windows and Linux, once the app
  carries the update key, it downloads an update only when `latest*.yml.sig` confirms the feed's files.

## 1. Cloudflare R2 (one time, done)

1. **Bucket.** R2 → create a bucket (`weavenly-downloads`).
2. **Public access.** Bucket → Settings → Custom Domains → add `downloads.weavenly.app` (needs the DNS at
   Cloudflare).
3. **API token.** R2 → Manage API tokens → create one with Object Read & Write on that bucket. Keep the
   Access Key ID and the Secret Access Key.
4. **The site.** `frontend/vercel.json` points `/download/:file` at the bucket's public URL. With
   `RELEASES_LIVE` on, the site build fails if that URL (or the Chrome Web Store link) is missing.

The public job index has its **own** bucket and key (served at `https://index.weavenly.app/v1/`); see
`.github/workflows/job-index.yml`. The H-1B data (`.github/workflows/visa-data.yml`) goes to the same bucket
under `v1/visa/`, and the weekly hiring report (`.github/workflows/hiring-report.yml`) under `v1/report/`.

## 2. Actions secrets (root repo → Settings → Secrets and variables → Actions)

| Secret | Used by | What |
|---|---|---|
| `SUBMODULE_PAT` | every workflow | Fine-grained token with **Contents: read** on **all five** repos: `weavenly`, `weavenly-app`, `weavenly-extension`, `weavenly-frontend`, `weavenly-backend`. The root is public; the four submodules are private, and checkout fetches them with this token. |
| `R2_ACCOUNT_ID` | release, publish, job index, visa data, hiring report | Cloudflare account ID (dashboard → R2 → right column) |
| `R2_ACCESS_KEY_ID` / `R2_SECRET_ACCESS_KEY` | release, publish | the downloads bucket's R2 API token from §1.3 |
| `R2_BUCKET` | release, publish | the downloads bucket's name |
| `INDEX_R2_ACCESS_KEY_ID` / `INDEX_R2_SECRET_ACCESS_KEY` / `INDEX_R2_BUCKET` | job index, visa data, hiring report | the job index bucket and its own token |
| `VERCEL_DEPLOY_HOOK` | site refresh, hiring report | optional: a Vercel deploy hook URL for the site's `main` (Vercel → project → Settings → Git → Deploy Hooks); the daily `site-refresh.yml` posts to it so the `/h1b` pages rebuild, and `hiring-report.yml` after publishing a week, so its page is live for the posts |
| `APPLE_ID` | release (macOS) | Apple ID email of the developer account |
| `APPLE_APP_SPECIFIC_PASSWORD` | release (macOS) | app-specific password from appleid.apple.com |
| `APPLE_TEAM_ID` | release (macOS) | Team ID (developer.apple.com → Membership); electron-builder reads it to notarize |
| `CSC_LINK` | release (macOS) | base64 of the Developer ID Application `.p12` |
| `CSC_KEY_PASSWORD` | release (macOS) | the `.p12` password |
| `UPDATE_SIGNING_KEY` | release (Windows, Linux) | the update signing key's private half (PEM), §2a. Needed once `app/src/main/update-key.ts` carries the public half: a tag then stops at "Check the update signing key" without it or with another key |

Paste secrets without a trailing newline.

## 2a. Update signing key (one time)

Windows and Linux installers carry no code signature, so their updates are signed with Weavenly's own
Ed25519 key instead. Make it once, on your own computer, in `app/`:

```bash
npx tsx scripts/update-signing-key.ts
gh secret set UPDATE_SIGNING_KEY --repo wkkkis/weavenly < ~/weavenly-update-signing-key.pem
```

The script writes the public half into `app/src/main/update-key.ts` (commit it, then bump the root's `app`
pointer) and the private half into `~/weavenly-update-signing-key.pem`, readable only by you. Back that file
up (a password manager), then delete it: every app that carries the public key refuses an update signed by
any other key. The private key never goes into a repo or a chat. Until `update-key.ts` carries a key, the
app checks nothing and a missing secret only skips the signing.

## 3. Releasing the desktop app

1. Bump `version` in `app/package.json`, merge, and bump the root's `app` pointer (and update the root
   docs if the release changes what they say; see `CLAUDE.md`).
2. Push `main` to GitHub. The checks run on the push.
3. `git tag v0.x.y && git push origin v0.x.y` in the root repo.
4. `release.yml` runs. First each build checks that the secrets from §2 it needs are set, and stops naming
   any that are missing. Then three builds, the upload to `releases/v0.x.y/`, and `publish-release.yml`
   copies them to the bucket's root. After adding a missing secret, re-run the failed run (Actions → the run
   → Re-run failed jobs): the tag stays.
5. **Check.** `https://weavenly.app/download/Weavenly-arm64.dmg` (and `Weavenly-x64.dmg`, `.exe`,
   `.AppImage`) downloads, and `…/download/latest-mac.yml` (and, once signed, `latest.yml.sig`) shows the new
   version.

A manual run (Actions → Release Desktop App → Run workflow, not on a tag) builds all three and keeps
them as workflow artifacts. It uploads nothing, so you can use it as a dry run.

## 4. Releasing the extension

The extension ships through the Chrome Web Store, not this pipeline: bump `version` in
`extension/manifest.json` and `extension/package.json`, run `npm run package` in `extension/`, and upload
`weavenly-extension.zip` to the existing store item. The listing texts and the review notes are in
`extension/docs/chrome-web-store-listing.md`.
