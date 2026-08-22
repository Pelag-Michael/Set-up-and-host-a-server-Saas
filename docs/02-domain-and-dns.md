# 2. Domain & DNS Setup

## Pick a DNS provider that also gives you a tunnel + email routing

Using one provider (e.g. Cloudflare) for DNS, the tunnel, and email routing
means fewer accounts, fewer places secrets can leak from, and DNS changes
for email propagate instantly since it's the same authoritative nameserver
you already control. This guide assumes that shape; adapt if you're
splitting providers.

## Step order

1. **Buy/point the domain** at the provider's nameservers.
2. **Set up the tunnel** so a process on your machine can expose local ports
   under your domain without opening router ports:
   - Install the tunnel daemon, authenticate it to your account.
   - Create a named tunnel, get a tunnel ID + credentials file (treat this
     credentials file like a secret — see `08-security-and-secrets.md`).
   - Write a tunnel config mapping public hostnames to local ports, e.g.:
     ```yaml
     tunnel: <tunnel-id>
     credentials-file: /path/to/<tunnel-id>.json
     ingress:
       - hostname: api.yourdomain.com
         service: http://127.0.0.1:8001
       - hostname: yourdomain.com
         service: http://127.0.0.1:8088
       - hostname: www.yourdomain.com
         service: http://127.0.0.1:8088
       - service: http_status:404
     ```
   - Add the DNS CNAME/records the tunnel setup tool tells you to add
     (usually automatic if you use the provider's CLI).
3. **Set up Email Routing** (receive-only, free on most DNS providers) for
   your domain *now*, even before you need to send anything — you'll want a
   real inbox behind `support@yourdomain.com` etc. before your first user
   emails you. This adds:
   - An MX record (or several) pointing at the provider's mail routing
     servers.
   - An SPF TXT record authorizing the provider's routing servers to
     receive on your behalf: `v=spf1 include:_spf.<provider>.net ~all`.
   - A DKIM TXT record for the routing provider's own signing.
   - **Do not delete these** later when you set up *sending* — the sending
     setup adds to this record set, it doesn't replace it.
4. **Add a destination address and routing rules**: pick one real inbox
   (a Gmail/Workspace account you actually check) as the destination, verify
   it, then create routing rules like `support@yourdomain.com → that inbox`,
   one per public-facing address you want. Leave catch-all **disabled** —
   letting every random `xyz@yourdomain.com` guess through is just spam
   surface area for no benefit.

## A concrete address plan (steal this)

| Address | Purpose | Shown publicly? |
|---|---|---|
| `support@yourdomain.com` | User-facing help | Yes |
| `hello@yourdomain.com` (or a founder-name alias) | Business/press/legal | Yes |
| `noreply@yourdomain.com` | Transactional email *from* your product | No — never put this in a footer, it's a send-only identity |

Three addresses, all forwarding to one real inbox you actually watch, is
enough for a solo/small project. Add more only when you have a concrete
reason (e.g. a dedicated `security@` for a disclosure policy).

## Gotchas

- **Email Routing has no mailbox of its own** — it only forwards. If your
  destination inbox account is ever deleted or its forwarding revoked, mail
  silently stops arriving. Check it occasionally.
- **A domain used for Email Routing "receive" cannot also be blindly reused
  for a different provider's receive setup** — but *can* be layered with a
  send-only provider (see `05-transactional-email-setup.md`) since sending
  and receiving use different DNS record types (SPF/DKIM entries can
  coexist via multiple `include:` mechanisms and unique DKIM selector
  names).
- Keep a short internal doc (even just a markdown file, not this repo) that
  says "this address does X, forwards to Y, added on Z date" — DNS records
  six months later look like inscrutable noise without that context.
