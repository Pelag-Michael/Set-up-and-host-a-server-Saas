# 11. Pitfalls & Lessons Learned

Every specific thing that actually broke (or almost did) while setting up a
system like this, in one place, so the next person recognizes the shape of
the problem faster than "why is this failing".

## DNS / Email

- **Gmail's "auto-detect SMTP server" when adding a domain alias picks your
  MX record** (your *inbound* mail router), not an outbound relay. If you
  let it auto-fill, sending will fail with an auth error against a server
  that was never meant to accept your login. Manually enter the real
  outbound relay host (`smtp.gmail.com`) instead.
- **A DKIM CNAME record must be DNS-only, not proxied**, on any DNS
  provider that offers CDN-style proxying (Cloudflare's orange cloud, for
  example). A proxied CNAME resolves through the provider's edge instead of
  acting as a plain alias, which breaks DKIM verification silently — the
  record *looks* correctly published, mail just fails signature checks.
- **No DMARC record at all is the default state**, not an edge case —
  check for one explicitly before assuming any advanced feature that
  requires it (like a verified sender logo) is reachable.
- **Adding a new sender's DNS records is additive.** Don't delete an
  existing SPF/MX/DKIM record when setting up a second sender on the same
  domain — multiple `include:` mechanisms and distinct DKIM selectors
  coexist fine.

## SEO / discoverability

- **A CDN/proxy provider can inject content into `robots.txt` that isn't in
  your own file.** Cloudflare's "Managed robots.txt" (AI Crawl Control →
  Signals) auto-prepends a block disallowing named AI crawlers
  (`GPTBot`, `ClaudeBot`, `Bytespider`, etc.), often on by default. Your own
  `User-agent: * / Allow: /` at the bottom doesn't override a more specific
  named-bot block above it — robots.txt matching is per-user-agent group,
  not "last rule wins." Always check what's actually served
  (`curl -s https://yoursite.com/robots.txt`), not just what's in your repo.
- **A freshly-verified Search Console domain property can silently fail to
  accept a sitemap submission** — the form appears to succeed but the
  sitemaps table stays empty with no error shown. Not a mistake in the URL
  you entered; just retry after a short wait.
- **Removing a `noindex` meta tag doesn't get a site indexed by itself** —
  Google still has to crawl and decide to index it, which can take days
  with no prompting. Use Search Console's URL Inspection → Request Indexing
  for the pages you actually care about instead of waiting on organic
  discovery.

## Account creation / verification friction

- **2FA is a hard prerequisite for app-specific passwords** on most
  providers — if 2FA isn't already on, budget that as its own step (it
  needs a phone number) before you can even attempt the SMTP relay setup.
- **A transactional email provider generating an SMTP/API credential often
  requires phone verification first**, even on a free tier — this is
  standard anti-abuse, not a sign of a broken signup.
- **A transactional email provider may require a real physical address**
  at signup, separate from phone verification — required by anti-spam
  disclosure regulations (footer-address requirements), not a data-grab.
- **Creating multiple accounts on the same third-party service back-to-back
  from one browser/IP will likely get rate-limited or blocked** by that
  service's own anti-abuse system after the second attempt, even though
  each individual signup is legitimate. If you need several distinct
  accounts on the same service, space the signups out rather than looping
  through them immediately.

## Local dev-server quirks

- **PHP's built-in server (`php -S`) does not load extensions from a
  system `php.ini` by default** the way a properly configured
  Apache/Nginx+PHP-FPM setup does. A script that uses `mb_substr()` or
  `PDO` with SQLite will throw "undefined function"/class errors under the
  built-in server specifically, even though the exact same script runs
  fine once actually deployed with the right flags. Fix: pass
  `-d extension=mbstring -d extension=pdo_sqlite` (and `extension_dir`)
  explicitly when launching the built-in server for local testing.
- **A script's own directory is not always first on the module import
  search path** on every interpreter/OS combination, even though that's
  the documented default behavior. If sibling-module imports mysteriously
  fail only in certain shells/terminals, explicitly insert the script's own
  directory into the import path at the top of the file — cheap, harmless
  if redundant, and removes an entire class of "works on my machine"
  inconsistency.
- **An HTTP test client's simulated request doesn't originate from
  `127.0.0.1` by default** — it typically uses a placeholder test hostname.
  A test for a loopback-only-guarded endpoint will fail confusingly (looks
  like the guard rejected a legitimate request) until you explicitly
  configure the test client to simulate connecting from `127.0.0.1`.

## Deploy / bundling

- **A cached, hashed bundle filename needs to actually change** for
  browsers to pick up a new version — editing the file's contents in place
  without renaming it means users with a cached copy see nothing different
  until the cache header's max-age expires.
- **If your build source and your deployed bundle have drifted apart**
  (someone hand-patched the deployed artifact directly at some point
  without porting the fix back to source), rebuilding from source and
  deploying it will silently *revert* every hand-patch that was never
  ported back. Diff first; don't assume source is authoritative just
  because it's source.

## Process / operational UX

- **A start script that only acts when something isn't already running,
  and gives no feedback otherwise, looks broken on every run after the
  first.** If the point of the script is operator reassurance ("is my
  system actually up"), always end it with something visible confirming
  current state — not just silence when there was nothing new to start.

## The meta-lesson

None of the above are exotic — they're all "the happy-path tutorial doesn't
mention this part" gaps. The fix in every case was the same shape: notice
the actual error message closely (it usually says exactly what's wrong),
don't assume the first plausible cause, and verify the fix against the real
system once mocked/isolated testing has already given you confidence it's
correct.
