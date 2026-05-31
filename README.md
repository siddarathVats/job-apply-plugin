# Job Apply Plugin for Claude Code

[![Discord](https://img.shields.io/badge/Discord-Join%20Server-7289da?style=flat&logo=discord&logoColor=white)](https://discord.gg/7xsxU4ZG6A)

AI-powered job application assistant. Auto-fills applications on LinkedIn Easy Apply, Greenhouse, Ashby, Lever, Rippling, and Workday — and goes further: discovers hiring at recently-funded startups, drafts cold outreach to founders/CXOs, and tracks every send.

## Skills

| Skill | Description |
|-------|-------------|
| `/job-apply` | Fill out job applications automatically using your resume |
| `/job-search` | Search LinkedIn, HN Who's Hiring, and Twitter/X for jobs with connections + hiring-manager insights |
| `/job-preferences` | Set target titles, salary floor, remote preference, exclusion patterns (used by other skills) |
| `/funded-startup-jobs` | Mine VC LinkedIn feeds, VC job boards, LinkedIn funding-news search, Crunchbase, and user-pasted posts for hiring at recently-funded startups |
| `/feed-scan` | Timeboxed scroll of your LinkedIn home feed (default 20 min) — classifies posts as job-opening or partnership opportunity, scores against your preferences, hands off to `/cold-outreach` |
| `/cold-outreach` | Draft a personalized LinkedIn DM + email to a founder/CXO from your profile. Handles both job and partnership outreach. Optional Apify email lookup. Auto-sends only after explicit confirmation. |
| `/outreach-tracker` | Log sent outreach, list pending follow-ups, mark replies, get reply-rate stats, **sweep stale LinkedIn invites** to protect your invite acceptance rate |

## Features

### Job Apply (`/job-apply`)
- **One-time profile setup**: Extract your information from a resume (PDF, DOCX, or TXT)
- **Multi-platform support**: LinkedIn Easy Apply, Greenhouse, Ashby, Lever, Rippling, Workday
- **Dual-tool architecture**: Chrome MCP for authenticated sites, Playwright MCP for form filling and file uploads
- **Smart field mapping**: Automatically matches your profile to form fields
- **Safety first**: Never submits without your explicit confirmation
- **Resume storage**: Profile saved locally for reuse across applications

### Job Search (`/job-search`)
- **Smart keyword suggestions**: Auto-generates search terms from your resume
- **Connection insights**: Finds jobs at companies where you have connections
- **Hiring manager discovery**: Identifies jobs with hiring managers listed
- **Auto-inferred filters**: Location and experience level from your profile
- **Results saved**: All searches saved to `~/.claude-job-searches/` as JSON

### Funded-Startup Jobs (`/funded-startup-jobs`)
- **Five signal sources**: VC LinkedIn company feeds, VC portfolio job boards, LinkedIn funding-news content search, Crunchbase/TechCrunch via WebSearch, and **user-pasted LinkedIn post URLs** (fast-path)
- **Curated VC list**: a16z, Sequoia, Greylock, Kleiner, YC, Accel, Contrary, Pear, NEA, Antler, Lightspeed, Bessemer, BITKRAFT, and more — editable in `skills/funded-startup-jobs/vc-sources.md`
- **Dedupe + enrich**: merges duplicates across sources, pulls funding stage / amount / headcount / careers URL
- **Scored**: 0-100 against your `/job-preferences` (title match, recency, funding freshness, remote fit, founder availability)
- **Hand-off**: top results pipe directly into `/cold-outreach`
- **Saved**: `~/.claude-funded-startups/funded-{date}.md`

### Feed Scan (`/feed-scan`)
- **Timeboxed scroll** of `linkedin.com/feed/` — default 20 minutes, configurable via `--time`
- **Dual classification**: every post tagged as `job` (hiring posts), `partnership` (collab / pilot / advisor / co-founder asks), or `both`
- **Connection-degree aware**: 1st/2nd-degree posters score higher (highest reply rates)
- **Dedupe across runs**: `~/.claude-feed-scans/seen-posts.txt` prevents re-surfacing the same post day after day
- **Hand-off**: top results pipe directly into `/cold-outreach`
- **Read-only**: no reactions, comments, follows, or DMs from the feed itself
- **Aborts on security challenges**: any CAPTCHA or rate-limit modal stops the scan cleanly

### Cold Outreach (`/cold-outreach`)
- **Picks your tone**: asks how you want to come across (Warm & curious / Direct / Peer / Formal; defaults to Warm & curious) and threads it through the draft
- **First-touch bedrock rules**: a connection-request note or first DM carries **no portfolio link and no meeting CTA** (both read as rushed on a cold contact), opens pain-first from real research, and closes **warm and curiosity-first** — never on "that's what I build." Links and chat invites are later-step (follow-up email / post-accept DM) moves
- **Two outreach types**: `job` (default) and `partnership` (collab / pilot / advisor asks) — drafts use different templates with type-specific validation
- **Personalized drafts**: pulls slots from your `~/.claude-job-profile.json` and the company/post record — no `{placeholder}` ever ships
- **Two channels**: LinkedIn DM (200-700 chars, four variants A/B/C/D) + cold email (~150-200 words, two variants)
- **Apify-powered email lookup**: best-effort founder/CXO email resolution via Apify MCP actors (fail-soft)
- **Validation pass**: rejects drafts with unresolved slots, anti-patterns, claims unsupported by your profile, or job-pitch keywords in partnership drafts
- **Auto-send with confirmation**: opens LinkedIn DM modal or Gmail compose pre-filled — explicit user `yes` clicks Send
- **One recipient per invocation**: hard cap of 8 cold LinkedIn touches per day (DMs + connect-notes combined) and 15 cold emails per day to avoid flag patterns
- **Suppression rules**: blocks new touches to recipients with pending invites (<14 days) or recent same-channel touches (<7 days); recently-rejected recipients are suppressed for 90 days unless explicitly overridden
- **Anti-AI tells stripped before preview**: em-dashes, "delve/landscape/leverage/synergy/furthermore", desperation phrases ("I need a job", "I would be grateful"), and AI-typical openers/closers are all rewritten silently. Hard-rejects (unsupported claims, compensation talk, generic flattery) surface to the user
- **Logged**: every send appended to `/outreach-tracker`

### Outreach Tracker (`/outreach-tracker`)
- **Single log**: `~/.claude-outreach-log.md` — append-only markdown table, one row per send
- **Status tracking**: sent / replied / meeting-set / no-reply / dead
- **Follow-up reminders**: default 7 days after send; `list overdue` and `list today` surface what needs attention
- **Reply-rate stats**: `stats` command gives 30-day totals by channel
- **Follow-up drafting**: `followup <ID>` hands off to `/cold-outreach` with prior-message context so you don't repeat openers
- **Invite hygiene** (`withdraw-stale`): opens LinkedIn's sent-invitations manager, surfaces invites older than N days (default 21), bulk-withdraws after explicit confirmation. Protects your invite acceptance rate so LinkedIn doesn't throttle your weekly invite limit.
- **Daily snapshot** (`daily`): one command for yesterday's outreach hygiene. Reads LinkedIn sent-invitations + recently-added connections + Gmail inbox to update: how many invites sent yesterday, how many accepted, how many pending, which emails got replies (and the reply gist). Drives the suppression logic that prevents over-touching the same person.

## Requirements

- [Claude Code](https://claude.ai/code) CLI
- [Claude in Chrome](https://chromewebstore.google.com/detail/claude-in-chrome) MCP server for authenticated browser sessions (LinkedIn, Gmail)
- [Playwright MCP](https://github.com/anthropics/claude-code/tree/main/.claude/plugins/playwright) server for form filling, file uploads, and iframe interaction
- **Optional**: [Apify MCP](https://apify.com/apify/actors-mcp-server) for founder/CXO email lookup in `/cold-outreach`. Without it, outreach falls back to LinkedIn DM only.

## Installation

```bash
claude plugin marketplace add siddarathVats/job-apply-plugin
claude plugin install job-apply@siddarathvats-plugins
```

## Usage

### First Time Setup

1. Invoke the skill:
   ```
   /job-apply
   ```

2. Provide your resume path when prompted:
   ```
   ~/Documents/resume.pdf
   ```

3. Review and confirm extracted profile information

### Applying to Jobs

Once your profile is set up:

1. Invoke the skill:
   ```
   /job-apply
   ```

2. Provide a job URL:
   ```
   https://www.linkedin.com/jobs/view/123456789
   ```

3. Watch as Claude fills out the application

4. Review the summary and confirm submission

### Searching for Jobs

Use `/job-search` to find jobs where you have an advantage:

1. Invoke the skill:
   ```
   /job-search
   ```

2. Claude will suggest keywords based on your resume:
   ```
   Based on your resume, I suggest searching for:
   - Keywords: "Senior Software Engineer", "Staff Engineer"
   - Location: San Francisco, CA
   - Experience: Mid-Senior (7 years)

   Ready to search, or would you like to adjust?
   ```

3. Confirm or modify the suggestions

4. Review results prioritized by:
   - Jobs with hiring managers listed (highest priority)
   - Jobs with 1st-degree connections
   - Jobs with alumni connections

Results are automatically saved to `~/.claude-job-searches/`.

### Finding Hiring at Funded Startups

```
/funded-startup-jobs
```

Mines all five sources (VC feeds, VC boards, LinkedIn funding-news, news, plus any pasted post). Or, fast-path a single post:

```
/funded-startup-jobs https://www.linkedin.com/posts/some-founder_we-just-raised-series-a-...
```

Single source only:

```
/funded-startup-jobs vc-boards
```

Results are saved to `~/.claude-funded-startups/` and offer a hand-off to `/cold-outreach`.

### Scanning Your LinkedIn Feed

```
/feed-scan                          # default: 20 min, job + partnership classification
/feed-scan --time 10                # shorter session
/feed-scan --types partnership      # collab/pilot/co-founder posts only
/feed-scan --min-score 70           # only surface high-score posts
/feed-scan --auto-outreach          # auto-queue top 3 for /cold-outreach (still requires send confirmation per recipient)
```

Saved to `~/.claude-feed-scans/feed-{date}.md`. Re-running on the same day skips already-seen posts.

### Cold Outreach to a Founder or Team Member

```
/cold-outreach Acme AI
/cold-outreach Acme AI, Founding MLE
/cold-outreach https://linkedin.com/in/jane-doe-acme
/cold-outreach https://linkedin.com/posts/jane-doe-acme_we-just-raised-...
/cold-outreach Acme AI --partnership     # explicit partnership framing instead of job pitch
/cold-outreach Acme AI short             # short DM variant, no email
```

The skill resolves the recipient (LinkedIn search + optional Apify email lookup), picks the right template variant (job vs partnership), drafts both a DM and an email personalized from your profile, previews them, and auto-sends only after you say so.

### Tracking Outreach

```
/outreach-tracker daily                # yesterday: sent / accepted / replied across all channels
/outreach-tracker daily week           # last 7 days rolled up by date
/outreach-tracker list                 # pending + overdue follow-ups
/outreach-tracker list overdue         # what's past due
/outreach-tracker reply 003            # mark a row as replied
/outreach-tracker followup 002         # draft a follow-up to a prior send
/outreach-tracker stats                # 30-day reply rate by channel
/outreach-tracker withdraw-stale       # sweep LinkedIn invites > 21 days unaccepted
/outreach-tracker withdraw-stale 14    # custom threshold (in days)
```

## Supported Platforms

| Platform | URL Pattern | Tool | Status |
|----------|-------------|------|--------|
| LinkedIn Easy Apply | `linkedin.com/jobs/view/*` | Chrome MCP | Supported |
| Greenhouse | `boards.greenhouse.io/*` | Playwright MCP | Supported |
| Ashby | `jobs.ashbyhq.com/*` | Playwright MCP | Supported |
| Lever | `jobs.lever.co/*` | Playwright MCP | Supported |
| Rippling | `*.rippling.com/*` | Playwright MCP | Supported |
| Workday | `*.myworkdayjobs.com/*` | Playwright MCP | Supported |

## Profile Storage

Your profile is stored at `~/.claude-job-profile.json` and includes:

- Personal information (name, email, phone, location)
- Work history
- Education
- Skills
- Social links (LinkedIn, GitHub, portfolio)

To reset your profile:
```
/job-apply reset profile
```

## Search Results Storage

Job search results are saved to `~/.claude-job-searches/` with timestamped filenames:

```
~/.claude-job-searches/
  search-2026-01-06T10-30-00.json
  search-2026-01-07T14-15-00.json
```

Each file contains:
- Search parameters (keywords, location, filters)
- List of jobs with full details
- Connection and hiring manager information
- Priority ranking

## Funded-Startup & Outreach Storage

```
~/.claude-funded-startups/
  funded-2026-05-14.md           # one file per /funded-startup-jobs run

~/.claude-feed-scans/
  feed-2026-05-27.md             # one file per /feed-scan run
  seen-posts.txt                 # cross-run dedupe (last 30 days)

~/.claude-outreach-log.md        # append-only outreach log

~/.claude-outreach-log/messages/
  001.json                       # full text of each sent message
  002.json

~/.claude-outreach-log/
  withdraw-history.md            # log of /outreach-tracker withdraw-stale sweeps
```

## Safety Features

- **Never enters passwords** - Stops if login is required
- **Never creates accounts** - You must create accounts yourself
- **Never submits without confirmation** - Always shows summary first
- **Never enters payment info** - Skips premium features
- **Confirms sensitive questions** - Salary, visa status, etc.
- **Outreach send caps** - `/cold-outreach` enforces one recipient per invocation and at most 5 cold LinkedIn DMs per day to avoid LinkedIn flag patterns
- **Identity honesty** - drafts always claim the profile owner's identity, never the operating account
- **Append-only outreach log** - `/outreach-tracker` cannot delete rows from chat; only the status/follow-up/reply columns are mutable

## Ethics & Responsible Use

This plugin automates outreach to and research about **real people**. Use it respectfully and lawfully:

- **You are the sender.** Every message goes out as *you*, from *your* logged-in accounts. Don't impersonate anyone, and don't operate it on an account that isn't yours to send from.
- **Consent & confirmation.** Nothing sends without your explicit in-chat `yes`. Read every draft before it goes out.
- **Respect rate limits — don't evade them.** The caps (one recipient per invocation, ~20 LinkedIn touches/day, weekly invite ceiling, send cooldowns) exist to keep you within platform norms and protect your acceptance rate. Don't add randomized delays or rotate accounts to dodge detection.
- **Don't hoard or publish others' data.** Research dossiers you generate contain personal information about real people. Keep them local and private — **do not commit them to a public repo** (this project's `.gitignore` excludes them). Delete what you no longer need. Be mindful of GDPR/CCPA-style expectations.
- **No scraping for resale or mass targeting.** This is for genuine, individualized professional outreach — not list-building or spam.
- **Email honesty.** Treat inferred/scraped emails as unverified guesses; prefer published addresses. Don't blast addresses you can't stand behind.

If a platform's Terms of Service conflict with a use you have in mind, follow the ToS.

## License

MIT License - See [LICENSE](LICENSE) for details.

## Attribution

This project builds on the original **[job-apply-plugin](https://github.com/neonwatty/job-apply-plugin)** by **Jeremy Watt ([@neonwatty](https://github.com/neonwatty))** (MIT). The base job-application skills (`/job-apply`, `/job-search`, `/job-preferences`) are his work. The outreach + sourcing skills (`/funded-startup-jobs`, `/feed-scan`, `/cold-outreach`, `/outreach-tracker`) are added on top in this fork.

## Contributing

Contributions welcome! Please open an issue or PR.
