# Set Up & Host a Server SaaS — A Practical Guide

A checklist-driven guide for standing up a small self-hosted SaaS backend
end-to-end: domain + DNS, a Python API server, a PHP/static frontend,
transactional email (with a free-tier fallback provider), branding/icons for
the accounts involved, automation, and the security/testing habits that keep
you from shooting yourself in the foot.

**No code and no credentials live in this repo.** It's the accumulated
"here's what to actually do, in what order, and here's what will trip you
up" from building and hosting one real system this way. Adapt the specifics
to your own stack — the sequence and the gotchas are the reusable part.

## Who this is for

You're one person (or a very small team) hosting your own backend — not a
platform team with a dedicated SRE. You're using free/cheap tools (Cloudflare
free tier, a personal Gmail account, a free-tier email API) instead of
enterprise infrastructure, and you need it to actually be reliable and not
leak secrets or get your mail flagged as spam.

## How this guide is organized

| # | File | Covers |
|---|---|---|
| 1 | [`docs/01-architecture-overview.md`](docs/01-architecture-overview.md) | The shape of the whole system, why each piece exists |
| 2 | [`docs/02-domain-and-dns.md`](docs/02-domain-and-dns.md) | Buying/pointing a domain, Cloudflare setup, tunnels |
| 3 | [`docs/03-backend-api-server.md`](docs/03-backend-api-server.md) | The API server itself: process model, DB, secrets, internal-only endpoints |
| 4 | [`docs/04-frontend-hosting.md`](docs/04-frontend-hosting.md) | Serving a frontend (static/PHP), cache-busting, build vs. hand-edit tradeoffs |
| 5 | [`docs/05-transactional-email-setup.md`](docs/05-transactional-email-setup.md) | Getting outbound email working: primary relay + fallback provider |
| 6 | [`docs/06-email-deliverability-and-spam.md`](docs/06-email-deliverability-and-spam.md) | SPF/DKIM/DMARC, sending caps, why mail lands in spam |
| 7 | [`docs/07-branding-icons-avatars.md`](docs/07-branding-icons-avatars.md) | Getting your logo to actually show up next to your emails |
| 8 | [`docs/08-security-and-secrets.md`](docs/08-security-and-secrets.md) | Where secrets live, how internal endpoints are guarded, what never gets committed |
| 9 | [`docs/09-automation-and-scheduling.md`](docs/09-automation-and-scheduling.md) | Autostart on boot, daily/recurring jobs, without a real ops platform |
| 10 | [`docs/10-testing-and-safety-practices.md`](docs/10-testing-and-safety-practices.md) | Never test against production data — how to actually enforce that on yourself |
| 11 | [`docs/11-pitfalls-and-lessons-learned.md`](docs/11-pitfalls-and-lessons-learned.md) | Every specific thing that broke, and why, in one place |
| 12 | [`docs/12-discovery-and-community-marketing.md`](docs/12-discovery-and-community-marketing.md) | `llms.txt` (what it's actually worth), and a zero-budget community/social playbook for a tiny team |
| 13 | [`docs/13-seo-and-search-console.md`](docs/13-seo-and-search-console.md) | On-page SEO, indexing, entity/name-collision, brand vs search modifier, SPA head/H1, JSON-LD `@graph`, Search Console that actually succeeds |
| 14 | [`docs/14-brand-accounts-and-social-search.md`](docs/14-brand-accounts-and-social-search.md) | Official brand accounts (X/LinkedIn/IG/Facebook): identity, handles, assets, in-app search vs feed, two-way links with the site |
| 15 | [`docs/15-brand-posting-style-and-image-assets.md`](docs/15-brand-posting-style-and-image-assets.md) | After creation: bio/cross-link conventions, a cross-platform posting-style guide, and why a banner looks sharp only when clicked |

There's also [`CHECKLIST.md`](CHECKLIST.md) — a linear, tickable version of
the whole setup if you just want the steps without the explanations.

## Order of operations (short version)

1. Domain + DNS (Cloudflare) → 2. Backend API server skeleton + local DB →
3. Frontend hosting → 4. Get outbound email working (primary relay) →
5. Add a fallback email provider + authenticate your domain for it →
6. Branding/icons → 7. Automation (autostart, scheduled jobs) →
8. Security pass (secrets, internal-endpoint guards) → 9. Testing habits →
10. Ship, watch logs, iterate.

Email (step 4-5) is deceptively the long pole — expect account verification,
2FA setup, phone verification, and DNS propagation to eat real wall-clock
time before your first test email lands anywhere but spam.
