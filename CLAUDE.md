# Weavenly (root repo)

## Keep the docs current (always)

The docs describe the code as it is **now**. A change that alters behaviour, structure, commands, scripts,
configuration, env vars, endpoints, the data layout, supported sites or user-facing wording that a doc
describes updates that doc **in the same commit / PR**. A task is not done while a living doc contradicts
the code.

Before you finish any change:

1. Read your own diff. For every name it adds, renames or removes (file, folder, function, endpoint,
   command, script, env var, setting, setting label), search the living docs below for it and fix what
   they say.
2. Check each sentence you write against the code. Don't write from memory or copy from older docs;
   delete what is no longer true instead of leaving it or marking it "old".
3. Keep each file's language (a Russian doc stays Russian, an English one English) and its plain, concrete
   tone. Don't add roadmap items nobody decided.
4. Say in the PR description which docs you updated, or "Docs: nothing describes this".

Dated records are history, not living docs: `docs/superpowers/specs/*`, `docs/superpowers/plans/*` and
their reports describe what was decided on their date. Don't rewrite them; a new decision gets a new dated
spec.

This repo is public. Dated records, analyses and anything else internal don't go here: the ones that span
projects live in the private app repo (`app/docs/superpowers/`, `app/docs/history/`).

Weavenly is five repos (root `weavenly` with the submodules `app`, `extension`, `frontend`, `backend`). When
a change here alters something the root repo describes (`README.md`, `CONTRIBUTING.md`,
`ARCHITECTURE-BACKEND.md`, `docs/PRODUCT.md`, `launch-docs/`), update the root repo too, together with the
submodule bump.

### Living docs in this repo

- `README.md` — the monorepo map, quick start, CI workflows, repository layout
- `CONTRIBUTING.md` — setup, the checks to keep green, adapters, branch and PR flow
- `SECURITY.md`
- `ARCHITECTURE-BACKEND.md` — what runs on a server and what never does
- `docs/PRODUCT.md` — what the product is and its status
- `launch-docs/release-checklist.md` — how a release is built and published, the secrets it needs
- the header comments of `.github/workflows/*.yml`

A submodule bump is the moment to check these: read what the bumped repo changed and update the root docs
in the same commit.
