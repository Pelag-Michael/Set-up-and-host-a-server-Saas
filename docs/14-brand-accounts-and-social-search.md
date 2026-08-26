# 14. Brand Accounts and In-App Social Search

Doc 12 is the *community* playbook (participate as a person, don’t spam
forums). This doc is the *official product presence*: X, LinkedIn Company
Page, Instagram, Facebook Page — bound to the brand mailbox, using the
real logo, and wired back to the site so Google and the platforms can
treat them as one entity.

These are not a ranking cheat. They corroborate the brand. Feed reach
still needs real posts.

## Two different “search” problems

| Layer | What it is | What actually moves it |
|---|---|---|
| In-app profile search | User types the brand into X/IG/LinkedIn/Facebook search | Handle, display name, bio, category, website URL, consistency |
| In-app feed / For You | The platform *recommends* posts | Watch time, replies, recency, network, spam health |
| Google finding social | Google indexes public profiles/posts | Same as site SEO, plus `sameAs` and real outbound links |

Optimizing the profile wins **brand** queries inside the app. It does not
put you first for a niche query. That needs content + community (doc 12).

## Bind everything to the brand mailbox

Use the Google account that already owns DNS / Search Console / outbound
mail — not a personal Gmail.

- Verification codes land in that inbox. Keep it open while signing up.
- X web signup may refuse email and only allow it in the mobile app.
  **Google OAuth** with the brand Google account is the reliable web path.
- When Google’s account picker appears, the first row is often a personal
  account already in the browser. Pick the brand address explicitly.

Do not create real community accounts (Reddit, Discord user, third-party
forums) as the brand bot. Those stay on doc 12 rules, with a disclosed
persona if you use one.

## Identity kit (reuse on every platform)

| Field | Rule |
|---|---|
| Display name | The brand, unchanged. Not `Brand AI` as the *name*. |
| Handle | Prefer `@Brand`. If taken, a modifier handle (`@BrandAI`, `@BrandSpace`) is fine — display name still `Brand`. |
| Website | Canonical `https://yoursite.com/` |
| Visible bio | Category language people search: AI, orchestrator/controller, creative workflows, closed-beta status. Not a single-app wedge. Not protocol internals. |
| Hidden / later copy | Supported-app names belong on the site (JSON-LD / `llms.txt` / “works with”), and in proof posts, not in the identity line. |
| Visuals | Same mark for avatar; same Quiet/dark banner family for headers. Generate variants *from the real logo*, don’t invent a second mark. |
| CTA | Match reality (`Closed beta`, `Request access`). |

X People search matches username + profile name plus social score. A
modifier handle still works if the display name starts with the brand.

If the perfect handle is taken, take the cleanest available modifier and
move on. Random suffix suggestions from the signup form (`brandl0k4`) are
not brand assets.

## Assets

Need at least:

- Square avatar (from the official mark)
- X header (~1500×500 class)
- LinkedIn cover (~1128×191 or current spec)
- OG / Facebook (~1200×630) — the site `og:image` should match this
  family; a 512×512 mark is a stopgap only (see doc 13)

Generate from the logo with an image model if you have one; keep palette
and mark recognizable. Upload during onboarding when the platform offers
it — cheaper than hunting settings later.

## Platform notes (organic, not ads)

**X.** Google OAuth → date of birth (required even for a business; not
public) → username. Skip passkeys/notifications if you want. Then Edit
profile: banner, bio, website. Pin one “what this is” post with the
domain and category phrase.

**LinkedIn.** Company Page name = brand. Tagline = category, not a tool
list. About opening paragraph same as the site H1 idea. Founder can add
it as employer after the Page exists. Empty Pages look worse than no
Page — have About + avatar + a few proof posts before making it public.

**Instagram / Facebook.** Name field can carry a short category after the
brand (`Brand · …`). Captions on proof posts may name a specific tool;
the bio should not.

Reserve handles early. Do not auto-crosspost the same caption everywhere.

## Wire it back to the site

Until this loop exists, Google still has a weak entity:

1. Profile website → `https://yoursite.com/`
2. Site footer / legal community column → the live profile URL (replace
   `href="#"`)
3. JSON-LD `Organization.sameAs` → the same URL, only after it resolves
4. Optional: `twitter:site` = the handle

If Discord/YouTube are not created yet, omit them. A dead `#` link is
worse than a missing column.

## What not to do

- Rename the product because `Brand AI` ranks easier than `Brand`.
- Stuff every app name into the display name or every caption.
- Buy followers, engagement, or reviews.
- Let an agent operate a normal user account in someone else’s Discord.
- Treat in-app search like Google SEO — there is no sitemap, no
  Search Console, no guaranteed #1 for a niche query.

## After create: minimum checks

- Logged-out / other-account search for the brand, `brand + AI`, and the
  category phrase.
- Click-through from profile to the site and from the site footer back.
- Search Console still has the homepage indexed (doc 13); social is extra
  corroboration, not a substitute.
