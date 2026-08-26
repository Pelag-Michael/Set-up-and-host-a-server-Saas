# 15. Brand Account Posting Style and Image Assets

Doc 12 covers *participation* (a founder's personal presence in existing
communities). Doc 14 covers *creating* the official accounts (X, Instagram,
Facebook Page, LinkedIn Company Page) — identity kit, handles, first assets.
This doc picks up after creation: keeping bios/banners consistent over
time, writing content that doesn't read as one caption copy-pasted four
times, and specific platform gotchas that cost real time to discover.

## Credentials: one dedicated file, not `.env`

`.env` is for app runtime config. Social account logins are not — they
don't get read by any process, they get read by a human. Put them in their
own gitignored file (e.g. `data/keys/social_accounts_credentials.md`) with
one section per platform: handle/URL, login method (password, "Sign in with
Google", etc.), and who actually holds that login if more than one person
touches the accounts. Update it the moment a new credential is generated —
not at the end of the session, since that's exactly the point session
summaries get lossy.

## Identity: a founder's personal account vs. a placeholder brand persona

If you create a placeholder personal profile (not the founder's real
identity) purely to get a company/brand page bootstrapped, expect platforms
with anti-abuse heuristics (LinkedIn is the clearest example) to gate
"create a Company Page" behind account-age/connection-count signals a fresh
placeholder account won't have. The two ways through:

1. Age the placeholder account for real (connections, activity) before
   attempting page creation — slow.
2. Use a real, already-established personal account (the founder's own) to
   perform the one-time page-creation action, then hand day-to-day
   management back to whoever runs the brand accounts.

Neither leaks the founder's identity to the public by default: a Company
Page has **no public admin list** on LinkedIn. The only public link between
a person and a company there is the page's "People" tab, which only shows
people who've added that company to their own profile's Experience
section — something you control by simply not doing it. If privacy of the
founder's association matters, confirm this explicitly before creating the
page, not after.

## Per-platform bio pattern that holds up

- **One distinct sentence per platform** describing what the product does —
  not one shared slogan copy-pasted everywhere. Each platform's audience
  reads bios differently (X/LinkedIn: skimmed fast; Instagram: aesthetic-
  first); a single generic line under-serves all of them.
- **No link inside the bio text.** Every platform has a dedicated link
  field (website field, "link in bio", Links section) — use that, not a
  URL pasted into the bio sentence itself. Keeps the bio scannable and
  avoids a dead/stale link surviving a bio edit.
- A short status line as needed (e.g. "Now in open beta") kept separate
  from the descriptive sentence, so it can be swapped independently as
  status changes without rewriting the positioning line.

## Cross-linking between platforms: don't

Resist the urge to fill in every platform's "link to our other social
accounts" field with links to the other three. Each platform gives you
essentially one primary link slot — spending it on a sister account dilutes
the one CTA that actually matters (usually your own site). The pattern that
works: **the product website is the one hub** — its footer and structured
data (`Organization.sameAs` in JSON-LD, see doc 13) link out to every real
social profile, and each individual social profile links back only to the
website. The one common exception: Facebook Pages have a dedicated "Links"
section supporting multiple entries without displacing the primary link —
safe to use for cross-links specifically because it doesn't compete with a
single CTA slot the way X/Instagram/LinkedIn's single link field does.

## A short cross-platform posting-style guide is worth writing once

Different platforms warrant different copy for the same
announcement — not a copy-pasted caption. A reusable shape:

| Platform | Format | Copy style | Audience |
|---|---|---|---|
| X | short demo clip/GIF, or a brief thread | 1–2 sentences, straight to the feature, no throat-clearing | builders, technical early adopters |
| LinkedIn | polished demo video or a longer post + carousel | serious tone, name the industry pain point first, then the solution | decision-makers, B2B, technical leads |
| Instagram | vertical video/carousel, visual-first | shortest copy of all four, closer to a philosophy/positioning line + status; link goes in bio, not caption (IG doesn't render links in captions as clickable) | aesthetic/consumer-leaning audience |
| Facebook | link preview post or short clip | clearest, most explicit copy of the four — put the link directly in the post, since this audience is less conditioned to "check the bio for the link" than Instagram's | broad/general reach |

Write this down once as an actual doc your team can reread, rather than
re-deriving tone-per-platform from scratch every time there's something to
announce. One shared visual asset across all four, four different write-ups
of the copy — not one asset-and-copy pair copy-pasted four times.

## Image assets: banner/avatar sharpness

A banner or cover photo that displays sharp only when clicked/zoomed (but
blurry in the normal feed/page view) is usually **not** a source-resolution
problem. Two things to check, in order:

1. **Confirm the source actually is high enough resolution** before
   assuming compression is the cause — measure it (e.g. Python/Pillow
   `Image.open(path).size`), don't eyeball it. Compare against the
   platform's stated minimum (each platform publishes recommended cover/
   banner pixel dimensions); if your source is already 2-3x the minimum,
   more resolution won't fix a still-blurry preview.
2. **If resolution is already sufficient, the platform's own re-compression
   is the likely cause** — several platforms (Facebook is the clearest
   example) apply much more aggressive lossy re-encoding to large PNGs than
   to JPEGs, especially for photographic/gradient content (as opposed to
   flat-color/text graphics, where PNG's lossless encoding is preserved).
   Re-export the asset as a **JPEG**, quality ~90-92, embedded sRGB
   profile, 4:4:4 chroma subsampling (`subsampling=0` in Pillow) instead of
   the default 4:2:0 — this typically produces a *smaller* file than the
   PNG while surviving the platform's re-compression pass with visibly less
   quality loss. A quick local re-encode (see below) is worth trying before
   asking anyone to re-export from the original design tool.

```python
from PIL import Image

im = Image.open("banner.png").convert("RGBA")
bg = Image.new("RGB", im.size, (255, 255, 255))
bg.paste(im, mask=im.split()[3])  # flatten transparency onto white
bg.save(
    "banner.jpg", "JPEG",
    quality=92, optimize=True, subsampling=0,
    icc_profile=im.info.get("icc_profile"),
)
```

On Windows, note that the `python` command may resolve to the Microsoft
Store stub launcher rather than a real interpreter even when a real
install exists — if `python --version` fails silently, check
`where python` for a second entry (e.g. under
`AppData\Local\Programs\Python\...`) and call that path directly, or
install Pillow for the interpreter that's actually first on `PATH`.

## LinkedIn Company Page gotchas specifically

- **Banner is not part of page creation.** The initial "Create a Page"
  wizard only has a Logo field. The banner/cover image lives under a
  separate control: Edit Page → Page info tab → Banner sub-tab → "Edit
  background" → upload. If a freshly created page has no banner, this is
  almost always a discoverability issue, not a failed upload — the control
  is just somewhere else than you'd expect.
- **"Another admin is trying to make changes to this page at the same time
  as you"** can appear as a transient false positive when saving a tab
  (e.g. Locations) even with only one admin/session active — a plain retry
  of the identical save, once or twice, is a reasonable first response
  before assuming a real concurrent-edit conflict.

## Where notes like this belong (a meta-lesson)

If your project keeps a folder of dated point-in-time status snapshots
(e.g. `docs/context/YYYY-MM-DD-*.md`, see how this pattern is used across
this guide's source project), resist filing a reusable guideline — like
this posting-style table, or a branding convention — into that folder just
because it was written on a particular day. Dated-snapshot folders are for
"here's where things stood as of this date"; evergreen guidance (brand
voice, style rules, playbooks like this one) belongs in a flat,
undated folder next to your other brand/style docs, so it doesn't get
silently treated as stale once a newer dated file exists on an unrelated
topic.
