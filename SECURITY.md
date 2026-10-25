# Security Policy

Weavenly handles résumés, employment history, email, AI keys and the mail app
password. If you find a way to get at any of that, we want to hear about it.

## Reporting a vulnerability

**Do not open a public issue for a security bug.**

Report privately through [GitHub Security Advisories](https://github.com/wkkkis/weavenly/security/advisories/new),
or email **iskhak.i.codes@gmail.com** with `[security]` in the subject.

Please include what you'd need to reproduce it yourself: affected component,
version, steps, and what an attacker gets out of it.

This is a small project — expect a first reply within **7 days** and a fix or a
concrete plan within **30 days** for anything confirmed as high severity. We'll
credit you in the advisory unless you'd rather stay anonymous.

## Supported versions

Only the latest release gets security fixes. There are no long-term support
branches.

| Component | Supported |
|-----------|-----------|
| Desktop app — latest release | ✅ |
| Companion extension — latest Web Store version | ✅ |
| Anything older | ❌ |

## Scope

In scope:

- **Desktop app** (`app/`) — Electron main/renderer, IPC surface, the résumé
  tailoring pipeline, secret storage.
- **Companion HTTP server** (`app/src/companion/server.ts`) — the loopback
  service on `127.0.0.1:9217` that the extension talks to.
- **Companion extension** (`extension/`) — content scripts and service worker.
- **Backend** (`backend/`) — the job index builder (what it fetches and the
  files it publishes to `index.weavenly.app`), and the sign-in API only if you
  can reach a deployed instance.

Out of scope:

- Attacks needing physical access to an unlocked machine, or malware already
  running as the user. Weavenly's data is protected by the OS account, not
  against it.
- Vulnerabilities in an AI provider (Anthropic, OpenAI, Ollama), a mail provider,
  or an ATS — report those to the vendor.
- Missing hardening with no demonstrated impact (headers, version disclosure).

## What Weavenly's threat model assumes

Worth knowing before you file:

- **The loopback server answers any origin.** `Access-Control-Allow-Origin: *`
  is deliberate: the extension's content scripts call the server from whatever
  ATS page they run on, so the origin can't be allow-listed. Every endpoint
  except `/status` and `/pair` requires a bearer token that only the paired
  extension holds, and `/pair` is rate-limited. A report needs to show actual
  data access, not just that the port answers.
- **Secrets are encrypted at rest via the OS keychain** (`safeStorage`, through
  `app/src/main/secret-store.ts`): the AI key, the ChatGPT sign-in and the mail
  app password. On a system with no keychain backend the app falls back to
  plaintext — this is documented, not a finding on its own.
- **AI calls go straight to the provider you configured** (your key, your
  ChatGPT or Claude plan, or local Ollama). Weavenly runs no proxy and the app
  sends no telemetry. A path that sends user data anywhere else is a bug and we
  want to know.
