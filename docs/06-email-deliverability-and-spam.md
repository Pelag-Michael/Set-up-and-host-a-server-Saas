# 6. Email Deliverability & Spam

Getting an SMTP call to return "250 OK" is not the same as the email
arriving in an inbox. This is the part that actually determines whether
your users see your emails at all.

## The three records that decide whether you're "you"

- **SPF** (Sender Policy Framework) — a DNS TXT record listing which mail
  servers are allowed to send *claiming to be* your domain. Receiving
  servers check the connecting IP against this list.
- **DKIM** (DomainKeys Identified Mail) — a cryptographic signature added to
  outgoing mail, verified against a public key published in DNS. Proves the
  message wasn't altered in transit and really came from a system that has
  your private key.
- **DMARC** — a policy record telling receiving servers what to do if SPF
  and DKIM *both* fail alignment (`p=none` = do nothing, just report;
  `p=quarantine` = spam-folder it; `p=reject` = bounce it). Start at
  `p=none` — it's monitoring-only and cannot break existing mail flow, and
  is a prerequisite for advanced features (like a verified logo, see
  `07-branding-icons-avatars.md`). Only move to `quarantine`/`reject` once
  you've confirmed SPF/DKIM pass cleanly for everything you actually send.

## Multiple senders on one domain

Your domain can be authenticated for several different sending sources at
once — a receive-only routing provider, a personal-inbox relay, and a
dedicated ESP fallback can all coexist, because:
- SPF supports multiple sources in one record via repeated `include:`
  mechanisms: `v=spf1 include:_spf.providerA.net include:_spf.providerB.com ~all`.
- DKIM uses per-source *selectors* (the first label of the DNS record name),
  so provider A's `providerA._domainkey.yourdomain.com` and provider B's
  `providerB1._domainkey.yourdomain.com` / `providerB2._domainkey.yourdomain.com`
  don't conflict.

**Adding a new sender is additive** — you're adding new DNS records, not
replacing existing ones. Never delete an existing MX/SPF/DKIM record when
setting up a new sender unless you've confirmed nothing still depends on it.

## The gap nobody warns you about: personal-inbox relays

If you're sending "as" your domain through a personal email provider's own
SMTP relay (see `05-transactional-email-setup.md` option A), the mail is
authenticated as coming from *that provider's* infrastructure, not natively
from your domain's own SPF/DKIM setup — because you're using their servers,
signed with their DKIM selector, not your own. This means:
- SPF alignment for your domain will likely **fail** for this path, unless
  you also add that provider's SPF include to your domain's record.
- DKIM will never be domain-aligned this way on a free consumer account —
  custom DKIM signing tied to your own domain typically requires a paid
  business tier from that provider, not the free consumer version.
- **Net effect**: mail sent this way is more likely to land in spam,
  especially at any real volume, than mail sent through a properly
  domain-authenticated dedicated ESP. It's fine to start here (zero
  deliverability infrastructure needed) but plan to move volume to an
  authenticated sender as you grow.

## Domain authentication with a dedicated ESP

Most providers give you an in-dashboard flow: enter your domain, choose
"I'll add the DNS records myself" (safer than granting the provider direct
API access to your DNS account), copy the 3-4 records shown (typically: one
ownership-verification TXT, two DKIM CNAMEs, one DMARC TXT), add them to
your DNS provider, then click "verify" in their dashboard. Propagation is
usually fast (minutes) when your DNS provider is also your nameserver
(no external propagation delay), but the provider's UI may say "up to 48
hours" as a conservative default — try verifying after a couple of minutes
first.

**Important**: when adding a DKIM CNAME record on a DNS provider that also
proxies traffic (like Cloudflare's orange-cloud proxy), the record's proxy
status must be **DNS only** (grey cloud), not proxied. A proxied CNAME
resolves through the provider's edge network instead of acting as a plain
DNS alias, which breaks DKIM signature verification for receiving mail
servers doing their own DNS lookup.

## Sending caps and safety margins

Every provider — free personal relay or paid ESP — has a sending cap
(daily or monthly). Two failure modes to design around:

1. **Hitting the cap mid-day** — see the fallback pattern in
   `05-transactional-email-setup.md`. Track your own send count and switch
   proactively at a safety margin below the actual cap, not after a bounce.
2. **A sudden burst looking like abuse** — if your public-facing form can
   trigger a send (e.g. "email me a signup code"), a bot hammering that form
   can spike your send volume in a way that looks like spam to the
   provider's own abuse detection, independent of your daily cap. Rate-limit
   the *triggering endpoint* per IP and per identity (e.g. one code per
   email address, ever, not per request) — see `08-security-and-secrets.md`
   for where this check belongs.

## If mail is landing in spam right now

Check, in order:
1. Does the DMARC record exist at all? (`p=none` still gives you visibility
   via aggregate reports even if it's not enforcing.)
2. Does the *specific path* that sent this message pass SPF and DKIM? (Not
   "does the domain have SPF/DKIM" in general — a specific sender/selector
   can be misconfigured while others are fine.)
3. Is this the very first email from a brand-new sending identity? Some
   inbox providers apply extra scrutiny to mail from a sender/domain
   combination they've never seen before — this typically self-resolves
   after a small number of legitimately-opened messages ("reputation
   warm-up"), not something you can fix instantly.
4. Only after 1-3 are ruled out: consider content-based spam triggers
   (excessive links, ALL-CAPS subject lines, spam-trigger words) — this is
   rarely the actual cause when the real cause is 1-3.
