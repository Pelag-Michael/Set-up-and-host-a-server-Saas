# 8. Security & Secrets Management

## Where secrets live

**One `.env` file, gitignored, at your repo root.** Every process (backend
API, frontend PHP, standalone scripts) reads from the same file so there's
one place to look, one place to rotate a value, and one place that must
never be committed.

```gitignore
.env
data/keys/
secrets/
*.pem
*.key
data/*.sqlite3
```

- `.env` — every credential: SMTP passwords/API keys, internal shared
  tokens, signing keys' *paths* (not the key material itself if it's a
  separate file).
- Any directory holding actual private key files (signing keys, DEKs).
- Any `*.sqlite3` database that holds real user data (emails, IPs, license
  keys) — these are runtime data, not source, and often contain PII.

A `.env.example` (committed) listing every variable *name* with a
placeholder value is worth maintaining — it's the fastest way for you (or
anyone else) to reconstruct a working `.env` after a fresh clone, without
ever putting a real value in git history.

## Generating secrets

Use your language's cryptographically-secure random generator, not
`random`/`Math.random()`:

```python
import secrets
token = secrets.token_hex(32)          # e.g. for a shared internal-API secret
key_id = secrets.token_hex(2).upper()  # shorter, for a human-facing license/invite code
```

Give each generated secret a clear name tied to its purpose
(`INTERNAL_API_TOKEN`, not `SECRET1`) — six months later you need to know
what rotating it will break.

## The moment a secret is generated, write it down twice

1. Into `.env` immediately (not "I'll paste it in later" — App Passwords
   and API keys are frequently shown *exactly once* by the provider's UI
   and cannot be viewed again, only regenerated).
2. Into a password manager or other durable backup *outside* this machine.
   `.env` living only on one disk is a single point of failure — if that
   disk is lost, every credential in it needs to be regenerated from
   scratch (usually possible, but real downtime while you redo the 2FA /
   phone-verification dance for each provider).

Never paste a real secret value into a chat log, an issue tracker, a
commit message, or documentation — including "just so I remember it" notes.
Docs should reference *where* a secret lives (`.env`'s `FOO_KEY`) and how to
regenerate it, never the value itself.

## Guarding internal endpoints (recap from `03-backend-api-server.md`)

Two independent checks on any endpoint that shouldn't be callable by the
general public, even though it's technically reachable through the same
public port as everything else:

1. Shared-secret header, compared against an `.env` value both sides read.
2. Loopback-only source check (`127.0.0.1` / `::1`).

Test the *rejection* paths explicitly (missing token, wrong token, wrong
source address) — it's easy to write the happy path and never confirm the
guard actually guards anything.

## Rate limiting anything that can trigger a side effect

Any public endpoint that causes your server to do something costly or
abusable on your behalf (send an email, mint a credential, call a paid
external API) needs a limit independent of normal request validation:

- **Per identity** — e.g. "one license key per email address, ever" (not
  per request) stops the same form from being resubmitted to spam one
  target inbox.
- **Per source IP** — e.g. "max N submissions per IP per day" stops a
  single bot from cycling through many identities quickly.

Both together; either alone has an obvious bypass.

## What actually needs elevated OS privileges (less than you'd think)

- Registering a **per-user** scheduled task (daily job, your own login
  session) — no admin rights needed.
- Registering a **SYSTEM-level** task that runs regardless of anyone being
  logged in (e.g. "start on machine boot before any user logs in") — does
  need admin rights.
- If a task only needs to run daily while you're normally logged in
  anyway, prefer the per-user route — fewer permission prompts, easier to
  set up remotely, and it's still reliable enough for a solo project.

## Reviewing before you commit

Before any commit, actually look at what's staged (`git status` /
`git diff --cached`), not just what you meant to change — especially after
using a broad "add everything" command. Grep the diff for anything that
looks like a credential pattern (a long hex string, `-----BEGIN PRIVATE
KEY-----`, a provider-specific key prefix) before pushing, even if you're
confident you already avoided it — a five-second check beats a rotated
credential and a scrubbed git history later.
