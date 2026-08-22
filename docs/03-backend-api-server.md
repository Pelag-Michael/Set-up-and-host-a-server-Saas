# 3. Backend API Server

## Process model

One long-running process, one port, bound to `127.0.0.1` (the tunnel is
what makes it reachable publicly — never bind `0.0.0.0` just to "make it
work", that also makes it reachable from anything else on your LAN).

```text
python -m uvicorn --app-dir . app:app --host 127.0.0.1 --port 8001
```

(Or the equivalent for your framework/language — the pattern is the same:
one process, one loopback-bound port, a process supervisor restarts it if it
dies.)

## Config via `.env`, not hardcoded values

Every path, port, and secret the server needs should come from environment
variables loaded from a `.env` file at startup, with sane defaults for local
dev:

```text
DB_PATH=./data/app.sqlite3
SIGNING_PRIVATE_KEY_PATH=./data/keys/signing_private.pem
BOOTSTRAP_RATE_LIMIT_PER_MINUTE=10
```

A tiny hand-rolled loader (read the file, split on `=`, `setdefault` into
`os.environ`) is enough — you don't need a dependency for this. See
`08-security-and-secrets.md` for what goes in here and how it's protected.

## Database: SQLite is enough until it isn't

- One file per logical database is fine (e.g. a licenses/accounts DB
  separate from a leads/signups DB) — simpler backups, simpler reasoning
  about what's safe to touch in tests.
- **Migrations are just "additive `ALTER TABLE` on startup"**: a schema-sync
  function that runs `CREATE TABLE IF NOT EXISTS` for new tables and checks
  `PRAGMA table_info(table)` before adding any new column, so it's always
  safe to run against a DB that already has rows in it. No migration
  framework needed at this scale — just never write a migration that drops
  or renames a column with existing data without a deliberate, separate step.
- Every write to a table other than the one directly being modified belongs
  in the same DB transaction. SQLite's `with conn:` (or your language's
  equivalent) gives you this for free — use it.

## Stateless auth without a session store

If clients need to prove "I'm still allowed to do this" without hitting the
DB every time, sign a short-lived token (JWT or similar) with a private key
that never leaves the server, and ship clients the *public* key to verify
with. Keep the signed-token lifetime short (days, not months) — it's a
convenience cache on top of the real source of truth (your DB row), not a
replacement for it. Always re-validate against the DB on the operations that
matter (activation, revocation).

## Internal-only endpoints living in a public-facing app

If your frontend server needs to ask the backend to do something privileged
(mint a credential, send an email) and both run on the same machine, add an
endpoint guarded by:

1. A shared-secret header compared against an env var only the two
   processes read.
2. A loopback-only check on the request's client address.

```text
POST /internal/do-privileged-thing
  X-Internal-Token: <shared secret>
  (and request.client.host must be 127.0.0.1 / ::1)
```

This endpoint is *not* safer than your public ones just because you call it
"internal" in the code — it's exposed via the exact same tunnel/port as
everything else unless you go out of your way to guard it. Test both the
"wrong token" and "right token from the wrong host" rejection paths.

## Admin operations: a separate tool, a separate port, no tunnel route

Whatever lets you (and only you) issue/revoke credentials, read signup data,
etc. should be:

- A separate process/port from the public API.
- Bound to `127.0.0.1` only.
- Never referenced anywhere in your tunnel's ingress config — not even
  behind auth. The fewer places "is this route public?" can be gotten wrong,
  the better.

A minimal local web UI (a plain HTTP server rendering server-side HTML with
POST-redirect-GET forms) is enough for one operator; you don't need a
frontend framework for a tool only you use. The POST-redirect-GET pattern
matters specifically so refreshing the result page after an action (like
"generate a key") doesn't resubmit the form and double-create something.
