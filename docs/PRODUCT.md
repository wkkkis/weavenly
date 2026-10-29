# Weavenly — product overview

**Weavenly** is a privacy-first, local-first **job-hunt CRM**. Live at [weavenly.app](https://weavenly.app).

## What we're building

A job search is a pipeline: **find** roles → **tailor** your résumé → **fill** the application → **track** it.
Weavenly runs that pipeline **on your own machine**: your résumé, answers, mail and history never leave
your computer. AI runs on what you already have: your ChatGPT or Claude plan (sign in with ChatGPT, or the
Claude Code you installed), your own Anthropic or OpenAI key, or a local Ollama model. We never proxy the
calls, never pay a margin on tokens and never see your data. It's **free**, licensed AGPL-3.0.

- **Roles** — up to nine roles ("Frontend Engineer", "Product Designer", …), each with its target titles,
  its résumé, its color and its job search. Switch with a two-finger swipe on the sidebar or Ctrl+1…9; the
  extension's open panel follows.
- **Find** — every few hours the app downloads a public index of open jobs (Greenhouse, Lever, Ashby,
  Workable, SmartRecruiters, Recruitee, Personio, Teamtailor, Breezy, Pinpoint, BambooHR, Rippling and
  Workday boards, built by `backend/`) and reads five remote job boards, then matches them to each role
  **on your computer**. The best show up under **Found** on the Jobs page; AI rates up to 10 new ones per
  role per day (a setting; 0 turns it off).
- **Tailor** — an LLM rewrites your summary, bullets and skills to match the job **truthfully** (a job's
  keyword is woven in only when your résumé already grounds it; anything it can't ground is flagged) and
  shows a before/after **diff**. A **résumé library** keeps several versions and picks the best fit per job
  (you can override). No AI? A deterministic reorder/emphasis fallback still runs. Renders to PDF and DOCX.
- **Fill** — the browser extension fills application forms on any site (24 site adapters plus a generic
  engine). On a job's page without a form it offers only to create a résumé for that job; on a form for a
  different job it asks which résumé to attach (the original or the best ready one). The AI answers the
  form's questions from your résumé, except a short legal list (consents, date of birth and ID numbers,
  criminal record, right to work and visas, race and gender), which it asks you. Every fill is kept in the
  job's history. **You always click Submit.**
- **Track** — jobs move through Saved → Applied → Interview → Offer → Closed. Mail is read over IMAP with an
  app password (Gmail, Yandex, Mail.ru, iCloud, Yahoo or any server) and sorted **mostly by heuristics**
  (sender domain, ATS patterns, keywords), with the LLM only on the uncertain cases. The inbox has tabs for
  interviews, replies, results, recruiters, mail not linked to a job yet, and **suspicious** mail (fake
  recruiters are kept apart and never change a job). A rejection for a job you applied to, even from a hiring
  system's shared address (Greenhouse, Lever, Ashby, Workday), closes the job and shows a notification. An
  interview invite for a job you applied to elsewhere (LinkedIn, by email, through an agency) becomes that job
  at Interview, with its description looked up and a short **interview prep**: what the company is, what the
  round covers and what to have ready, what to ask, and the steps to an offer. Follow-ups, interview transcripts, and a **daily goal**
  (10 applications by default, a streak that survives weekends, one evening reminder).

The app speaks English and Russian.

## The pieces

Five repos: the root `weavenly` and four submodules.

| Dir | What it is | Stack |
|-----|------------|-------|
| **`app/`** | The product — desktop app for macOS, Windows and Linux; also runs the local server the extension talks to. | Electron + React + Vite + TS |
| **`extension/`** | Browser extension that fills application forms from your résumé, published in the Chrome Web Store. | MV3 + esbuild + TS |
| **`frontend/`** | The site, weavenly.app, in English and Russian. | Astro, plain CSS |
| **`backend/`** | The public job index builder (runs every 8 hours in GitHub Actions, publishes static files) and a magic-link sign-in API the app doesn't use. | TS, Cloudflare R2; Fastify + Postgres |

The desktop app and the extension talk over a **local** server (`127.0.0.1:9217`) — nothing routes through
a cloud. The job index is the same files for everyone; what a person searches for never leaves their
computer.

## Money

The app and the extension are **free**. There is **no paid feature**: everything runs on your own machine
and your own AI plan or key, so there's nothing to host and nothing to charge for. No ads, no data sale. If
an optional hosted convenience (a managed AI key for people who don't want to connect their own) is ever
added, it stays optional and the local app remains free. That restraint *is* the product.

## Brand & feel

The app's look: white cards on a soft cool gray, ink text, a deep indigo accent, and each role's color
(lavender, coral, peach, sand, sage, sky, periwinkle, slate, sunset) carrying the window frame and the
bands, under a faint film grain; Nunito for display text, the system font for the rest. The site borrows
it on warm paper, with the role colors on loud "stage" colors. The mark is the app's icon, never a letter. Voice: direct, honest, plain — "your ChatGPT or Claude plan", "your email".
The site and the extension don't say "open source", "BYOK", "Companion" or "GitHub" (their tests enforce
it). The site's tokens are in [`../frontend/README.md`](../frontend/README.md), the extension panel's in
`extension/DESIGN.md`.

## Status

- **Desktop app** — v0.3.14, downloadable from weavenly.app (macOS 13+ on Apple silicon and Intel, signed
  and notarized; Windows and Linux unsigned), with in-app updates. See `launch-docs/release-checklist.md`.
- **Extension** — 1.0.2, public in the Chrome Web Store since 2026-10-19.
- **Job index** — built and published every 8 hours to `index.weavenly.app`.
- **Sign-in API** — exists in `backend/`; nothing in the app needs an account.

Work happens on feature branches merged into `main`; designs and plans are dated documents in
[`superpowers/`](superpowers/). See [`../CONTRIBUTING.md`](../CONTRIBUTING.md).
