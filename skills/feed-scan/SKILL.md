---
name: feed-scan
description: Scroll the user's LinkedIn home feed for a fixed time budget (default 20 min), classify posts as job-opening, partnership-opportunity, or both, dedupe, score against preferences, and hand off to /cold-outreach. Use when the user wants to mine their existing LinkedIn network for opportunities posted in the last few days.
allowed-tools: Read, Write, Bash, mcp__claude-in-chrome__*
---

# LinkedIn Feed Scan

Wall-clock-timeboxed scroll through `linkedin.com/feed/`. The user's feed already filters for people/companies they care about, this skill mines that signal for hiring posts and partnership/collab opportunities, then routes both to `/cold-outreach`.

---

## Phase 1: Load Profile & Parse Args

### Load profile
Read `~/.claude-job-profile.json`. Require core fields + `preferences`.

### Parse invocation args
Defaults in **bold**.

| Arg | Default | Effect |
|-----|---------|--------|
| `--time <minutes>` | **20** | Wall-clock budget for scrolling |
| `--max-posts <N>` | **300** | Hard cap on posts processed regardless of time |
| `--types <list>` | **job,partnership,service** | Comma-sep; can be `job`, `partnership`, `service`, or any combination |
| `--min-score <N>` | **50** | Filter output to posts scoring this or above |
| `--auto-outreach` | **false** | If set, queues top results for `/cold-outreach` without asking |

Show active config before scrolling:

```
Feed Scan
- Time budget: 20 min
- Max posts: 300
- Types: job + partnership
- Score floor: 50
- Auto-outreach: off
```

---

## Phase 2: Open Feed & Verify

1. Use Chrome MCP `tabs_create_mcp` → new tab.
2. Navigate to `https://www.linkedin.com/feed/`.
3. Verify logged in via `read_page`, look for the "Start a post" composer.
4. If not logged in: stop. Instruct the user to log in. Do not proceed.

---

## Phase 3: Scroll Loop (timeboxed)

Track wall-clock start time. Loop until `elapsed >= --time` OR `processedCount >= --max-posts` OR three consecutive empty-scroll cycles.

### Each iteration

1. Use `read_page` (or `javascript_tool` returning structured DOM data) to enumerate all currently-visible feed posts.
2. For each post not already in `seenPostIds` (dedupe by post URN / activity ID):
   - Capture: poster name, poster headline, post timestamp, body text, embedded links, hashtags, reactions count.
   - Run **classification regex** (Phase 4).
   - If classification is non-empty AND post is within the last 7 days, add to results.
   - Add post ID to `seenPostIds`.
3. Scroll down 2-3 viewport heights via `computer` (`scroll`, direction `down`) OR `javascript_tool` (`window.scrollBy(0, window.innerHeight * 2.5)`).
4. **Wait 3-4 seconds** for LinkedIn to lazy-load next batch.
5. Loop.

### Empty-scroll handling

If a scroll cycle adds zero new post IDs, increment `emptyScrollCount`. After 3 consecutive empty cycles, exit the loop (feed is exhausted for the current session).

### Time check

Read elapsed time before each iteration. If `elapsed >= --time`, exit the loop cleanly even if not at `--max-posts`.

### Anti-flag rules

- **Never click reactions, comment buttons, or follow buttons.** Read-only scroll.
- **Never click into post details.** Stay on the feed page.
- **Never auto-DM or auto-connect from the feed.** Outreach is `/cold-outreach`'s job, with confirmation.
- If LinkedIn shows a CAPTCHA, security check, or "We've restricted your access", stop immediately and report to the user.

---

## Phase 4: Classification

For each captured post body, apply regex-based classification. A post can carry **both** labels.

### `job` classification

Post matches if body contains any of (case-insensitive):

```
\b(we'?re hiring|now hiring|join us|join our team|open role|open position|currently hiring|looking for (a |an |our )?(engineer|MLE|ML engineer|data scientist|AI engineer|applied scientist|research engineer|founding))\b
\bapply (here|now|at)\b
\bDM (me|us) (if|for)\b
\b(ML|AI|data|engineering) (lead|head|director|VP) opening\b
```

### `partnership` classification

Post matches if body contains any of:

```
\b(looking to (chat|connect|collaborate|partner)|open to collab|seeking (a |co-|partners|builders))\b
\b(anyone (working on|building|interested in))\b
\b(co-founder search|technical co-founder|looking for co-founder)\b
\b(beta test|pilot|design partner|research partner|early users)\b
\b(would love to hear from|reach out if you)\b
\b(advisor|advisory) (search|role|capacity)\b
```

### `service` classification (revenue-opportunity / build-for-hire)

Pain-signal regex (match indicates the poster has an automatable workflow problem):

```
\b(still (doing|running|using|managing) .{0,40} (manually|by hand|in (excel|spreadsheets|notion)))\b
\b(spending (hours|days|weeks) on)\b
\b(takes (us|me|the team) .{0,30} (hours|days))\b
\b(wish (there was|i had|we had) (a |an )?(tool|way|automation|script|agent))\b
\b(no one (is )?building|nobody'?s built|why isn'?t there)\b
\b(repetitive|tedious|copy[- ]?paste|clicking through tabs|exporting (csv|csvs|to excel))\b
\b(productivity ceiling|bottleneck|stuck (at|in) (2015|2018|2020))\b
\b(looking for (a )?(freelancer|contractor|dev|developer|builder|engineer|automation))\b
\b(need help (with|automating|building))\b
\b(anyone (know|build|building|made) (a )?(tool|system|agent|automation) (for|that))\b
\b(open to (suggestions|recommendations|help|builders))\b
\b(if you (build|ship|do) .{0,40} (DM|message|reach out))\b
\b(side project|building this myself|hacking together)\b
\b(MVP|prototype|proof of concept|POC) (in|for|to)\b
```

Vertical-fit boost (additional points if the post also matches the user's stack/domain):

```
- Renewable energy / solar: \b(solar|PV|photovoltaic|SAM|PVsyst|inverter|grid|BESS|wind|renewable|clean ?energy)\b
- Healthcare / EHR: \b(MIMIC|EHR|clinical|patient|SDOH|healthcare|hospital|medical record|HL7|FHIR)\b
- RAG / vector search: \b(RAG|vector (search|db|database)|pgvector|embedding|semantic search|chunk|reranker)\b
- Agentic / LangGraph: \b(LangGraph|LangChain|multi-agent|agent (orchestration|workflow|loop)|tool[- ]?use|MCP server|CrewAI)\b
- Backend / API automation: \b(FastAPI|Flask|email (pipeline|automation)|AWS SES|cron|scheduled job|webhook)\b
- BI / analytics: \b(Power BI|Tableau|dashboard|KPI|reporting automation|spreadsheet|Google Sheets)\b
- Financial / NLP: \b(SEC|EDGAR|10-K|financial report|earnings|invoice|contract (parsing|review))\b
```

A post is classified `service` if it matches the pain-signal regex. Vertical-fit only boosts score; it isn't required.

### `service` disqualifiers (in addition to the global ones)

Drop or de-prioritize if the body contains:
- `pro bono only`, `volunteer`, `unpaid`, `internship-style`, not paid work.
- `looking for a co-founder`, that's partnership, not service.
- `we already have a contractor / vendor / team for this`, closed door.
- `looking for senior staff engineers`, that's job-hire.

### Disqualify

Drop the post if body contains any of (case-insensitive):

- `intern`, `internship` (matches `excludePatterns`)
- `course`, `bootcamp`, `cohort`, `webinar`, `newsletter` (content marketing, not opportunity)
- "Repost" / "Reposted" without new content from the user
- Pure congratulations posts ("Excited to announce I joined X")
- Job board aggregator reposts (Wellfound, Y Combinator news, etc., user already gets these via `/funded-startup-jobs`)

### Title-keyword match (for job posts)

If classified `job`, check whether the body contains any `preferences.targetTitles` (fuzzy: "MLE" ≈ "Machine Learning Engineer"). Record `titleMatch: true/false`.

### Domain-keyword match (for partnership posts)

If classified `partnership`, check overlap between post keywords and user's `skills` + recent work descriptions. Compute `overlapScore: 0-5` based on shared keyword count.

---

## Phase 5: Enrich & Score

### Enrich

For each kept post:

1. Capture poster's LinkedIn profile URL.
2. If the poster's headline / company isn't already in the post data, capture it.
3. Note connection-degree if visible on the card (1st / 2nd / 3rd).
4. If the post mentions a company different from the poster's (e.g. "we're hiring at Acme"), capture that as `target_company`.

### Score (0-100)

| Signal | Points |
|--------|--------|
| Job + exact title match | 25 |
| Job + partial title match | 15 |
| Job + remote preference match | 10 |
| Partnership + overlap score ≥ 3 | 20 |
| Partnership + overlap score 1-2 | 10 |
| **Service: pain signal matched** | **25** |
| **Service: vertical-fit (renewable / healthcare / RAG / agentic / etc.)** | **20** |
| **Service: poster looks like founder / solo dev / small-team** (headline contains `founder`, `solo`, `indie`, `bootstrapped`, `building`, `CEO @ small`, `CTO @ small`) | **15** |
| **Service: pain mentions specific tool the user knows** (PVsyst, Power BI, Excel, etc.) | **10** |
| **Service: poster invites contact** ("DM me", "reach out", "looking for") | **10** |
| Poster is 1st-degree connection | 15 |
| Poster is 2nd-degree connection | 8 |
| Posted within last 24h | 15 |
| Posted within last 3 days | 8 |
| Posted within last 7 days | 3 |
| High engagement (50+ reactions) | 5 |
| Post links to careers page / Calendly / form | 5 |
| Dallas-local keyword in body or headline | 8 |

**Note on connection-degree weight in `service` mode**: When `--types` includes `service` and the post is classified `service`, connection-degree weighting is HALVED (1st = 7, 2nd = 4) because revenue-opportunity outreach goes to anyone with a clear pain, regardless of how warm the connection is. Connection just means lower friction, not higher fit.

Sort descending. Drop anything below `--min-score`.

---

## Phase 6: Output

### Terminal summary

```
============================================================
  Feed Scan, 2026-05-27  (18m 42s elapsed)
============================================================
  Posts seen: 247 | Job-classified: 19 | Partnership: 11 | Both: 4
  After dedupe + score floor (50): 22
  1st-degree posters: 8 | 2nd-degree: 11 | Out-of-network: 3
============================================================

Score | Type      | Poster              | Conn | Topic / Role                          | Posted
----- | --------- | ------------------- | ---- | ------------------------------------- | -------
  88  | job       | Jane Doe (CEO Acme) | 2nd  | Founding MLE, Series A, remote US    | 6h ago
  82  | partner   | Sam Ko (Bolt)       | 1st  | Looking for design partners, agents  | 1d ago
  78  | both      | Lin Park (Verdant)  | 2nd  | Hiring 2 MLEs AND pilot partners      | 2d ago
  ...

============================================================
  Saved: ~/.claude-feed-scans/feed-2026-05-27.md
============================================================
```

### Markdown file

`~/.claude-feed-scans/feed-{YYYY-MM-DD}.md`, create directory if missing. One section per post:

```markdown
### 1. Jane Doe, Founding MLE at Acme AI (score: 88)
- **Type**: job
- **Connection**: 2nd-degree
- **Posted**: 2026-05-27 09:14 (6h ago)
- **Post snippet**: "Series A closed last week, hiring 3 founding engineers..." (≤30 words, fair-use)
- **Post URL**: https://linkedin.com/feed/update/urn:li:activity:...
- **Poster LinkedIn**: linkedin.com/in/janedoe
- **Target company**: Acme AI
- **Role/topic match**: Founding MLE / Remote US
- **Suggested outreach**: `/cold-outreach https://linkedin.com/feed/update/urn:li:activity:...`
```

### Hand-off

After saving, prompt:

> {N} posts scored ≥ {threshold}. Want me to queue any of them for `/cold-outreach`? Reply with a number, "top 3", "top job", "top partnership", or "none".

If `--auto-outreach` was set, automatically open `/cold-outreach` for the top 3 (one at a time, each with full preview + confirmation, the `--auto` flag only skips the queuing prompt, never the send confirmation).

---

## Safety Rules

1. **Read-only scroll.** No reactions, comments, follows, or clicks beyond the scroll action itself.
2. **Wall-clock cap is hard.** Even if results are sparse, do not exceed `--time + 60s` (60s buffer for cleanup).
3. **Stop on security challenge.** Any CAPTCHA, security check, or rate-limit modal aborts the scan immediately and reports to the user.
4. **No credential entry.** If LinkedIn logs the user out mid-scroll, stop and tell them.
5. **Copyright**, post snippets stored in the output file must be ≤30 words. Never reproduce full posts. Always store the source URL alongside the snippet.
6. **Dedupe across runs.** Maintain `~/.claude-feed-scans/seen-posts.txt` (one post URN per line, last 30 days). Skip post IDs in this file when classifying. This prevents re-surfacing the same post day after day.
7. **No DM-from-feed.** Even when a 1st-degree connection's post is high-score, the skill does not DM directly. It hands off to `/cold-outreach` with full preview and confirmation. The skill enforces this even if the user says "just DM them".

---

## Example Invocations

```
User: /feed-scan
Claude: [20 min scroll, classifies, saves, prompts for outreach hand-off]

User: /feed-scan --time 10 --types partnership
Claude: [10 min scroll, partnership posts only]

User: /feed-scan --time 30 --auto-outreach
Claude: [30 min scroll, then auto-opens /cold-outreach for top 3, each still requires explicit send confirmation]

User: /feed-scan --min-score 70
Claude: [Default 20 min, but only outputs posts scoring 70+]
```

---

## Why this complements `/funded-startup-jobs`

| Source | Strength | Weakness |
|--------|----------|----------|
| `/funded-startup-jobs` | Surfaces companies you don't follow yet; mines VC accounts + Crunchbase | Doesn't see what's in your network |
| `/feed-scan` | Mines signal from people you already follow / are connected to, highest reply rate | Limited to what's in your feed; misses big news you don't follow |

Run both in the same week for full coverage. `/feed-scan` is the higher-leverage one because 1st/2nd-degree outreach beats cold outreach 5-10x on reply rate.
