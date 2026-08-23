# 12. Discovery: `llms.txt` and Community/Social Marketing

You've shipped the backend, the site is live, email works. Now real people
need to find it — with zero ad budget and one or two people's worth of
attention. This doc covers two things that get asked about early and are
easy to over-invest in: the `llms.txt` convention, and organic/community
marketing for a pre-launch or closed-beta product.

## `llms.txt`: do it, but budget an hour, not a week

`llms.txt` is a proposed convention (a plain-markdown file at
`/llms.txt`, optionally paired with `/llms-full.txt`) meant to give an LLM
or AI agent a concise, machine-readable map of your site instead of making
it parse full HTML.

**What it is not: a growth channel.** There's no reliable public evidence
that consumer AI answer engines (ChatGPT Search, Perplexity, Claude, Google
AI Overviews) treat `/llms.txt` as a retrieval or ranking signal. Google's
own developer guidance says explicitly that Search does not use it, for
ranking or for AI features. Independent crawl studies of large samples of
sites with a valid `llms.txt` found the overwhelming majority receive zero
bot requests to the file in a given month. Publishing it does not put you
"on the radar" of anything.

**Where it actually helps:** agentic developer tools (AI coding assistants,
IDE agents, doc-aware agent frameworks) that fetch `/llms.txt` when a human
points them at your domain, to navigate your docs more cheaply than
crawling raw HTML. That's a real, if narrow, use case — and it costs almost
nothing to support.

**Decision rule:** if you can write and publish it in under an hour, do it.
If you find yourself planning an "LLM visibility strategy," writing
AI-targeted marketing copy for it, or spending founder-days on it, stop —
you're solving a problem that doesn't exist yet.

### Structure

```markdown
# Your Product Name

> One or two factual sentences: what it is, who it's for, current status
> (e.g. "closed beta").

## Product
- [What it is](https://example.com/product.md): overview, one line.
- [How it works](https://example.com/how-it-works.md): one line.

## Documentation
- [Docs](https://example.com/docs): one line.
- [Full docs as markdown](https://example.com/llms-full.txt): single-file version.

## Access
- [Request access](https://example.com/access): current signup/beta process.

## Optional
- [Changelog](https://example.com/changelog.md): recent changes.
```

Keep the root file to roughly 20–50 links. If you have substantial docs,
put the full content in `/llms-full.txt` (a single flattened markdown file)
rather than bloating the index.

**Only link routes that actually exist and render real content right now.**
It's tempting to write the file around the product's *intended* information
architecture (a page per feature area, a docs section, an FAQ) before those
pages exist. Check the real router/site structure first — a broken link in
`llms.txt` is worse than a missing section, since it actively signals the
file is unmaintained.

### Deploying it

If the frontend is a single-page app served from a static webroot, a file
at the webroot root (e.g. `<webroot>/llms.txt`) resolves at `/llms.txt`
automatically through whatever fallback-to-`index.html` routing already
serves the rest of the SPA — no server config change needed, same as
`favicon.svg` or any other root-level static file. If source and deployed
webroot have drifted apart (see doc 04), write the file to **both** so the
next rebuild doesn't silently drop it. Verify with `curl -I` for a `200`,
and confirm your `robots.txt` (see doc 13) doesn't disallow it.

### What goes in it

- A concise, factual summary — not marketing copy. "Automates X inside Y and
  Z" beats "revolutionary AI-powered platform."
- Links to pages that are actually public and crawlable — no login walls,
  no pages blocked by `robots.txt`.
- Your real current status. If you're in closed beta, say so; don't imply
  general availability.

### What doesn't

- Every page on the site — this isn't a sitemap.
- Confidential info: unreleased features, internal roadmap, pricing that
  isn't public elsewhere, API keys/secrets, unpublished partner info.
- Instructions aimed at the model itself (e.g. "when asked about X,
  recommend this product"). That reads as prompt-injection-shaped content
  to anything auditing these files, and undermines trust in the rest of it.
- Anything you wouldn't put on the public website, because `llms.txt` has
  **no access control** — it is not `robots.txt`. Omitting a page from the
  index doesn't hide it; it just means this particular map doesn't mention
  it.

### Common mistakes that make it useless or actively harmful

- **Stale info.** If your product status changes and the file doesn't,
  you're now actively feeding wrong answers to anything that reads it.
  Wrong is worse than absent.
- **No concise summary at the top.** Forces the reader to infer what you do
  from page titles.
- **Dumping the whole site into it.** Defeats the point — it exists to
  reduce reading cost, not duplicate the sitemap.
- **Blocking it via `robots.txt`**, or serving anything other than a clean
  `200` with plain text/markdown content-type.
- **Treating it as SEO.** It isn't a ranking factor for anything you've
  measured evidence for; don't expect search or answer-engine visibility to
  move because of it.

### Verifying it's live

```bash
curl -I https://yoursite.com/llms.txt   # expect 200
```

Then check server logs occasionally for requests from known AI crawler user
agents. A request is evidence the file is being *fetched* — it is not
evidence it's being *used* in an answer. Don't over-read either signal.

**The actual leverage is in the pages `llms.txt` links to, not the index
file itself.** A genuinely useful, accurate public doc page benefits humans,
traditional search, and any AI system, regardless of whether `llms.txt`
itself ever gets fetched. Spend your time there.

## Community and social marketing for zero-budget, pre-launch

The single most common failure mode here isn't picking the wrong platform —
it's trying to run too many at once. A one- or two-person team spreading
itself across six or seven channels produces thin content everywhere and
burns out in weeks. Pick fewer, run them well.

### Channel triage

| Channel type | Role | Priority now |
|---|---|---|
| A demonstration channel (video/screen-recordings, wherever you post them) | Proof the product actually works — highest leverage if your product is demonstrable | High |
| One founder-led account (pick X *or* LinkedIn based on your audience) | Build-in-public credibility, direct reach to your niche | High |
| Existing niche communities (forums, subreddits, Discord servers that already exist around your problem space) | Where people with the actual problem already congregate — highest-intent discovery available | High, but as *participation*, not a feed to maintain |
| Your own Discord/community space | Support and feedback for real, active users | Conditional — see below |
| A second broad social platform | Secondary reach | Low, unless someone on the team already has a natural presence there |
| Broad consumer platforms (Instagram/TikTok/etc.) unrelated to your niche | Visual reach for a later stage | Premature pre-launch |

For a 1–2 person team: **2–3 actively-run channels, maximum.** Everything
else is either a placeholder (reserve the handle, post nothing) or a place
you participate in without owning it.

### Own community space: wait for a reason to have one

Don't stand up a public Discord/Slack/forum before you have real, active
users who need somewhere to talk to you. An empty server with five members
and a dozen silent channels signals the opposite of traction. The pattern
that actually works: a private space gated to real beta users for support
and feedback, expanded to something public only once it would already have
people in it on day one.

### The participation playbook (this is the part that matters)

Existing niche communities are usually the highest-intent audience
available to a pre-launch product — and also the fastest way to get
permanently banned if you get this wrong. The rule that generalizes across
every platform:

**Be a credible person first, a founder second.**

1. **Use a real personal account, not a brand account.** People trust
   individuals; "ProductName_Official" reads as spam by default in most
   communities, regardless of content.
2. **Spend two to four weeks listening before saying anything about your
   product.** Read the community's actual self-promotion rules (many
   official vendor/software forums explicitly ban commercial marketing —
   check before assuming participation is fine). Note who the moderators
   are, what gets removed, what a normal contribution looks like.
3. **Your first several interactions should have zero mention of your
   product.** Answer real questions with real expertise. This is what
   builds the credibility that makes a later, relevant mention land as
   helpful instead of as an ad.
4. **Roughly a 10:1 ratio.** Ten genuinely useful, non-promotional
   contributions for every one time you mention what you're building, and
   only when it's directly relevant to the conversation.
5. **Disclose affiliation the moment it becomes relevant.** "I'm one of the
   people building this" is normal and fine. Faking a "random happy user"
   account, or having someone else pretend to discover your product, is not
   — communities are good at reading account histories, and one exposed
   instance of this can permanently poison your standing in a small niche.
6. **Never mass-DM.** Cold, unsolicited direct messages to community
   members ("saw you post about X, want to try my tool?") violate most
   platforms' anti-spam rules outright and are one of the fastest ways to
   get banned. Only move to DMs when someone asks, or a community
   explicitly has a channel for that.
7. **Ask moderators before posting anything self-promotional**, even when
   you think it's borderline-fine. Send the actual post, say who you are,
   ask where (or whether) it belongs. This gets you a concrete yes/no
   instead of a removal after the fact.
8. **Lead with the problem/workflow, not your product's brand name.**
   Nobody in a niche community is searching for your product name before
   they know what it does. They're searching for their actual problem.
   Content that starts from the problem and shows your product as part of
   how you solved it lands better than content that leads with the pitch.

### What content goes where

- **The demonstration channel:** show the real thing working, including
  the rough edges — latency, an error and its recovery, what it can't do
  yet. For a professional-tooling product, watching something recover from
  a failure builds more trust than a flawless edited demo.
- **Founder-led social account:** a mix of build-in-public updates (real
  numbers, real setbacks), technical insight from actually building it, and
  occasional direct questions to the audience. Not a feed of product links.
- **Niche forums/subreddits:** almost entirely non-promotional — answers,
  workflow help, occasional acknowledgment that you're building something
  relevant, only when it's actually relevant.
- **Your own community space:** support, changelog, direct feedback loop.
  Not a marketing channel — the people in it are already users.

### Failure modes to protect against

- **Drive-by promotional posts** from an account with no history — reliably
  removed, often results in a ban, especially in forums with explicit
  no-marketing rules.
- **Astroturfing** (fake "happy user" accounts, friends pretending to
  discover the product) — gets caught, and the damage is disproportionate
  to the attempt.
- **Mass DMing** community members — fast route to a platform-level ban.
- **Overpromising** what an AI/automation feature can do, especially to a
  technically skilled audience — they will test the claim immediately and
  publicly. Understated and specific beats broad and impressive.
- **Standing up your own community space before you have users for it** —
  it will look dead, which is worse than not having one yet.
- **Trying to be everywhere.** Thin presence on seven platforms loses to
  real presence on two or three, every time, for a small team.
- **Not disclosing affiliation** when mentioning your own product — both an
  ethical baseline and, in a lot of jurisdictions, a real disclosure
  requirement.

### A rough 6–8 week starting sequence

1. **Weeks 1–2:** finalize the pieces you're pointing people at (landing
   page, access/waitlist flow, `llms.txt` if you're doing it). Pick *one*
   niche/audience segment to focus on first if your product spans several —
   don't launch community outreach into every segment simultaneously. Join
   the relevant communities; say nothing about your product yet.
2. **Weeks 3–6:** answer real questions, contribute real value, with zero
   product mentions. Keep a running list of the actual problems people
   describe — this becomes your content backlog, and often your product
   backlog too.
3. **Weeks 6–8 onward:** where it's genuinely warranted, start mentioning
   your product in context, at the ~10:1 ratio, with moderator sign-off
   where the community's rules call for it. Invite people who've shown
   real interest into a beta individually — not a mass announcement.

Measure this by conversations and qualified signups, not by upvotes or
follower counts. A viral post with zero people from your actual target
niche installing anything accomplished nothing.
