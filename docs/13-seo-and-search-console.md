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

## After the site is indexed: why branded search still misses you

Indexed ≠ ranking. A brand-new domain can be “URL is on Google” in Search
Console and still fail a bare-brand search (`YourBrand`) because:

1. **Name collision.** If the brand string already belongs to older
   companies, products, artists, or baby-name pages, Google treats the
   query as an ambiguous entity. A *modifier* people actually type
   (`YourBrand AI`, `YourBrand app`) disambiguates; the bare name does not.
2. **Thin association.** A `<title>` that is only the brand name confirms
   the name and says nothing about category. Pair the brand with a stable
   category phrase in title, H1, and the first paragraph.
3. **SPA shell.** If the first HTML response is metadata plus
   `<div id="root"></div>`, crawlers that skip JS never see product copy.
   Google *can* render JS, but it is a queue, not a guarantee, and social
   crawlers usually do not. Put a real `<h1>` + one factual paragraph +
   real `<a href>` links in the *initial* HTML. A short, genuine
   screen-reader-style block is a stopgap; prerender/SSR is the real fix.
4. **One-app titles.** Do not put a single integration (the current GTM
   wedge) into the *homepage* title. It trains search and humans that the
   product *is* that one app. Keep homepage language general (category +
   “many tools”). Put specific apps in a “works with” section, JSON-LD
   `featureList`, and `llms.txt` — the machine-facing layer.
5. **Dummy social links.** Footer `href="#"` for X/Discord/YouTube is not
   a placeholder Google can use. Only publish URLs that resolve. Add those
   same URLs to Organization JSON-LD `sameAs` after the profile is live.

### Brand name vs search modifier

Keep the **legal / display brand unchanged**. If users type
`Brand + AI` (or `Brand + app`), put that modifier in:

- title / meta description / OG / Twitter text
- JSON-LD `alternateName` (the search-shaped variant, not a keyword dump)
- social *handle* if the clean `@Brand` is taken

Do **not** rename the product to `Brand AI`. Display name stays `Brand`.
The modifier is discoverability, not a new brand.

Visible copy: category language (orchestrator, command surface, creative
workflows, AI). Hidden / structured: supported-app names, product lines.
Do not put protocol internals (internal bridges, private APIs) in public
copy.

### JSON-LD that actually helps entity matching

One `SoftwareApplication` blob is a start. Once you have a real homepage
and at least one public profile, ship a small `@graph`:

- `WebSite` — `name` = brand, `alternateName` = search variants you
  actually want (`Brand AI`, the domain)
- `Organization` — same brand, `logo`, `sameAs` only for live profiles
- `SoftwareApplication` — factual description, OS, `featureList` of real
  capabilities / supported tools

Do not invent ratings, user counts, awards, or “supported” apps that are
not actually in the current public product.

### SPA routes that share one `index.html`

If the PHP/static router always serves the same `index.html`, every path
inherits the homepage canonical. Sitemap listing `/download` then does
nothing useful — Google will collapse it to `/`.

Cheap fix without a rebuild: the fallback router reads `index.html` and
string-replaces `<title>`, canonical, and `og:url` when the path is a
known public route. Only do this for routes that actually render distinct
content. Keep the replacements exact-string and unique so you cannot
accidentally rewrite a substring inside JSON-LD.

### Search Console — what actually worked after the first pass

- Verify the **Domain** property under the **brand** Google account (the
  same mailbox that already owns DNS / email / Gravatar), not a personal
  one. Switching accounts in the Google picker is easy to miss; “you
  don’t have access” usually means the wrong `authuser`.
- URL Inspection is the source of truth for “is this URL on Google,” not
  one incognito SERP.
- After a real head/schema change, **Request indexing again**. Repeat
  requests do not jump the queue.
- Sitemap submit via the UI can no-op silently on a freshly verified
  property. Retry later with the full sitemap URL
  (`https://yoursite.com/sitemap.xml`). Success looks like: status
  **Success**, last read = today, discovered URL count matching the
  sitemap.
- Re-check `curl` of live HTML after deploy — the git worktree you edited
  is not always the document root the tunnel is serving. Copy (or edit)
  the live webroot too.

## Quick verification checklist

```bash
curl -s -o /dev/null -w "%{http_code}\n" https://yoursite.com/llms.txt
curl -s -o /dev/null -w "%{http_code}\n" https://yoursite.com/robots.txt
curl -s -o /dev/null -w "%{http_code}\n" https://yoursite.com/sitemap.xml
curl -s https://yoursite.com/ | grep -oE '<title>[^<]+|<meta name="robots"[^>]*>|<link rel="canonical"[^>]*>|<h1>[^<]+'
curl -s https://yoursite.com/robots.txt   # read the FULL output, not just your own file's content — check for injected blocks
curl -s https://yoursite.com/download | grep -oE '<title>[^<]+|<link rel="canonical"[^>]*>'
```

Also, in Search Console: URL Inspection → “URL is on Google”; Sitemaps →
row present, status Success. Then search incognito for the bare brand
*and* `brand + modifier` from more than one country/language before
declaring ranking status.
