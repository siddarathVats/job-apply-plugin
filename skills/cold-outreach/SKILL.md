---
name: cold-outreach
description: Draft and send a cold LinkedIn DM and/or email to a startup founder/CXO, personalized from the user's profile. Uses Chrome MCP for LinkedIn, Apify MCP for email lookup, and auto-sends only after explicit user confirmation. Use when the user wants to reach a specific founder or CXO about a role.
allowed-tools: Read, Write, Bash, WebSearch, WebFetch, mcp__claude-in-chrome__*, mcp__plugin_playwright_playwright__*, mcp__apify__*
---

# Cold Outreach

Finds a founder/CXO, drafts a personalized DM and email, optionally resolves their email via Apify, previews to the user, and (on explicit confirmation) sends via Chrome MCP. Every send is logged via `/outreach-tracker`.

---

## Bedrock outreach rules (apply to every draft, override template defaults)

These reflect what actually earns replies — and what reads as rushed or AI-generated:

1. **Pain-first, researched opener.** Open on a specific pain/focus that only real research surfaces (their own post, a stat they quote, their exact product framing). Never generic flattery ("love what you're building").
2. **First touch carries NO link and NO meeting CTA.** In a connection-request note or the first DM/message, do NOT include a portfolio/resume/work URL, and do NOT ask for a coffee chat or call. Both read as rushed/pushy on a cold contact. Spend the space on substance. The link and the chat invite are *later-step* moves — fine in a follow-up email or a post-accept DM once there's rapport.
3. **Close warm and curiosity-first — never on what you build/sell.** Do not end on "that's what I build" / "that reliability layer is what I build" (reads sales-y and presumptuous). End oriented toward THEM: e.g. "Would love to connect and explore this with you", "Looking forward to connecting and learning more about what you're building."
4. **No em-dashes (—) and no "+" standing in for "and."** Both read as AI-generated. Use periods and commas. (The validator strips these, but write clean — don't rely on it.)
5. **Stay grounded.** Every claim maps to the profile (`skills` / `workHistory`). No invented numbers, tools, or scope.
6. **Honor the chosen tone** (Phase 1, Step 0) without breaking rules 1-5.

---

## Phase 1: Load Inputs

### Step 0: Confirm outreach tone

Before drafting, ask the user how they want to come across. Default to **Warm & curious** if they don't pick:

```
How should this read?
  1. Warm & curious (recommended) — genuine interest in their work, low-pressure,
     closes on "would love to connect and explore this"
  2. Direct & concise — straight to the point, minimal warm-up
  3. Peer / casual — builder-to-builder, informal
  4. Formal / professional — measured and polished

(Press enter for Warm & curious.)
```

Thread the chosen tone through the draft voice and the closer. Tone **never** overrides the Bedrock rules above (no link/CTA in the first touch, warm-not-salesy close, no AI tells, grounded claims).

### Step 1: Load profile

Read `~/.claude-job-profile.json`. Require the core fields (name, email, phone, linkedInUrl, workHistory, skills). If `preferences.targetTitles` is missing, warn the user — outreach without preferences leads to generic drafts.

### Step 2: Resolve target

The skill accepts any of these invocation forms:

| Form | Example | Behavior |
|------|---------|----------|
| Company name | `/cold-outreach Acme AI` | Look up most-recent record in `~/.claude-funded-startups/` first; else ask user for context |
| Company + role | `/cold-outreach Acme AI, Founding MLE` | Pin the role hint |
| LinkedIn profile URL | `/cold-outreach https://linkedin.com/in/jane-doe-...` | Target individual directly |
| LinkedIn post URL | `/cold-outreach https://linkedin.com/posts/...` | Pull poster as target + post as reference |
| No arg | `/cold-outreach` | Ask user for target |

### Step 3: Confirm target context

Show the user what you resolved before drafting:

```
Outreach target
- Company: Acme AI (Series A, $15M, announced 2026-05-08)
- Role hint: Founding MLE
- Recipient: Jane Doe (CEO) — linkedin.com/in/janedoe (LinkedIn ✓ / Email: not yet resolved)
- Reference post: linkedin.com/posts/... ("we just raised $15M and we're hiring...")

Proceed?
```

If the recipient block is incomplete (no LinkedIn URL or no name), run Phase 2 to find them.

---

## Phase 2: Find the Recipient (when missing)

### Step 1: LinkedIn search via Chrome MCP

1. Open `https://www.linkedin.com/search/results/people/?keywords=<company>%20<role_hint>` (role_hint defaults to "founder OR CEO OR CTO OR Head of AI").
2. Use `read_page` to get the top 5 results.
3. Score each result:
   - +30 if title contains "Founder", "Co-founder", "CEO", "CTO", "Chief"
   - +20 if title contains "Head of AI", "Head of Engineering", "VP Engineering"
   - +10 if current company exactly matches target
   - -50 if current company does not match
4. Pick top scorer. Capture: name, title, profile URL, location, current company.

### Step 2: Email lookup, prefer public sources first

Order matters: public sources before scraping, so `also_emailed_clause` logic stays honest.

1. **Public source check first.** Search for the email via WebFetch on:
   - Company website About / Team / Contact page
   - The target's personal site (linked from their LinkedIn profile)
   - AngelList / Wellfound profile (public)
   - GitHub profile bio (some founders list email)
   - Their public blog or Substack
   If found, record `emailSource: "public:<url>"`.

2. **Apify fallback.** Only if public sources turn up nothing:
   - `mcp__apify__search-actors` query "email finder linkedin", pick a well-rated actor (e.g. `lukaskrivka/linkedin-email-finder` or similar).
   - Call the actor with `{ "linkedInUrl": "<profile_url>" }`. `waitSecs: 30`.
   - If verified email returned, record `emailSource: "apify"`.

3. **User-supplied.** If the user pastes an email in chat, record `emailSource: "user-supplied"` (treated as public for `also_emailed_clause` logic, since the user vouches for it).

4. **Fail-soft.** If nothing found, leave email blank. LinkedIn DM only.

`emailSource` flows downstream to Phase 3 step 8 to control whether the DM mentions the email.

### Step 2.5: **Mandatory DEEP research before drafting (the #1 reply-rate driver)**

> **Follow the full process in `research-dossier.md`** (the standard approach): 4 phases (full profile capture incl. About → recent activity → company/product deep-dive → synthesis), written to a `outreach-dossiers/<lastname-company>.md` dossier ending with the single strongest hook. Notes that come out of it are pain-first and link-free, closing warm and curiosity-first (no coffee-chat/call CTA in the first touch) per Bedrock rules 2-3.

Research the *entirety* of the person before any draft. Depth correlates directly with replies — the deeply-researched, pain-first messages get instant responses (proof: Geir/DeepXL replied within hours), generic ones get ignored. Do all 4 layers, in priority order (per `service-pitch-requires-research` + `outreach-message-voice` memories):

1. **Their product** — WebFetch the homepage + product/features + blog/launch posts; note the sandbox/demo if any.
2. **Their posts & comments** — read the *person's own* LinkedIn activity via Chrome MCP (`/in/<slug>/recent-activity/all/` and `/comments/`). This is the richest pain signal — capture the exact framing/stats/events they evangelize so you can echo their own words back.
3. **Them** — background, prior ventures, location, a genuine rapport hook (e.g. Geir: ex-Special-Forces, Dallas-local, a GitHub repo on the exact problem).
4. **Their team (if possible)** — co-founders/key engineers (company team page, "People you may know from their company") — reveals stack, hiring needs, and where an extra pair of hands fits.

Extract: what they sell, who to, their *current* focus/pain (from their own posts), and the rapport hook. The opener must name a pain only this research surfaces.

If any of the following is true, **defer the recipient** — do NOT draft:
- Homepage is unreachable (404, SSL expired, parked domain).
- Research turns up no concrete product detail / pain hook after a real attempt.
- Recipient's role/identity is unclear from LinkedIn + the company website (wrong-person risk — see the FluxAI mis-match: Apify mapped a name to the wrong company's domain).

Surface the deferral with a one-line reason. Never auto-fall-back to a generic "I assume you have manual workflows" pitch — reads as arrogant + lazy.

### Step 3: Show resolved recipient to user

```
Resolved:
- Name: Jane Doe
- Title: Co-founder & CEO at Acme AI
- LinkedIn: linkedin.com/in/janedoe
- Email: jane@acme.ai (Apify-verified) / not found

Use this person? (y / n / pick another)
```

Wait for user confirmation before drafting.

---

## Phase 3: Draft the Messages

Load `templates.md`. Build personalization slot values from the profile + target record:

1. Compute `{years}` from `workHistory[].startDate / endDate`.
2. Extract `{domain_1}`, `{domain_2}` from work history descriptions (one short noun phrase each).
3. Pick `{top_3_skills}` by intersecting `profile.skills` with keywords found in the target's careers-page text, company name domain, and post snippet. If no intersection found, pick the user's top 3 skills.
4. Pick `{strongest_project}` from current/most-recent role with a one-line summary.
5. Build `{specific_detail_from_post}` only if a post reference exists — quote ≤6 consecutive words from the post snippet, never more.
6. Resolve `outreachType`:
   - `job` (default) — pitching for an open role
   - `partnership` — collaboration / pilot / research-comparison ask, no role pitch
   - `service` — offering to build / automate / recreate something for revenue, with a free POC as wedge
   - Inferred from invocation context: if input record from `/funded-startup-jobs` or carries a `roleHint`, default `job`. If input record is from `/feed-scan` with `classification = partnership`, default `partnership`. If `/feed-scan` classification is `service` or the record has `revenueOpportunity = true`, default `service`. User can override: `/cold-outreach <target> --partnership` / `--job` / `--service`.
   - **Connection-degree filter relaxed for `service`** — unlike `job`/`partnership` modes which suggest preferring 1st/2nd-degree, service-pitch outreach goes to anyone with a clear pain regardless of connection. Cold-DM via connection-request-note for 2nd/3rd, or InMail / email for out-of-network.

7. Pick variant:
   - **Variant A** (DM, job): post reference exists → Variant A from `templates.md`.
   - **Variant B** (DM, job): no post reference → Variant B.
   - **Variant C** (connection-request note): use whenever the channel is a connection-request note (the default for cold 2nd/3rd-degree). Pain-first, **NO portfolio/work link, NO meeting CTA**, warm curiosity-first close (Bedrock rules 1-3). Also used when the user says "short".
   - **Variant D** (DM, partnership): when `outreachType = partnership` → Variant D.
   - **Variant E** (DM, service): when `outreachType = service` → Variant E (free POC wedge mandatory).
   - **Email (job)**: long form unless user says "short".
   - **Email (partnership)**: partnership-variant email.
   - **Email (service)**: service-pitch email with concrete POC scope + 2 evidence bullets + free POC wedge.

8. Compute `also_emailed_clause`:
   - If this invocation sends both DM and email, AND the email address came from a **public source** (company website About page, AngelList profile, personal site, GitHub README, or user-supplied), render one of the three soft phrasings from `templates.md`. Vary across recipients (track last-used phrasing in `~/.claude-outreach-log/phrasing-history.txt`, pick a different one).
   - If email came from Apify scraping or any non-public lookup, `also_emailed_clause` is **empty string**. Do not signal that the inbox was discovered.
   - If user said "don't mention email" this session, `also_emailed_clause` is empty.
   - For the 300-char connection-request note (Variant C), use `short_also_emailed_clause` instead, capped at 30 chars; if budget tight, leave empty.

9. Validation pass enforces (see "Validation pass, before preview" section for the three-layer logic):
   - Slot resolution, anti-AI tells, mode-integrity (partnership/job/service), length, single CTA, opener hook quality.
   - **Service mode extras**:
     - Free-POC line must be present (literal "free POC" or "no commitment" phrasing).
     - No pricing tokens: `$`, `quote`, `rate`, `budget`, `proposal`, `SOW`, `retainer`, `per hour`, `per project`.
     - No job-pitch tokens: `hiring`, `role`, `position`, `opening`, `career`, `join your team`.
     - No overpromise tokens: `guarantee`, `100%`, `silver bullet`, `transform your business`, `10x your`.
     - At least one piece of evidence with a measurable outcome (number, percentage, named project, or scale).
     - POC scope must name a *narrow* deliverable, not "I can help with stuff."

### Validation pass, before preview

The validator runs three layers. Layer 1 hard-rejects (stops, asks user). Layer 2 soft-fixes (rewrites silently). Layer 3 final integrity check.

#### Layer 1, hard rejects

Stop and surface to user:
- Any `{slot}` left unresolved (zero `{` characters allowed in output).
- Sender claims a skill, tool, framework, or domain not present in profile `skills` or `workHistory` descriptions.
- Sender claims a specific number not present in profile ("shipped to 1M users", "team of 10").
- Compensation mentioned anywhere in first touch.
- Same recipient triggered the suppression rule from Safety #4.
- `outreachType = partnership` AND any of these forbidden keywords appear: `hiring`, `role`, `position`, `opening`, `career`, `join your team`, `apply`.
- `outreachType = job` AND any partnership-only slot (`{capability_1}`, `{topic_keyword}`, etc.) appears.
- CXO opener hook fails any of the four "CXO opener construction rules" in `templates.md` (no concrete reference, generic flattery, or unsupported expertise claim).

#### Layer 2, silent rewrites (anti-AI tells)

The validator applies these substitutions before preview:

**Punctuation rewrites:**
- Every `—` (em-dash, U+2014) is removed. Context-aware replacement:
  - Mid-sentence interruption: replace with `,` if the interrupting clause is short (<8 words), else split into two sentences with `.`
  - Sentence-ending dash: replace with `.` or `,` depending on what follows
- Every `–` (en-dash, U+2013) is removed via the same logic.
- ` - ` used as a sentence break: replace with `,` or `.`
- `; ` between independent clauses: replace with `. ` + capitalize next word
- Mid-sentence `...`: remove or replace with `,`

**Lexical rewrites (banned words and phrases, scan case-insensitively):**

Strip and rewrite the surrounding sentence in a lower-register, more concrete voice:
```
delve, delving, delved, tapestry, landscape (filler), leverage (verb), synergy,
furthermore, moreover, additionally, in today's fast-paced, seamlessly, robust,
cutting-edge, state-of-the-art (unless quoting company), ecosystem (unless quoting),
intricate, intricacies, multifaceted, paradigm, testament to, holistic, realm,
underscore, foster (verb meaning encourage), elevate (verb), unlock (verb),
empower, harness (verb), navigate (figurative)
```

Banned opener / desperation phrases (hard strip, rewrite sentence):
```
I hope this finds you well, Hope you are doing well, Trust this email finds you well,
I came across, I stumbled upon, I happened to see,
I wanted to take a moment, I am writing to express my interest,
I need a job, I am looking for a job, any role, any opportunity,
I would be grateful, I'd be so grateful, please consider, please give me a chance,
I promise, I won't let you down, humbly request
```

Banned closers (rewrite to topic-specific close):
```
I look forward to hearing from you, Hoping for a positive response,
Thank you for your time and consideration (in closing)
```

**Structural rewrites:**
- Three-bullet "I bring X, Y, Z" lists with parallel verb starts: rewrite to vary sentence openings.
- Symmetric parallel sentences ("Your X is impressive. My Y is impressive."): break parallelism.
- DM body containing bullet markers (`-`, `*`, `•`) other than Variant D: convert to prose.
- "As a [role] with [N] years of experience..." opener: rewrite to reference the recipient first.

#### Layer 3, final integrity check

After layers 1 and 2:
- All slots still resolved.
- Length within bounds: DM <= 700 chars (300 for connection-request note); email body 130-220 words.
- One CTA only (count "15 min", "call", "chat", "write-up" mentions, must be exactly one).
- No banned phrase, AI tell, or em-dash slipped through (re-scan).
- The draft does NOT contain any phrase from layers 1 or 2 (regression check).
- Opener hook still satisfies the 4 CXO rules from `templates.md`.

If layer 3 fails, loop back to layer 2. Maximum 2 loops, after which surface the unresolvable issue to user.

**Never preview a draft that failed any layer.** Silently fix layer 2 issues; surface layer 1 and layer 3 issues.

---

## Phase 4: Preview to User

Show both drafts, clearly labeled:

```
─── LinkedIn DM (612 chars) ───────────────────────
Hi Jane — congrats on the Series A ($15M). The
agentic eval framework angle caught my eye.

I'm an MLE with 3.5 yrs across renewable-energy ML
(Adani Green) and healthcare ML (SMU); current work
is multimodal EHR models at SMU. Stack: PyTorch,
LangGraph, LLMs.

If the Founding MLE role is still open, would love
to chat. Resume here: github.com/siddarathVats.

— Siddarath
───────────────────────────────────────────────────

─── Email — Subject: "Founding MLE role + LangGraph match" ─
Body (178 words):

Hi Jane,

Congrats on Acme AI's Series A ($15M) last week. Loved
the framing in your launch post about agentic evals.

I'm reaching out about the Founding MLE role.

Quick background:
- 3.5 yrs as an ML engineer — Adani Green Energy
  (SVM/ANN/LSTM on GCP for ~9 GW renewable portfolio),
  now at SMU (multimodal MIMIC-III EHR models on the
  SuperPOD).
- Stack matches: PyTorch, LangGraph, LLMs.
- Recent project: Multimodal EHR readmission model —
  github.com/siddarathVats.

Resume: github.com/siddarathVats
LinkedIn: linkedin.com/in/siddarath-vats-51bb65155

Happy to send a more detailed write-up, or jump on a
15-min call this week or next.

Best,
Siddarath Vats
───────────────────────────────────────────────────

Send options:
  1. Send DM only
  2. Send email only
  3. Send both (DM first, email as follow-up timing)
  4. Edit (tell me what to change)
  5. Cancel
```

Wait for explicit user choice. Never auto-send without selection.

---

## Phase 5: Send

### Step 1: LinkedIn DM via Chrome MCP

If user picked 1 or 3:

1. Navigate to recipient's LinkedIn profile.
2. Verify logged in.
3. Find the "Message" button. If not visible:
   - If "Connect" is the primary action, the user isn't connected and free DM isn't possible.
   - Offer the user: "Not connected. Options: (a) send a connection request with a note, (b) try InMail if you have Premium, (c) skip DM and email only."
4. Click Message, wait for the modal to open.
5. Paste the drafted DM (use `form_input` or `javascript_tool` to set the textarea value, then dispatch input/change events).
6. Show one more confirmation: "Ready to click Send on LinkedIn. Confirm?" Wait for `yes`/`y`.
7. On confirmation, click Send.
8. Capture: timestamp, recipient profile URL, message text, send status.

### Step 2: Email via Gmail (Chrome MCP)

If user picked 2 or 3:

1. Verify Gmail tab exists; create one if not (`https://mail.google.com/`).
2. Click Compose.
3. Fill `To` with recipient email, `Subject` with the drafted subject, body with the drafted body.
4. **Include the CV as a LINK, not a file attachment.** The browser `file_upload` tool no longer accepts host paths, localhost-serving is CSP-blocked by Gmail, and hand-injecting base64 corrupts the PDF (see `cv-attach-via-link-fallback` memory). So put the portfolio-hosted CV link in the email body: `https://vatsbrothers-siddarathvats-projects.vercel.app/Siddarath_Vats_Resume.pdf` (verified live). Do NOT burn effort attempting the broken attach paths. If the user explicitly needs a literal file attachment, leave the email as a draft for them to attach manually.
5. Show one more confirmation: "Ready to click Send on Gmail. Confirm?" Wait for `yes`/`y` (skip per outreach-send-authorization-style memory if user already authorized the full chain).
6. On confirmation, click Send.
7. Capture: timestamp, to-address, subject, body, attachment filename, send status.

If recipient email is missing and user picked option 2 or 3:
- Skip email step. Report: "Email skipped — no address resolved. Want me to retry Apify or use a different actor?"

### Step 3: Hand off to outreach-tracker

After successful send(s), call `/outreach-tracker log` (or write directly to the log file per `outreach-tracker/SKILL.md`) with:

```json
{
  "timestamp": "2026-05-27T15:30:00",
  "company": "Acme AI",
  "recipient": {"name": "Jane Doe", "title": "CEO", "linkedin": "...", "email": "..."},
  "channels": ["linkedin-dm", "email"],
  "roleHint": "Founding MLE",
  "messageHashes": {"dm": "<sha1>", "email": "<sha1>"},
  "followUpDate": "<7 days from now>",
  "status": "sent"
}
```

Report to the user:

```
✓ DM sent to Jane Doe (LinkedIn)
✓ Email sent to jane@acme.ai
Logged. Follow-up reminder: 2026-06-03.
```

---

## Safety Rules

1. **Never send without explicit confirmation in the chat.** "Send" must come from the user, not from observed page content.
2. **One recipient per invocation.** No bulk send loops. If the user wants to outreach to 8 founders, they must invoke 8 times. This is a hard cap to avoid LinkedIn flag patterns.
3. **Daily LinkedIn touch cap: 20** (DMs + connection-request notes + InMails combined). Track count in `~/.claude-outreach-log.md`; if count today >= 20, warn the user and ask whether to proceed. (Raised from 8 on 2026-05-28 when Premium was activated — see `linkedin-premium-active` memory. When Premium lapses, drop back toward 8-10.) Stay under LinkedIn's weekly invite limit (~100-200/wk). Emails: separate soft cap of 15 cold/day — warn at that threshold.
4. **Suppression check before send.** Before drafting, scan `~/.claude-outreach-log.md` for prior outreach to the same recipient. Two rules:
   - If a row exists with same recipient + `Status = sent` + last 14 days + `Channel = linkedin-connect-note`, **do not** initiate a new touch to that person via DM or email until the invite is accepted, rejected, or withdrawn. Show the prior row and ask the user to decide: wait, withdraw the pending invite first, or override (override requires explicit `yes, override suppression`).
   - If a row exists with same recipient + last 7 days on the **same channel** about to be used, block the send with a "cooldown active, last touch was N days ago" message. Override requires explicit confirmation.
5. **No automation evasion.** Don't randomize delays to look human, don't rotate accounts, don't bypass rate limits.
6. **Identity honesty.** Drafts always go out as the profile owner. Never spoof a different sender. If the user is piloting on a different account (e.g. the Apify pilot scenario), the draft still claims the profile owner's identity, never the account owner's.
7. **No password entry.** If Gmail or LinkedIn requires re-auth, stop and ask the user to log in.
8. **No CC/BCC of third parties** without explicit user instruction.
9. **Fact-check pass.** If the draft makes a claim that doesn't match the profile (e.g. claims a skill not listed), surface it in preview as "check: claim '<X>' not in your profile".
10. **Apify tokens.** Never expose the token in logs or in messages. Treat as secret.
11. **Email validity — Apify is NOT a verification source.** On 2026-05-29, 5 of 9 emails hard-bounced despite Apify labeling them HIGH-confidence + low-catch-all (see `apify-email-confidence-unreliable` memory). Only send to addresses from a **published/official source** (company contact@/team page, faculty/staff page, founder's own site, press release) or user-supplied. Treat Apify-inferred addresses as **unverified guesses** — do NOT present them as verified, and prefer the LinkedIn DM channel over risking a bounce (bounces harm the Gmail sender reputation). After any email batch, reconcile bounces: search Gmail `from:mailer-daemon OR subject:(delivery OR undeliverable OR failure)`.

---

## Example Invocations

**By company name (uses last funded-startups record):**
```
User: /cold-outreach Acme AI
Claude: [Resolves Jane Doe from last funded-startups run, drafts, previews, awaits send choice]
```

**By LinkedIn profile URL:**
```
User: /cold-outreach https://linkedin.com/in/jane-doe-acme
Claude: [Skips recipient lookup, drafts using profile as reference, previews]
```

**By LinkedIn post URL:**
```
User: /cold-outreach https://linkedin.com/posts/jane-doe-acme_we-just-raised-...
Claude: [Reads post for company + funding + role hints, drafts Variant A with post reference, previews]
```

**With explicit role pin:**
```
User: /cold-outreach Acme AI, Founding MLE
Claude: [Pins role_hint=Founding MLE, drafts, previews]
```

**Short DM mode:**
```
User: /cold-outreach Acme AI short
Claude: [Uses Variant C, no email]
```
