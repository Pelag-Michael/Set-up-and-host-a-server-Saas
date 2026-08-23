# Setup Checklist

A linear, tickable version of the whole guide. See the linked doc for the
"why" behind any step that isn't self-explanatory.

## 1. Domain & DNS — [`docs/02-domain-and-dns.md`](docs/02-domain-and-dns.md)

- [ ] Domain pointed at your DNS provider's nameservers
- [ ] Tunnel daemon installed, authenticated, named tunnel created
- [ ] Tunnel config maps public hostnames → local ports
- [ ] DNS records for the tunnel added
- [ ] Email Routing enabled (receive-only) for the domain
- [ ] MX + SPF + DKIM records for email routing present
- [ ] Destination inbox added and verified
- [ ] Routing rules created for each public address (support@, etc.)
- [ ] Catch-all left disabled

## 2. Backend API server — [`docs/03-backend-api-server.md`](docs/03-backend-api-server.md)

- [ ] Server process bound to `127.0.0.1` only, one port
- [ ] `.env` loader in place, config comes from env vars not hardcoded values
- [ ] Database schema-sync function is additive-only (`CREATE TABLE IF NOT
      EXISTS`, `ALTER TABLE ADD COLUMN` guarded by a column-existence check)
- [ ] Signing key pair generated if you need stateless auth tokens; private
      key path is in `.env`, key file itself is gitignored
- [ ] Any internal-only endpoint guarded by shared-secret header **and**
      loopback-only check
- [ ] Admin tooling is a separate process/port, bound to `127.0.0.1`, never
      referenced in the tunnel config

## 3. Frontend hosting — [`docs/04-frontend-hosting.md`](docs/04-frontend-hosting.md)

- [ ] Confirmed which build output directory is actually served (check the
      running server's document root, don't assume)
- [ ] If PHP built-in server: extensions loaded explicitly via `-d` flags
- [ ] Router script (if needed) serves static files directly, falls back to
      app entrypoint otherwise
- [ ] Cache-busting plan for hashed bundle filenames confirmed

## 4. Outbound email — [`docs/05-transactional-email-setup.md`](docs/05-transactional-email-setup.md)

- [ ] 2FA enabled on the mailbox account used for primary relay
- [ ] App Password generated, saved to `.env` **and** a password manager
- [ ] Domain alias added as "send mail as", manually pointed at the real
      outbound SMTP host (not an auto-detected MX)
- [ ] Alias ownership verified (confirmation email clicked)
- [ ] One real test email sent via SMTP from a script, confirmed delivered
- [ ] Fallback provider account created (phone-verified, real address
      entered, free plan selected)
- [ ] Fallback provider SMTP credentials generated, saved to `.env` + backup
- [ ] Fallback pattern implemented: proactive switch at a safety margin,
      one-time admin alert sent via the fallback channel itself

## 5. Deliverability — [`docs/06-email-deliverability-and-spam.md`](docs/06-email-deliverability-and-spam.md)

- [ ] DMARC record added at `p=none` (monitoring only) if not already present
- [ ] Fallback provider's domain authentication completed (verification TXT
      + DKIM CNAMEs + DMARC, all as **DNS-only**, not proxied)
- [ ] Confirmed a real send via the fallback provider lands in inbox, not spam
- [ ] Rate limits in place on any endpoint that can trigger a send (per
      identity, per IP)

## 6. Branding — [`docs/07-branding-icons-avatars.md`](docs/07-branding-icons-avatars.md)

- [ ] Favicon is the real brand mark (not a placeholder), rendered on your
      actual theme background color
- [ ] Google Account profile photo of the primary mailbox set to the brand
      mark
- [ ] Gravatar set up for at least your main public-facing address
- [ ] (Optional, later) BIMI only attempted once DMARC enforcement + SPF/DKIM
      alignment are solid

## 7. Automation — [`docs/09-automation-and-scheduling.md`](docs/09-automation-and-scheduling.md)

- [ ] Start-on-boot script covers every process in the stack, checks
      already-running state before starting anything
- [ ] Start-on-boot script gives visible confirmation even when nothing
      needed starting (don't let "already running" look like "broken")
- [ ] Any recurring job (reminders, cleanup) has a scheduled trigger and is
      idempotent (safe to run twice the same day)
- [ ] Background process output redirected to a gitignored `logs/` directory

## 8. Security pass — [`docs/08-security-and-secrets.md`](docs/08-security-and-secrets.md)

- [ ] `.env`, key files, and real DB files all in `.gitignore`
- [ ] `.env.example` committed with variable names + placeholder values only
- [ ] Every generated secret backed up outside this one machine
- [ ] `git status` / `git diff --cached` actually reviewed before every
      commit, especially after a broad `git add`

## 9. Testing habits — [`docs/10-testing-and-safety-practices.md`](docs/10-testing-and-safety-practices.md)

- [ ] Every automated test runs against an isolated temp DB, not real data
- [ ] Real external side effects (email sends, third-party API calls) are
      mocked in tests, asserted on their actual call arguments
- [ ] Fallback/failure paths have explicit tests, not just the happy path
- [ ] Dedup/guard logic has a test that calls twice and asserts the second
      call is a no-op
- [ ] Any one-off real-system test used clearly-marked throwaway data,
      cleaned up immediately after

## 10. Discovery — [`docs/12-discovery-and-community-marketing.md`](docs/12-discovery-and-community-marketing.md)

- [ ] `llms.txt` published (only if it takes under an hour — it's hygiene,
      not a growth channel) and confirmed reachable (`curl -I` returns 200)
- [ ] `robots.txt` does not disallow `/llms.txt`
- [ ] No confidential/unreleased info, secrets, or model-directed
      instructions inside `llms.txt`
- [ ] Picked at most 2-3 channels to actively run, not spread across
      everything
- [ ] Target communities identified, their self-promotion rules read,
      before posting anything
- [ ] No public community space (Discord/forum/etc.) stood up before there
      are real active users for it

## 11. SEO and getting indexed — [`docs/13-seo-and-search-console.md`](docs/13-seo-and-search-console.md)

- [ ] Checked for an accidental `noindex` meta tag before doing any other
      SEO work
- [ ] Title/description/canonical/OG/Twitter Card tags present on real pages
- [ ] `robots.txt` and `sitemap.xml` exist and are reachable
- [ ] If domain is behind Cloudflare: checked the *actual served*
      `robots.txt` (not just the file on disk) for an injected AI-crawler
      block from "Managed robots.txt" — decided deliberately whether to
      keep or disable it
- [ ] Google Search Console property added and verified
- [ ] Sitemap submitted and homepage indexing explicitly requested (not just
      waiting for organic crawl discovery)

## 12. Ship

- [ ] Full test suite passing
- [ ] Service restarted with the final code, health check green
- [ ] One real end-to-end pass through the riskiest path (the one touching
      email + a real credential) confirmed working, then cleaned up
- [ ] Docs updated to describe what's actually running, not what was
      originally planned
