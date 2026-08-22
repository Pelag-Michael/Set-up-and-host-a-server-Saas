# 7. Branding: Icons & Avatars

Getting your logo to show up consistently — favicon, email sender avatar,
etc. — is a checklist of small independent settings, not one setting. Do
all of them; skipping one just means that one surface still looks generic.

## Favicon

- Use your actual brand mark (the icon-only version, not a full wordmark
  logo) — a wordmark shrunk to 16-32px is illegible.
  - This deserves double-checking: repos often accumulate *several*
    logo/icon files over a project's life (early placeholder icons, an
    "app icon" variant for a different sub-product, the real brand mark).
    Confirm you're using the actual current brand mark, not a leftover
    placeholder that happens to be named `favicon.svg` already.
- Render it on a background matching your site's actual theme color (check
  your CSS variables for the real background color rather than guessing),
  with rounded corners consistent with how your other app icons are treated
  — this reads as "a deliberate icon" rather than "a random square export".
- SVG favicons work in all modern browsers and scale cleanly at every size —
  prefer SVG over a single fixed-resolution PNG/ICO if your logo is
  vector-based.
- If your site has multiple copies of `favicon.svg` in different build
  output directories, verify which one your server's document root actually
  serves (see `04-frontend-hosting.md`) and update that one — but sync all
  the copies anyway so the next rebuild doesn't silently revert it.

## Sender avatar for your transactional/support emails

There are two independent, free things that together cover most email
clients:

1. **The Google Account profile photo of the mailbox your addresses forward
   to / send from.** Since most of your addresses ultimately land in (or
   send through) one real inbox account, setting *that account's* profile
   photo to your brand mark makes it show up as the sender avatar for
   Gmail-to-Gmail mail. Five-minute change: Google Account settings →
   personal info → profile picture. Upload a square image (crop tools in
   the flow will handle the exact circle-crop).

2. **Gravatar**, for every public-facing address independently
   (`support@yourdomain.com`, `hello@yourdomain.com`, etc.) — used by
   Outlook, Apple Mail, and a number of other clients that look up an
   avatar by a hash of the sender's email address. Gmail itself does *not*
   use Gravatar (it uses the Google Account photo instead), so this and
   (1) are complementary, not redundant.
   - **Gravatar/WordPress.com's current model is one account per email
     address** — there's no "add multiple emails under one account with
     different avatars each" for this purpose, contrary to what you might
     expect. Budget one signup flow per address.
   - **Expect anti-abuse rate limiting** if you create more than one
     account back-to-back from the same browser/IP: after roughly the 2nd
     consecutive signup you may get "you can't create a new account at this
     time... disable any VPN". This is a genuine anti-bot measure, not a
     bug — wait and retry later, or spread the signups across sessions. One
     verified address with the right avatar is a reasonable place to stop
     if the remaining ones keep getting blocked; it's a nice-to-have across
     less-common mail clients, not a functional requirement.

## The "real" verified-logo standard: BIMI (know when to skip it)

BIMI (Brand Indicators for Message Identification) is the standard that
shows a *verified* logo natively next to your messages in Gmail, Yahoo, and
Apple Mail — stronger than either of the above, but has real prerequisites:

- **DMARC enforcement** (`p=quarantine` or `p=reject`, not the default
  `p=none`) — meaning your SPF/DKIM alignment must already be solid for
  everything you send, or enforcement mode starts rejecting/quarantining
  your own legitimate mail.
- A hosted square SVG logo referenced by a BIMI DNS record.
- For the logo to render in **Gmail specifically**, a **Verified Mark
  Certificate** from an authorized certificate authority — a paid product,
  and in most cases requires an officially registered trademark for your
  logo.

**Skip BIMI until** you're past the free-relay/fallback-provider stage and
onto a properly authenticated dedicated sending domain with confirmed clean
SPF/DKIM alignment. Attempting it earlier just means paying for a
certificate you can't fully use yet, or flipping DMARC to enforcement mode
before you're ready and breaking your own mail.

## Quick check before you start

```text
dig TXT _dmarc.yourdomain.com     (or: nslookup -type=TXT _dmarc.yourdomain.com)
```

If this returns nothing, you have no DMARC record at all yet — confirms
BIMI is out of reach for now and tells you where `p=none` should be added
first (see `06-email-deliverability-and-spam.md`).
