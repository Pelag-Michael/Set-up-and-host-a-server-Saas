# 4. Frontend Hosting

## Pick the simplest thing that serves your files

For low-to-medium traffic, you don't need a reverse proxy stack:

- **Static site** (pre-built HTML/JS/CSS): any static file server, or even
  your DNS/tunnel provider's own static hosting if it has one.
- **PHP site with a couple of dynamic endpoints** (a contact form, a signup
  handler): PHP's own built-in server is genuinely fine for this traffic
  level: `php -S 127.0.0.1:8088 -t webroot router.php`. Add a `router.php`
  that serves static files directly if they exist on disk and falls back to
  your app entrypoint otherwise:

  ```php
  <?php
  $uri = parse_url($_SERVER['REQUEST_URI'] ?? '/', PHP_URL_PATH) ?: '/';
  $root = realpath(__DIR__);
  $candidate = realpath(__DIR__ . DIRECTORY_SEPARATOR . ltrim(str_replace('/', DIRECTORY_SEPARATOR, $uri), DIRECTORY_SEPARATOR));
  if ($root !== false && $candidate !== false && str_starts_with($candidate, $root) && is_file($candidate)) {
      return false; // let the built-in server serve it directly
  }
  require __DIR__ . '/index.html';
  return true;
  ```

- **PHP extensions**: the built-in server does *not* load extensions (like
  `mbstring`, `pdo_sqlite`) from a default `php.ini` the way a configured
  Apache/Nginx setup would. Launch it with explicit `-d` flags:
  ```text
  php -d "extension_dir=<path>\ext" -d extension=mbstring -d extension=pdo_sqlite -d extension=sqlite3 -S 127.0.0.1:8088 -t webroot router.php
  ```
  If a PHP script errors with "Call to undefined function mb_substr()" (or
  similar) only when run via the built-in server but not elsewhere, this is
  almost always the cause — extensions silently not loaded, not a code bug.

## Cache-busting a hand-served bundle

If your build hashes filenames (`app.a1b2c3.js`) and you serve the output
directly (not through a CDN that auto-invalidates), remember the *browser*
still caches that hashed filename aggressively (`max-age` measured in
hours). If you patch the bundle in place without renaming it, users with a
cached copy won't see the change until the cache expires. Two options:

1. Re-run your real build (regenerates the hash automatically) — the
   correct long-term answer.
2. If you're hand-patching a deployed bundle directly (see the next
   section for why you might), **bump a suffix on the filename yourself**
   (`app.a1b2c3-v2.js`) and update the HTML that references it. The HTML
   file itself should be served with no/short caching so this reference
   update takes effect immediately.

## When your build source and your deployed bundle have diverged

This happens more often than it should: someone hand-edits the deployed,
minified bundle directly (a copy fix, an emergency patch) without updating
the source that would regenerate it, and over time the two drift apart. If
you ever find the deployed bundle contains changes the source doesn't
(check for a distinguishing string only one side has), **do not rebuild
from source and deploy it** — that silently reverts every hand-edit fix that
was never ported back. Two honest options:

1. Treat the deployed bundle as the temporary source of truth: apply new
   changes there too (surgically, validate syntax after every edit with
   your language's own parser/linter — e.g. `node --check file.js`), and
   separately schedule time to actually port everything back into real
   source and do a clean rebuild.
2. Stop the drift now: diff what's different, port it into source, rebuild
   once, verify the rebuilt output matches what's live, then never hand-edit
   the bundle again.

Neither is fun; (1) buys time, (2) is the actual fix. Don't let (1) become
permanent.

## What to keep multiple copies of, and why

If your repo has more than one directory that looks like "the built site"
(a `dist/`, a `public/`, a `live-webroot/`), figure out and document which
one is *actually served* before touching any of them. Check what your
frontend server's document root is set to — that's the only one that
matters for behavior; the others may be stale build artifacts from earlier
tooling. Don't assume "the newest-looking one" is right; verify by grepping
for a build-specific hash or string that only appears in the live one.
