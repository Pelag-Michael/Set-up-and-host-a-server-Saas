# 13. SEO, `llms.txt`, and Getting Indexed

Doc 12 covers whether `llms.txt` is worth doing and how to write one. This
doc covers the rest of the "can anyone actually find this site" problem:
on-page SEO basics, the accidental blockers that keep a real site invisible
long after launch, and the actual steps to get a new domain indexed by
Google — not just "submit a sitemap and wait."

## Before touching anything: check for an accidental `noindex`

The single most common reason a site that "should" be indexable isn't:
someone (often a scaffold/template, or a leftover from a staging setup)
left `<meta name="robots" content="noindex, nofollow">` in the page head.
This actively tells every search engine "don't index me" — it overrides
everything else you do. Check for it first, before any other SEO work:

```bash
curl -s https://yoursite.com/ | grep -i 'name="robots"'
```

If it's there and you actually want to be found, remove it (or change to
`content="index, follow"`). This is a real decision, not just a technical
fix — confirm with whoever owns the product that public discoverability is
actually wanted at this stage (a closed-beta product might have set this
deliberately).

## On-page SEO checklist

For each real, public route (not client-side-only fragments that never
resolve to a URL a crawler would fetch):

- `<title>` — specific, not just the brand name alone once you have more
  than one real page.
- `<meta name="description">` — one or two factual sentences.
- `<link rel="canonical" href="...">` — the URL you want indexed for this
  content, especially important once you have any URL variants (with/without
  trailing slash, `?utm_` params, etc.)
- Open Graph tags (`og:title`, `og:description`, `og:url`, `og:type`,
  `og:image`) + `twitter:card` (`summary_large_image`) + matching
  `twitter:title`/`twitter:description`/`twitter:image` — controls how the
  link looks when shared, not search ranking, but cheap and high-visibility.
- `og:image` needs a real ~1200×630 asset. If you don't have one yet, a
  square brand mark is a legitimate stopgap (better than nothing) — note it
  as a follow-up, don't block the rest of the SEO pass on commissioning
  a banner.
- JSON-LD structured data matching what's actually shipped — don't
  overclaim capabilities/platforms/pricing that aren't public yet.

### SPA caveat

If the site is a client-side-rendered single-page app with one static
`index.html` serving every route (no server-side rendering, no per-route
head management library), these tags are effectively site-wide — you
can't give `/pricing` a different `<title>` than `/` without adding
something like a head-management library and route-aware content. Don't
promise per-route metadata you can't actually implement without that; note
it as a known limitation instead of faking a canonical URL for a route that
doesn't have its own rendered content.

## `robots.txt` and `sitemap.xml`

If they don't exist yet (check `curl -s https://yoursite.com/robots.txt` —
a 404 means they don't), add them:

```
# robots.txt
User-agent: *
Allow: /
Allow: /llms.txt

Sitemap: https://yoursite.com/sitemap.xml
```

```xml
<!-- sitemap.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://yoursite.com/</loc>
    <changefreq>weekly</changefreq>
    <priority>1.0</priority>
  </url>
  <!-- one <url> per real route -->
</urlset>
```

Only list routes that actually render real content — don't enumerate
hypothetical future pages.

## The Cloudflare gotcha: your own `robots.txt` may not be the whole story

If the domain is proxied through Cloudflare, **check what's actually being
served**, not just what you wrote:

```bash
curl -s https://yoursite.com/robots.txt
```

Cloudflare has a zone-level feature (dashboard: your domain → **AI** →
**Signals** → **Managed robots.txt**) that, when enabled, *auto-injects* a
block at the top of your `robots.txt` response disallowing a long list of
named AI crawlers (`GPTBot`, `ClaudeBot`, `Bytespider`, `CCBot`, `Amazonbot`,
`Meta-ExternalAgent`, `Google-Extended`, etc.) — separate from, and
prepended to, whatever's actually in your file on disk. This is often
**on by default** and you won't find it by reading your own repo.

This matters a lot if you just published `llms.txt` for AI agents to read:
robots.txt matching is per-named-user-agent, so a generic
`User-agent: * / Allow: /` at the bottom of the file does **not** override
a more specific `User-agent: GPTBot / Disallow: /` block above it. Those
named bots are still blocked regardless of what you wrote.

Two things worth knowing before you decide what to do about it:

1. **The blocked bots are specifically the training-corpus crawlers**, not
   the ones that answer live user questions. Most AI companies run separate
   user-agents for "crawl to build a training set" (`GPTBot`, `ClaudeBot`,
   `Google-Extended`) versus "fetch this page right now to answer a user's
   question" (`OAI-SearchBot`, `Claude-SearchBot`, `PerplexityBot`,
   `Applebot`) versus "user explicitly asked me to browse this" (`ChatGPT-
   User`, `Claude-User`). Check the **AI Crawl Control → Security** tab in
   Cloudflare — it shows actual request logs per bot. The search/answer bots
   are very often *already allowed and already crawling successfully* even
   while the training bots show `Disallow`. If your goal is "be findable
   when someone asks an AI assistant about this," that goal may already be
   met without touching anything.
2. Disabling the master "Managed robots.txt" toggle removes the block
   entirely (all AI crawlers, including training, get `Allow`) — a real,
   deliberate tradeoff (giving your content to model training runs for
   free, no attribution, no compensation), not a pure bug fix. Decide this
   explicitly rather than toggling it because a bot list looked alarming.

## Getting a new domain indexed (Google Search Console)

Publishing `robots.txt`/`sitemap.xml`/removing `noindex` does not make
Google index the site — it has to discover, crawl, and decide to index it,
which can take days to weeks with zero prompting. To speed this up:

1. **Add the property in Google Search Console** — use the **Domain**
   property type (covers `www`/non-www, all protocols) rather than
   URL-prefix, unless you have a specific reason not to.
2. **If the domain's DNS is on Cloudflare, use the built-in one-click
   authorization flow** instead of manually copying a TXT record: Search
   Console's domain-verification screen offers "Verify via Cloudflare.com"
   — clicking it opens a Cloudflare authorization screen showing exactly
   the one TXT record it's about to add, you approve once, and it's done.
   Faster and less error-prone than hand-copying a TXT value.
3. **Submit the sitemap** under Indexing → Sitemaps. Note: this submission
   form can silently no-op right after a domain property is freshly
   verified (empty result, no error shown) — if the table doesn't show your
   sitemap after submitting, wait a bit and retry rather than assuming
   you made a mistake.
4. **Directly request indexing for the homepage** via the URL Inspection
   tool (search box in the Search Console header) — inspect the URL, then
   use "Request Indexing." This is the fastest lever for a brand-new site;
   don't rely on sitemap submission alone for the first page you actually
   care about.
5. Set expectations: request-indexing is not instant. Hours to a couple of
   days is normal. A brand-new domain also has zero authority, so even
   once indexed, ranking for a branded search term takes more time and
   possibly backlinks — indexed and top-of-results are different
   milestones, don't conflate them when reporting status.

## Quick verification checklist

```bash
curl -s -o /dev/null -w "%{http_code}\n" https://yoursite.com/llms.txt
curl -s -o /dev/null -w "%{http_code}\n" https://yoursite.com/robots.txt
curl -s -o /dev/null -w "%{http_code}\n" https://yoursite.com/sitemap.xml
curl -s https://yoursite.com/ | grep -oE '<meta name="robots"[^>]*>|<link rel="canonical"[^>]*>'
curl -s https://yoursite.com/robots.txt   # read the FULL output, not just your own file's content — check for injected blocks
```
