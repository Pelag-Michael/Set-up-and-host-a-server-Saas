# 5. Transactional Email Setup

This is the step that eats the most real wall-clock time, because it
involves account verification steps you can't script around: 2FA setup,
phone verification, and DNS propagation. Budget an afternoon, not 20 minutes.

## Decide: personal-inbox relay first, dedicated ESP later

Two viable starting points:

**A) Free personal-inbox SMTP relay** (e.g. Gmail's SMTP as your outbound
relay, sending "as" your domain address). Zero cost, works immediately once
set up, but:
- Capped at a few hundred recipients/24h (varies by provider/account type).
- Deliverability is weaker: the mail is authenticated as coming from the
  personal inbox provider, not natively from your domain, unless you also
  do the domain-alignment work in `06-email-deliverability-and-spam.md`.

**B) A dedicated transactional email provider** (Brevo, SendGrid, Postmark,
Resend, Mailgun, etc.), most of which have a genuinely free tier (hundreds
of emails/day, no credit card). Proper domain authentication support built
in, better deliverability from day one, but:
- Usually requires phone verification before you can generate API/SMTP
  credentials (anti-abuse measure — expect this, it's not specific to one
  provider).
- Requires a real physical address for your account (anti-spam law
  requirement — CAN-SPAM-style rules require a physical address in
  commercial email footers; the provider often collects this at signup even
  though it's not always rendered visibly).

**Recommended path for a small project**: start with (A) for zero cost and
instant setup, add (B) as an automatic fallback once (A)'s cap starts to
matter (see the fallback pattern below) — you get the best of both without
paying anything until you actually need the second provider's higher volume.

## Setting up (A): personal-inbox SMTP relay

1. **Enable 2-Step Verification** on the account. App-specific passwords
   (needed for SMTP login from a script) don't exist until 2FA is on.
2. **Generate an App Password**, scoped/named for this purpose specifically
   (not reused from something else) so you can revoke it independently
   later.
3. **Add your domain address as a "send mail as" alias** in the account's
   settings. The provider will offer to auto-detect an SMTP server from your
   domain's MX record — **decline that auto-detected value**, it'll be your
   *receiving* mail router, not a sending relay. Manually enter the
   provider's own outbound SMTP host (e.g. `smtp.gmail.com:587`), with the
   account's own username + the App Password from step 2.
4. **Verify ownership**: the provider sends a confirmation email to the
   alias address. Since (per `02-domain-and-dns.md`) that address forwards
   to this same inbox, you can find and click it there.
5. **Test-send one real email** via SMTP from a script before wiring it into
   your app. If it doesn't arrive, check spam before assuming it failed —
   see `06-email-deliverability-and-spam.md`.

## Setting up (B): dedicated provider as fallback

1. Sign up (using "Sign up with Google" against the same inbox account you
   already control saves a separate email-verification round-trip).
2. Fill in the required organization details, **including a real address**
   — home address is fine for a solo project, this just satisfies the
   anti-spam disclosure requirement.
3. Pick the free plan explicitly (some flows default-select a paid tier;
   look for a "free forever, no card" option).
4. **Verify your phone number** — required before the dashboard will let you
   generate SMTP credentials or an API key at all. This is normal, not a
   sign something's wrong.
5. Generate an SMTP key (not always the same thing as a general API key —
   check the provider's docs for which credential type their SMTP relay
   expects).
6. **Authenticate your domain** (see `06-email-deliverability-and-spam.md`)
   before relying on this as anything other than a last-resort fallback —
   an unauthenticated sending domain gets flagged immediately by most
   providers ("sender not valid, authenticate your domain").
7. Add the domain as a verified sender too if the provider distinguishes
   "sender verification" from "domain authentication" (some require both).

## The fallback pattern (pseudocode)

```text
send(to, subject, body):
    if today's primary-relay send count < safety margin (e.g. 480 of a 500 cap):
        try primary relay
        on success: record it, done
        on failure: fall through (don't retry primary — assume exhausted for today)
    send via fallback provider
    if this is the first fallback use today:
        send ONE alert to the admin, via the fallback provider
        (not via the primary — it's the one that just failed)
```

Key details that matter:
- **Track your own send count** rather than waiting for a hard bounce —
  switching proactively at a safety margin (not the literal cap) avoids a
  failed delivery attempt on the exact request that pushes you over.
- **The admin alert must go out through the channel that's still working**,
  not the one that just failed — alerting through the broken channel
  silently no-ops exactly when the alert matters most.
- **Rate-limit the alert itself** to once per day (or per cap-hit event) —
  otherwise every subsequent send that day re-triggers it.
