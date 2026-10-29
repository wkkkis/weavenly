# Weavenly — root repo

> Privacy-first job hunt CRM. Local-first desktop app; AI on your own ChatGPT or Claude plan, your own key,
> or local Ollama. **Free**, licensed AGPL-3.0. Live at [weavenly.app](https://weavenly.app).

Weavenly is five git repos. This root repo pins the other four as submodules, owns the npm workspace (one
lockfile for all four) and the CI, and keeps the docs that span projects:

```
.
├── app/        → wkkkis/weavenly-app        Electron desktop app (the product)
├── extension/  → wkkkis/weavenly-extension  browser extension that fills application forms
├── frontend/   → wkkkis/weavenly-frontend   weavenly.app site (Astro)
└── backend/    → wkkkis/weavenly-backend    public job index builder + magic-link sign-in API
```

What the product is and where it stands: [`docs/PRODUCT.md`](docs/PRODUCT.md).

## Quick start

```bash
# Clone and install everything (submodules included). Node 22.12+.
git clone --recurse-submodules https://github.com/wkkkis/weavenly.git
cd weavenly
npm install                    # installs all four projects
# already cloned? → git submodule update --init --recursive

# The desktop app in dev
npm run dev:app                # Vite on http://localhost:5173 + the Electron window

# The browser extension (load extension/dist as unpacked in Chrome)
npm run dev:extension          # esbuild watch → extension/dist

# The site in dev
npm run dev:frontend           # http://localhost:4321

# The job index, built locally (publishes nothing)
npm --workspace backend run index:build -- --out /tmp/wv-index

# The sign-in API (needs Postgres; see backend/README.md)
npm run dev:backend
```

## Build & ship

```bash
npm run build:all              # app, frontend, backend, extension

# Desktop installers → app/release/
npm run package:mac            # DMG + zip, Apple silicon and Intel as separate builds
npm run package:win            # NSIS .exe
npm run package:linux          # .AppImage
npm run package:all            # all three
```

A release is a tag: bump `app/package.json`, merge, bump the `app` pointer here, then
`git tag v0.x.y && git push origin v0.x.y`. `release.yml` builds and uploads to Cloudflare R2, and
`publish-release.yml` puts the files where the site (`weavenly.app/download/<file>`) and the in-app updater
read them. The extension ships through the Chrome Web Store instead. Details, secrets and checks:
[`launch-docs/release-checklist.md`](launch-docs/release-checklist.md).

## The projects

### `app/` — desktop application

Electron + React + TypeScript + Vite. A menu-bar app on macOS (tray on Windows and Linux) that keeps
running when its window is closed, so the extension can always reach it.

- **Roles**: up to nine, each with its titles, résumé, color and job search.
- **Jobs**: Saved → Applied → Interview → Offer → Closed, plus **Found** — jobs matched on your computer
  from the public job index and five remote boards.
- **Résumé tailoring**: a truthful AI rewrite with a before/after diff, a résumé library with best-fit
  pick, PDF and DOCX.
- **Mail**: IMAP/SMTP with an app password; heuristic sorting first, the LLM only on uncertain mail;
  suspicious mail kept apart. An interview invite for a job applied to elsewhere becomes that job, with a
  short interview prep.
- **Daily goal**, follow-ups, interview transcripts; English and Russian.
- **Storage**: a `data/` folder of JSON/JSONL you own. Secrets (AI key, ChatGPT sign-in, mail password) are
  encrypted at rest through the OS keychain.
- **AI**: your ChatGPT or Claude plan, an Anthropic or OpenAI key, or local Ollama. No proxy.
- **Companion server**: `127.0.0.1:9217`, the extension's only way in (pair code → bearer token).

See [`app/README.md`](./app/README.md) and [`app/ARCHITECTURE.md`](./app/ARCHITECTURE.md).

### `extension/` — browser extension

Manifest V3, esbuild, TypeScript; in the Chrome Web Store. Runs on application forms on any site (24 site
adapters plus a generic engine), fills them from the app's data, asks only what it must, keeps every fill
in the job's history; the person clicks Submit. See [`extension/README.md`](./extension/README.md).

### `frontend/` — the site

Astro, plain CSS, English and Russian, deployed on Vercel. Home, What's new, Privacy, Terms, the job index
page, downloads and the extension link. See [`frontend/README.md`](./frontend/README.md).

### `backend/` — job index and sign-in

- **The public job index**: reads public ATS job lists (Greenhouse, Lever, Ashby, Workable,
  SmartRecruiters, Recruitee, Personio, Teamtailor, Breezy, Pinpoint, BambooHR, Rippling, Workday) for
  ~37,000 company boards every 8 hours and publishes compact static files to `index.weavenly.app/v1/`. No
  descriptions, no personal data, no API.
- **Magic-link sign-in** (Fastify + Postgres): kept for possible server features; the app doesn't use it.

See [`backend/README.md`](./backend/README.md) and [`ARCHITECTURE-BACKEND.md`](./ARCHITECTURE-BACKEND.md).

## CI / CD (`.github/workflows/`)

| Workflow | When | What |
|---|---|---|
| `app.yml` | PRs / pushes touching `app` | type-check (main + renderer), unit tests, build |
| `extension.yml` | … `extension` | type-check, manifest ↔ `atsHosts` drift check, tests, build |
| `frontend.yml` | … `frontend` | `astro check`, unit tests, build (Vercel deploys through its Git integration) |
| `backend.yml` | … `backend` | type-check and unit tests |
| `job-index-format.yml` | … `app` or `backend` | the two copies of the job index format and its fixture are identical |
| `job-index.yml` | every 8 hours (+ by hand) | builds and publishes the job index; the Monday run also finds new boards |
| `release.yml` | a `v*` tag (+ by hand as a dry run) | macOS (signed, notarized), Windows, Linux builds → R2 `releases/<tag>/` |
| `publish-release.yml` | called by `release.yml` (+ by hand) | copies a release to the bucket root: installers first, update feeds last |

`.github/dependabot.yml` keeps the root lockfile, the actions and the submodule pointers current.

This root repo is public, so its workflows run on GitHub's free Actions minutes; the four project repos
stay private, and every workflow checks them out with the `SUBMODULE_PAT` secret. A pull request from a
fork gets no secrets, so its checks can't fetch the submodules.

## Docs

- [`docs/PRODUCT.md`](docs/PRODUCT.md) — the product and its status
- [`CONTRIBUTING.md`](CONTRIBUTING.md) — setup, checks, adapters, PR flow
- [`ARCHITECTURE-BACKEND.md`](ARCHITECTURE-BACKEND.md) — what runs on a server and what never does
- [`SECURITY.md`](SECURITY.md) — reporting and the threat model
- [`launch-docs/release-checklist.md`](launch-docs/release-checklist.md) — releasing
- [`CLAUDE.md`](CLAUDE.md) — the rule that keeps these docs current (every repo has one)

## Why submodules

Each project has its own repo, history and PRs; the root pins which versions go together, so a release
tag here builds a known set. A change that spans projects lands in each repo, then one root commit bumps
the pointers (and updates the root docs if they describe what changed).

## License

**AGPL-3.0** — © 2026 Iskhak Ishmakhametov. See [`LICENSE`](./LICENSE) (each project carries the same
license). Contributions are accepted under AGPL-3.0 — see [`CONTRIBUTING.md`](./CONTRIBUTING.md).

## Contact

[hello@weavenly.app](mailto:hello@weavenly.app)
