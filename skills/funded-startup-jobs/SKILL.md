---
name: funded-startup-jobs
description: Discover hiring at recently-funded startups by mining VC LinkedIn feeds, VC job boards, LinkedIn funding-news search, Crunchbase/news feeds, and user-pasted LinkedIn posts. Use when the user wants to find jobs at startups that just raised, or wants to act on a specific funding-announcement post.
allowed-tools: Read, Write, Bash, WebSearch, WebFetch, mcp__claude-in-chrome__*, mcp__plugin_playwright_playwright__*
---

# Funded-Startup Jobs Discovery

Surfaces hiring signals from startups that recently raised. Combines five signal sources, dedupes by company, scores against the user's preferences, and saves a ranked list ready to feed into `/cold-outreach` or `/job-apply`.

---

## Phase 1: Load Profile & Parse Mode

### Step 1: Load Profile

Read `~/.claude-job-profile.json`. Require the `preferences` block (set via `/job-preferences`). If missing, stop and direct the user to `/job-preferences`.

### Step 2: Parse Invocation Mode

The user can invoke this skill with arguments that select sources. If no argument is given, run **all** sources.

| Argument | Source |
|----------|--------|
| `vc-feeds` | VC company LinkedIn pages (Tier 2 in `vc-sources.md`) |
| `vc-boards` | VC job boards (Tier 1 in `vc-sources.md`) |
| `funding-search` | LinkedIn content search for "just raised" + "hiring" |
| `news` | WebSearch on Crunchbase / TechCrunch funding announcements |
| `<URL>` | Single LinkedIn post URL pasted by the user — fast-path mode |
| `all` | All four sources above (default) |

### Step 3: Show Active Config

```
Funded-Startup Discovery
- Sources: VC feeds, VC boards, funding search, news
- Time window: 2 weeks
- Target titles: <from preferences>
- Remote: <from preferences>
- Min salary: <from preferences>
```

---

## Phase 2a: User-Pasted LinkedIn Post (fast-path)

If the invocation includes a `linkedin.com/posts/` or `linkedin.com/feed/update/` URL, skip Phase 2b–2e and:

1. Open the URL via Chrome MCP (`tabs_create_mcp` + `navigate`).
2. Use `read_page` to extract: poster name, poster headline, post body, any company mentions, any links.
3. Parse the post body for:
   - Company name(s) — usually after "join", "we're hiring at", "@<company>"
   - Funding stage/amount — "Series A", "$10M", "seed"
   - Role hints — "engineer", "ML", "PM"
   - Application URL or contact (apply link, founder email)
4. Output a single startup record (see Schema below) with `source: "linkedin-post-paste"` and `sourceUrl: <pasted URL>`.
5. Jump to Phase 4 (enrichment).

---

## Phase 2b: VC LinkedIn Feeds (Chrome MCP)

Load `vc-sources.md` Tier 2 list. For each VC (cap at 10 per run to avoid LinkedIn rate-limit):

1. Navigate to `https://www.linkedin.com/company/<handle>/posts/`.
2. Verify logged in. If not, stop and instruct user to log in.
3. Use `read_page` to capture the first ~10 posts.
4. For each post, extract: post date, body text, author, embedded URLs, company tags.
5. **Keep only posts** that match these patterns (case-insensitive regex):
   ```
   /\b(just raised|announcing our|raised \$|series [a-d]|seed round)\b/
   /\b(hiring|we're hiring|join the team|open roles|come build)\b/
   /\bportfolio (company|startup)\b/
   ```
6. Discard posts older than `defaultTimeRange` (default 2 weeks).
7. For each kept post, extract company name(s) and funding details. If the post is the VC announcing the round, the linked company is the target.
8. **2-3 second delay** between VC feed page loads.
9. Cap: 5 matching posts per VC, 30 total.

---

## Phase 2c: VC Job Boards (Playwright / WebFetch)

Load `vc-sources.md` Tier 1 list. For each board (cap at 8 per run):

1. Use `WebFetch` first — most VC boards render server-side.
2. If `WebFetch` returns no role data (JS-rendered SPA), fall back to Playwright (`browser_navigate` + `browser_snapshot`).
3. Search the page for the user's `targetTitles`. Capture: company, role title, location, link, posted-date if available.
4. Filter by:
   - `remotePreference` — match "remote", "anywhere", "us-remote" in the role's location
   - `excludePatterns` — drop any role matching these
   - posted within `defaultTimeRange`
5. Cap: 10 roles per board, 50 total.

---

## Phase 2d: LinkedIn Funding-News Search (Chrome MCP)

1. Build the query from `vc-sources.md` "Funding-news search patterns".
2. Navigate to `https://www.linkedin.com/search/results/content/?keywords=<encoded>&datePosted=past-month` (use `past-week` if `defaultTimeRange` is "last week", `past-month` otherwise).
3. Scroll-load 3 times (~30 posts).
4. Apply the same regex filter as Phase 2b.
5. Cap: 20 posts.

---

## Phase 2e: News / Crunchbase Fallback (WebSearch)

1. Run 2-3 WebSearch queries from `vc-sources.md` "Crunchbase / WebSearch fallback queries", substituting the current month/year.
2. For each search hit (cap 5 per query), use `WebFetch` to pull the article and extract: company name, funding amount/stage, announcement date.
3. For each unique company surfaced, attempt to find the careers page:
   - Try `https://<company>.com/careers`, `/jobs`, `/join-us`.
   - Try `WebSearch` for `<company> careers <target_title>`.
4. Capture roles that match `targetTitles` and `remotePreference`.
5. Cap: 15 companies, 30 roles.

---

## Phase 3: Dedupe & Enrich

### Dedupe

Normalize company names (lowercase, strip "Inc.", "Labs", ", Inc"). If two records share a normalized name, merge them — keep the earliest funding signal, union the roles, list all `sourceUrl`s.

### Enrich (best-effort, soft-fail)

For each unique company:

1. **Careers page**: if we don't already have one, try `WebSearch` for `"<company>" careers site:<company>.com`.
2. **Funding history**: if missing stage/amount, WebSearch `"<company>" "Series" OR "seed" raised`.
3. **Headcount estimate**: WebSearch `"<company>" linkedin employees` (capture range like "11-50").
4. **Founder/CXO**: capture if surfaced from the post; otherwise leave blank — `/cold-outreach` will resolve later.

**Time-box enrichment**: max 5 seconds per company. Skip on timeout.

---

## Phase 4: Scoring

Each record gets a 0-100 score:

| Signal | Points |
|--------|--------|
| At least one role matches `targetTitles` exactly | 25 |
| Role matches partially (e.g. "AI Engineer" matches "Staff AI Engineer") | 15 |
| Remote / location matches `remotePreference` | 15 |
| Posted within last 7 days | 15 |
| Posted within 14 days (but not 7) | 8 |
| Salary listed and ≥ `minBaseSalary` | 10 |
| Funding announced in last 60 days (recent raise) | 10 |
| Company size 11-200 (typical fundable hiring window) | 10 |
| User has a direct LinkedIn connection at the company | 15 (LinkedIn-sourced only) |
| Founder/CXO name surfaced (outreach-ready) | 10 |
| Matches `excludePatterns` | -100 (drop) |

Cap at 100. Sort descending.

---

## Phase 5: Output

### Schema (per record)

```json
{
  "company": "Acme AI",
  "normalizedName": "acme",
  "fundingStage": "Series A",
  "fundingAmount": "$15M",
  "fundingDate": "2026-05-08",
  "headcount": "11-50",
  "careersUrl": "https://acme.com/careers",
  "roles": [
    {
      "title": "Founding ML Engineer",
      "location": "Remote (US)",
      "url": "https://acme.com/careers/ml-eng",
      "postedDate": "2026-05-12"
    }
  ],
  "founder": {"name": "Jane Doe", "title": "CEO", "linkedin": "linkedin.com/in/janedoe"},
  "sources": ["vc-feed:a16z", "linkedin-post-paste"],
  "sourceUrls": ["https://linkedin.com/posts/..."],
  "postSnippet": "We just raised $15M Series A and we're hiring 5 engineers...",
  "score": 87
}
```

### Terminal Display

```
============================================================
  Funded-Startup Jobs — 2026-05-14
============================================================
  Sources: VC feeds (12), VC boards (28), Funding search (9), News (6)
  Total unique companies: 38 | After scoring: 22 with score >= 50
============================================================

Score | Company         | Stage  | Roles                              | Source         | Founder
----- | --------------- | ------ | ---------------------------------- | -------------- | ----------
  91  | Acme AI         | A $15M | Founding ML Eng (Remote US)        | a16z feed      | Jane Doe
  87  | Bolt Robotics   | Seed   | ML Eng, Data Sci (Remote)          | YC WaaS        | Sam Ko
  74  | Verdant ML      | A $20M | AI Engineer (Remote US)            | Sequoia board  | —
  ...

============================================================
  Saved: ~/.claude-funded-startups/funded-2026-05-14.md
============================================================
```

### Markdown File

Save full results to `~/.claude-funded-startups/funded-{YYYY-MM-DD}.md`. Create the directory if missing.

Include for each company:
- Header with company, score, stage, amount, date
- Role list with links
- Source URLs (post URLs, board URLs)
- Founder block (if known)
- Post snippet (≤30 words, original wording preserved — short enough to be fair-use)

### Hand-off Prompt

After saving, ask:

> Top {N} startups with founder/CXO info ready for outreach. Want me to launch `/cold-outreach` for any of them? Say a number, "top 3", or a company name.

If the user picks one, hand off to `/cold-outreach` with the company record as input.

---

## Safety Rules

1. **Never enter credentials.** Stop if login is required.
2. **LinkedIn rate-limiting** — minimum 2-3 second delays between page loads. Stop after 3 consecutive empty/blocked responses and report.
3. **No bulk-scrape**. Caps in Phase 2 are hard limits, not suggestions.
4. **Copyright** — when storing post snippets, keep them ≤30 words and never reproduce full posts. Always store the source URL alongside.
5. **Graceful degradation** — if any source fails, skip and continue. Report at the end.
6. **Verify before recommending** — if the skill surfaces a company, confirm the careers page actually loads before claiming the role is open.
7. **Honor profile preferences strictly** for `excludePatterns` — these are hard filters, not soft signals.

---

## Example Invocations

**Default (all sources):**
```
User: /funded-startup-jobs
Claude: [Runs all 4 sources, dedupes, scores, saves]
```

**One LinkedIn post (fast-path):**
```
User: /funded-startup-jobs https://www.linkedin.com/posts/some-founder_we-just-raised-series-a-activity-1234
Claude: [Reads post, extracts company + funding + role hints, saves single record, offers /cold-outreach hand-off]
```

**Single source:**
```
User: /funded-startup-jobs vc-boards
Claude: [Only scrapes Tier 1 VC job boards from vc-sources.md]
```

**Time-range override:**
```
User: /funded-startup-jobs last week
Claude: [Tightens window to 7 days for funding signals]
```
