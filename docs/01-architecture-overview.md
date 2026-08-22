# 1. Architecture Overview

## The shape of a small self-hosted SaaS backend

```text
                        ┌─────────────────────┐
   Users ──HTTPS──────▶ │  Cloudflare (DNS,    │
                        │  proxy, tunnel)      │
                        └──────────┬───────────┘
                                   │ tunnel (no inbound ports opened)
                                   ▼
                     ┌─────────────────────────────┐
                     │        Your machine          │
                     │                               │
                     │  Frontend server (PHP/static) │  :8088
                     │  Backend API server (FastAPI) │  :8001
                     │  Admin panel (local only)      │  :8010
                     │  SQLite DB(s)                  │
                     └─────────────────────────────┘
```

Three things make this viable for one person to run without a real ops
platform:

1. **A tunnel, not a port-forward.** You never open inbound ports on your
   router/firewall. A tunnel daemon (e.g. `cloudflared`) makes an *outbound*
   connection to the provider, which then proxies public traffic back
   through it. This also means your public domain works even if your
   machine's IP changes.
2. **SQLite, not a database server.** For anything under a few hundred
   thousand rows, a file-based DB removes an entire category of ops work
   (no separate DB process to keep alive, back up, or secure network access
   to). Back it up by copying the file.
3. **Everything else is a plain OS process**, started by a script and kept
   alive by your OS's own scheduler (Windows Task Scheduler / systemd /
   launchd), not a container orchestrator. This is the right trade for one
   machine; revisit if you ever need more than one.

## The pieces, and why each exists

- **Domain + DNS provider** (e.g. Cloudflare) — also doubles as: tunnel
  provider, email routing (receive), and (optionally) email sending, all on
  one free/cheap account. See `02-domain-and-dns.md`.
- **Backend API server** — the actual product logic (auth, licensing,
  whatever your product does). Runs as a single process, talks to SQLite,
  signs its own tokens if you need stateless auth. See
  `03-backend-api-server.md`.
- **Frontend** — whatever serves your marketing site / app shell. Doesn't
  need to be fancy; a PHP built-in server or a static file server both work
  fine for low-to-medium traffic. See `04-frontend-hosting.md`.
- **Admin panel** — a *separate*, unauthenticated-by-network-topology tool
  (bound to `127.0.0.1` only, never routed through the tunnel) for the
  operations only you should be able to do: issuing keys, revoking access,
  reading signup data. Not exposed publicly, ever.
- **Outbound mail** — almost always the part people forget to plan for.
  You need it the moment your product sends a single transactional email
  (signup confirmation, license key, password reset). Budget real setup
  time for this; see `05-transactional-email-setup.md`.
- **Scheduled jobs** — anything that needs to happen "once a day" (expiry
  reminders, cleanup, digest emails) without a real job queue. See
  `09-automation-and-scheduling.md`.

## Boundary between "internal" and "public"

Everything running on your machine that's reachable via the tunnel is
*public*, full stop — even routes you think of as "internal", because the
same process/port serves both. If you need a route that only your own
frontend server should call (e.g. "mint a real credential"), you must guard
it in the application layer:

- A shared-secret header, checked against a value only your two processes
  know (see `08-security-and-secrets.md`).
- A loopback-only check (`request.client.host in ("127.0.0.1", "::1")`),
  since your frontend and backend run on the same machine.

Both together, not either alone — the secret alone doesn't stop it from
being hit remotely if leaked; the loopback check alone doesn't stop another
process on the same machine.

The admin panel gets the stronger treatment: it's not just guarded, it's
bound to `127.0.0.1` at the socket level and never referenced by the
tunnel's routing config at all. Two layers of "this must never be public"
is not paranoia the first time you almost route it by accident.
