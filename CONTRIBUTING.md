# Contributing to Weavenly

Welcome! Weavenly is **free and open source** ([AGPL-3.0](LICENSE)) — read the code, file issues, and
send PRs. Start with [`docs/PRODUCT.md`](docs/PRODUCT.md) for what we're building, then the README of the
area you'll work in. Doing **frontend**? → [`frontend/README.md`](frontend/README.md).

By contributing, you agree your contributions are licensed under AGPL-3.0.

## Repo layout

Monorepo via npm workspaces — one `npm install` at the root installs everything.

- `app/` — Electron desktop app (the product)
- `extension/` — MV3 browser extension that fills application forms (in the Chrome Web Store)
- `frontend/` — Astro marketing site (weavenly.app)
- `backend/` — the public job index builder (runs in GitHub Actions) and a magic-link sign-in API the app doesn't use

## Setup

The repo uses git submodules (one per project), so clone recursively:

```bash
git clone --recurse-submodules https://github.com/wkkkis/weavenly.git
cd weavenly
npm install
# already cloned? → git submodule update --init --recursive
```

Run what you're working on:

```bash
npm run dev:frontend     # landing site → http://localhost:4321
npm run dev:app          # desktop app (Vite + Electron window)
npm run dev:extension    # esbuild watch → extension/dist (load it unpacked in Chrome)
npm run dev:backend      # sign-in API, needs Postgres — see backend/README.md
npm --workspace backend run index:build -- --out /tmp/wv-index   # build the job index locally
```

## Before you push — keep these green

The same commands CI runs (`.github/workflows/`):

- App: `cd app && npx tsc -p tsconfig.main.json --noEmit && npx tsc --noEmit`, then `npm --workspace app test`
  and `npm --workspace app run build`
- Extension: `npm --workspace extension run check`, `npm --workspace extension run check:hosts`,
  `npm --workspace extension test`, `npm --workspace extension run build`
- Frontend: `npm --workspace frontend run check`, `npm --workspace frontend test`,
  `npm --workspace frontend run build`
- Backend: `npm --workspace backend exec tsc -- --noEmit` and `npm --workspace backend test`
- Job index format: `backend/src/job-index/format.ts` and `app/src/discovery/index-format.ts` must stay
  identical (`job-index-format.yml` compares them)

Some tests read across projects (for example the app checks that its copies of shared files match the
extension's and the site's), so run them from a recursive clone of this repo.

## Keep the docs current

A change that alters what a doc says updates that doc in the same PR: the repo's README and architecture
docs, and the root docs (`README.md`, `docs/PRODUCT.md`, `ARCHITECTURE-BACKEND.md`, `launch-docs/`) when it
changes something they describe. Each repo's `CLAUDE.md` lists its living docs. Dated specs and plans in a
repo's `docs/superpowers/` are history: a new decision gets a new spec instead of an edit to an old one.

## Adding or fixing an ATS adapter (the most useful contribution)

Autofill works through a generic fill engine plus small per-ATS adapters, and ATS sites change their DOM
often — so keeping adapters working is the single highest-leverage thing you can contribute.

- **Where:** [`extension/src/content/platforms/`](extension/src/content/platforms/) — one file per ATS
  (`workday.ts`, `greenhouse.ts`, `lever.ts`, `ashby.ts`, `icims.ts`, …), registered in `index.ts`;
  anything without an adapter goes through `generic.ts` and the shared heuristics in `common.ts`. Copy the
  closest existing adapter as a template.
- **Hosts:** a new site also goes into `platforms/atsHosts.ts` and `manifest.json`;
  `npm --workspace extension run check:hosts` fails when they drift apart.

When you add or fix one:
1. Prefer **stable selectors** (`data-*`, `aria-*`, label text) over hashed class names that break weekly.
2. Add a fixture + test next to [`extension/src/content/__tests__/`](extension/src/content/__tests__/)
   (see `workday-widgets.test.ts` for the pattern) so the selector drift is caught next time.
3. Note the ATS + the URL you tested against in the PR description.

Adapters are assistive only — they **fill** the form; the user always clicks Submit. Don't add anything
that auto-submits or runs off the user's machine.

## Branch & PR flow

1. Branch off `main`: `git checkout -b your-feature`
2. Commit small, with clear messages.
3. Open a PR into `main`; keep the description concrete — what, why, and how you tested.
4. **Never commit secrets** (`.env`, API keys) or the local `app/data/` user data — they're gitignored; keep it that way.

## House rules

- Match the surrounding code's style; reuse existing components/utilities before adding new ones.
- Privacy-first is a product promise, not a tagline — nothing user-private leaves the device.
- Use the existing design tokens and role colours, never new hard-coded colours: the site's are in
  `frontend/README.md`, the extension panel's in `extension/DESIGN.md`.

Questions → ask the maintainer.
