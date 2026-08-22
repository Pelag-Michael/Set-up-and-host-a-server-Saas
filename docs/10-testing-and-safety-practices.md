# 10. Testing & Safety Practices

The single habit that matters most here: **anything that sends a real email,
mints a real credential, or writes to your real database is not something
you casually run "just to check it works."** Every one of those has a real
side effect on real infrastructure or a real (if fake-looking) user record.
Build the habit of testing against isolation *before* you build the feature
that needs it, not after the first time you accidentally spam a real
provider's abuse system.

## Isolated database for every test

Never point a test at your real data file. Construct your storage layer
against a fresh temp path per test:

```python
import tempfile
from pathlib import Path

tmp_db = Path(tempfile.mkdtemp()) / "test.sqlite3"
store = YourStorageClass(tmp_db)
```

If your app reads its DB path from an environment variable at import time
(common for a script-style app rather than a properly dependency-injected
one), you have two options:
- Monkeypatch the module-level variable directly in the test
  (`module.DB_PATH = tmp_db`) before calling into it — works for simple
  scripts.
- Set the env var *before* importing the module (works when the module
  reads env at import time, not lazily) — needed when the app builds
  significant state (like a whole web framework app) at import time.

## Mock the actual external side effect, not just "the network"

For anything that sends mail, mock the function that does the actual send
(`smtp_client.send(...)`), and assert on *what* it was called with (the
right recipient, the right template) — not just that "something didn't
crash". This catches real bugs (wrong recipient, missing personalization)
that a bare "did it throw" test won't.

```python
from unittest import mock

with mock.patch.object(mailer_module, "send_email", return_value={"sent": True}):
    result = do_the_thing_that_sends_mail(...)
mailer_module.send_email.assert_called_once()
assert mailer_module.send_email.call_args.kwargs["to"] == "expected@example.com"
```

## Test the failure paths, not just the happy path

For anything with a fallback (primary provider down → secondary provider),
write a test that forces the primary to fail and asserts the fallback was
used — and a separate test that forces *both* to fail and asserts you get
a clear error back rather than a silent no-op or an unhandled exception.
These are exactly the paths that only run in production during an actual
incident, when you'd most like to already know they work.

## Test guard/dedup logic explicitly

Anything with a "don't do this twice" guard (don't re-send a reminder, don't
mint a second credential for the same identity) deserves a test that calls
the function twice and asserts the second call is a no-op — not just a test
that the first call works. Off-by-one dedup bugs (checking the wrong field,
comparing before normalizing case/whitespace) are exactly the kind of thing
that looks correct on first read and fails silently in exactly the
scenario the guard exists for.

## Before you run anything against real infrastructure

A short mental checklist, every time:

- [ ] Does this call a real external API (email provider, payment
      processor, third-party account creation)?
- [ ] Does this write to the database file real users' data lives in?
- [ ] If yes to either — is there an isolated/sandbox equivalent I should
      use instead, or do I need to accept this is a deliberate real test
      (and if so, use throwaway/clearly-marked test data, and clean it up
      after)?

If you do need to test against the real system once (confirming an actual
end-to-end integration works, which mocked tests can't fully prove), use
data that's unambiguously identifiable as a test (a `+test` suffix on an
email address you control, a name like "E2E Test") and revoke/delete the
resulting record immediately after confirming it worked.

## Environment quirks worth knowing exist (so you recognize them, not this exact fix)

- A script run via `python script.py` is *supposed* to have its own
  directory as the first import search path automatically — if imports
  that should obviously resolve don't, on some machine/interpreter
  combinations this can silently not hold. If a script can't find a sibling
  module it should trivially see, explicitly insert its own directory into
  the import path at the top of the file rather than assuming the default
  behavior — check what convention the rest of your codebase already uses
  for this and match it.
- An HTTP test client's simulated request often does **not** originate from
  `127.0.0.1` by default (it uses a placeholder test hostname) — if you're
  testing an endpoint that checks the caller's source address, you may need
  to explicitly configure the test client's simulated client address to
  match what you're checking for.
- A CLI tool's *default* invocation may not load extensions/config that a
  properly deployed instance of the same tool does — if something works in
  "real" deployment but throws "undefined function/module" errors when you
  try to reproduce it via a quick CLI test, check whether the deployed
  version passes extra flags or a specific config file your ad-hoc
  invocation is missing, before assuming it's a code bug.
