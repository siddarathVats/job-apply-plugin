# Cold Outreach Templates

These templates are the starting point. `/cold-outreach` personalizes every slot before showing the draft to the user. **Never send a template with unfilled `{slots}`.**

The validator strips em-dashes, en-dashes, and any phrase in the **Anti-AI tells** section below before preview. If you see one of those patterns in a draft, fix it before showing.

---

## Tone bedrock (applies to every variant)

1. **Never sound desperate.** Banned phrases (case-insensitive substring match, hard reject):
   - "I need a job", "I am looking for a job", "I need work", "I'm desperate", "any role", "any opportunity"
   - "I would be grateful", "I'd be so grateful", "thank you so much for your time" (in opener), "humbly request"
   - "Please consider", "please give me a chance", "I promise", "I won't let you down"
   - "I hope this finds you well", "Hope you are doing well", "Trust this email finds you well"
   - "I came across", "I stumbled upon", "I happened to see"
   - "I wanted to take a moment to", "I am writing to express my interest in"
   - **Research-skipping tells** (hard reject for service-pitch): "won't pretend to know", "won't pretend", "most founders", "most early-stage founders", "most teams", "I assume you have", "I'm guessing", "I bet you have", "I imagine you have"
2. **Lead with what you bring, not what you need.** Every opener references something specific about their company or post. Every middle paragraph names a concrete contribution.
3. **Length: 130 to 200 words for email.** Not 80, not 300. Tight enough to read in 20 seconds, dense enough to prove research.
4. **Closer depends on the step. First touch = NO meeting CTA.** In a connection-request note or first DM, do NOT ask for a coffee chat / call and do NOT end on what you build ("that's what I build" reads sales-y). Close **warm and curiosity-first, oriented toward THEM**: "Would love to connect and explore this with you", "Looking forward to connecting and learning more about what you're building." A single soft chat invite ("happy to chat about {their_specific_thing}, no pressure") is acceptable **only in a later-step email or a post-accept DM**, never in the first cold touch. Never two CTAs.
5. **DEEP RESEARCH IS MANDATORY before drafting (the single biggest reply-rate driver).** Research the *entirety* of the person, 4 layers in priority order: (1) their **product** (site, features, blog, launches, sandbox); (2) their **posts & comments**, their own LinkedIn/X activity (richest pain signal: echo their own words/stats back); (3) **them** (background, prior ventures, a genuine rapport hook); (4) their **team** if possible (co-founders/engineers → reveals stack + where help fits). Open the note on a pain only this research surfaces. Generic = ignored; deeply-researched + pain-first = instant replies (proof: Geir/DeepXL replied within hours).
6. **LinkedIn connection-request notes carry NO portfolio/work URL and NO meeting CTA.** The note's whole character budget goes to the researched pain-hook + a warm, curiosity-first close, never a link or a "let's chat" ask. The recipient sees the profile after accepting. Portfolio/CV links and chat invites belong only in **emails** and **post-accept DMs** (where there's room and they've already engaged). Do NOT spend note characters on `{github_or_portfolio_url}`.

---

## Anti-AI tells (validator strips/rewrites these before preview)

These patterns scream "LLM wrote this." Validator must catch and fix every one.

### Punctuation

| Banned | Replacement |
|--------|-------------|
| em-dash (U+2014) | period + capital letter, OR comma, OR `(...)` parens depending on context |
| en-dash (U+2013) | hyphen or period (same logic) |
| spaced hyphen used as a sentence break | comma or period |
| `; ` between independent clauses | period + new sentence |
| Triple-period `...` mid-sentence | comma or just remove |

### Lexical tells (hard ban in message bodies, case-insensitive)

```
delve, delving, delved
tapestry
landscape (as filler: "the AI landscape")
leverage (as verb)
synergy, synergies
furthermore, moreover, additionally
in today's fast-paced
seamlessly, robust, robustly
cutting-edge, state-of-the-art (unless quoting the company)
ecosystem (unless quoting the company)
intricacies, intricate
multifaceted
paradigm, paradigm shift
testament to
holistic
realm
underscore, underscores
foster (as verb meaning encourage)
```

### Structural tells

- Three-bullet "I bring X, Y, Z" lists with parallel verb starts (validator rewrites to vary sentence structure)
- Closing line "I look forward to hearing from you", replace with something specific
- Opening with "As a [role] with [years] years of experience...", replace with reference to their work
- Symmetric sentences ("Your X is impressive. My Y is impressive."), break the parallelism
- Numbered or bulleted lists in DM bodies, DMs are pure prose

### Rewrite rule

When the validator finds a tell, it does NOT just delete the word. It rewrites the sentence in a more concrete, lower-register voice. Example:

- Before: "I leverage cutting-edge transformer architectures to deliver robust solutions."
- After: "I work mostly on transformer fine-tuning, ablations and serving."

If the rewrite changes the meaning materially, flag for user review instead.

---

## LinkedIn DM Templates

DM character limits: free DM ~700 chars, connection-request note 300 chars, InMail body 1900 chars.

### Subject (InMail only)

```
{role_hint} interest, saw your {funding_or_topic_hint}
```

Filled: "Founding MLE interest, saw your Series A post" (no em-dash, no colon)

### Body, Variant A: Reference-the-post (highest reply rate)

Use when a specific post or announcement is the trigger. Free DM length budget.

```
Hi {first_name}, congrats on the {funding_stage}{funding_amount_clause}. The bit about {specific_detail_from_post} was the part that landed for me.

Short on me: {years} yrs as an MLE, {prev_company_1_one_line} and currently {current_focus_one_line} at {current_org}. Stack overlaps with what you're building: {top_3_skills}.

If the {role_hint} role is open, would love a quick call. Work and resume here: {github_or_portfolio_url}.{also_emailed_clause}

{sender_first_name}
```

Filled length target: 500 to 650 chars. If `also_emailed_clause` is non-empty, budget for an extra ~90 chars.

### Body, Variant B: No specific post (warm cold)

Use when you have the company and role but no post to reference.

```
Hi {first_name}, saw {company} is hiring for {role_hint} after the {funding_stage}. Short on me: {years} yrs MLE, {prev_company_1} plus {current_org}, focused on {current_focus}. One recent project worth a look: {one_project_one_line}.

Would 15 min next week make sense?{also_emailed_clause}

{sender_first_name}
{github_or_portfolio_url}
```

### Body, Variant C: Connection-request note (~200-300 char cap, pain-first, NO link, NO meeting CTA)

Use for the connection-request note specifically. The cap forces brevity, so every character earns its place: open on a *specific* pain/focus that the 4-layer deep research (bedrock rule 5) surfaced, then close **warm and curiosity-first**, no link, no coffee-chat/call ask, and never on "that's what I build" (Bedrock rules 2-3, 6). No generic "love to connect and trade notes."

```
Hi {first_name}, the {specific_pain_or_focus_from_their_post_or_product} side of {company} is exactly the kind of problem I find most interesting ({one_keyword_overlap}). Would love to connect and explore it with you.
```

Filled example (Geir/DeepXL style): "Hi Geir, the anticipate-not-just-detect angle on AI-manipulated docs is exactly the fraud problem I find most interesting (multimodal + eval). Would love to connect and explore it with you."

Rules for this variant:
- `{specific_pain_or_focus_from_their_post_or_product}` must come from real research (their post/comment/product), not a guess, echo their own framing where possible.
- NO `{github_or_portfolio_url}`, NO `{short_also_emailed_clause}` with a link, NO meeting CTA. Keep it pure: pain hook + warm curiosity-first close.
- Close oriented toward THEM ("Would love to connect and explore this", "Looking forward to connecting and learning more about what you're building"), never on what the sender builds/sells and never "let me show you my portfolio."

### Body, Variant E: Service-pitch with free POC (revenue / contract)

Use when `outreachType = service`. The sender is offering to build / automate / recreate something the poster has explicitly described as a pain. End-goal is a paid build; free POC is the wedge.

Job-pitch keywords (`hiring`, `role`, `position`, `opening`) are **forbidden** in this variant, validator strips them. Same with overt pricing talk (`$`, `rates`, `budget`, `quote`), also forbidden in first touch.

```
Hi {first_name}, your post on {pain_or_topic_from_post} landed for me. The bit about {specific_pain_quote} is exactly the kind of thing I build small AI / automation systems for.

What I'd bring: {one_relevant_capability_from_stack}. Recent shipped work: {one_concrete_evidence_with_outcome}.

Happy to scope a free POC focused on {narrow_subset_of_their_pain} so you can decide whether it's worth a longer conversation. No commitment.

{sender_first_name} | {github_or_portfolio_url}
```

Fits ~500-650 chars filled. The "free POC" line is the load-bearing wedge, never drop it.

### Body, Variant D: Partnership (no job pitch)

Use when `outreachType = partnership`. Job-pitch keywords (`hiring`, `role`, `position`, `opening`, `career`, `join your team`) are forbidden in this variant. Validator strips them.

```
Hi {first_name}, your post on {project_or_topic_specific} landed for me. I'm working on {sender_adjacent_project_one_line} and there's real overlap on {specific_overlap_point}.

Two things I could plug in on:
{capability_1}
{capability_2}

15 min to compare notes? No pitch, just curious if the threads connect.{also_emailed_clause}

{sender_first_name}, {github_or_portfolio_url}
```

(Note: bullet-style two lines are okay here only because Variant D is the partnership variant and reads like a builder-to-builder note. Other variants stay prose.)

---

## "Also emailed you" clause logic

Render `also_emailed_clause` only if **both**:
- Email channel is also being sent in the same `/cold-outreach` invocation
- Email address was sourced from a **public source** (company About page, AngelList profile, personal site, GitHub README)

If email came from Apify scraping or any non-public source, **omit the clause** to avoid signaling that you scraped their inbox.

### Phrasings (pick one, vary across recipients to avoid template smell)

- `Also sent a longer note to your inbox in case email is the better channel.`
- `Mailed a fuller version too, no pressure on which channel.`
- `Sent the longer version to your work email as well.`

For `short_also_emailed_clause` (300-char note):
- `Emailed a longer note too.`
- (omit entirely if budget tight)

If the user explicitly tells the skill in this session "don't mention email," skip the clause.

---

## Cold Email Templates

### Subject lines

Job outreach (pick one):
1. `{role_hint} at {company}, {sender_first_name} from {prev_company_or_school}`
2. `On {company}'s {funding_stage_or_recent_news}, MLE interest`
3. `{one_specific_skill_match} for {role_hint} at {company}`

Partnership outreach:
4. `Possible overlap, {specific_overlap_point}`
5. `{sender_first_name} on {project_or_topic_specific}, quick note`
6. `Your {project_or_topic_specific} post, builder note`

Service-pitch outreach (revenue / build-for-hire with free POC):
7. `On your {pain_or_topic} post, could build something for that`
8. `{one_relevant_capability} for the {pain_or_topic} thing you mentioned`
9. `Re your {pain_or_topic} post, free POC offer`

Rules:
- Under 60 chars
- No emojis
- No "Quick question", no "Re:" or "FW:" prefixes (deceptive)
- Use commas where the impulse is to use a dash
- Service-pitch subjects must reference the poster's pain, not the sender

### Body, Job variant (CXO-aware opener required)

**Opener must reference something specific about the company before introducing the sender.** Generic "I'm reaching out because..." openers are rejected by the validator.

Length target: 150 to 200 words.

```
Hi {first_name},

{cxo_opener_hook}

On where I'd contribute: {contribution_paragraph}.

Quick background. {years} years as an MLE. {prev_company_1} ({prev_company_1_one_line}), now at {current_org} ({current_focus_one_line}). The stack maps to what you're building: {top_3_skills_relevant_to_company}. Recent work I'd point to: {strongest_project_with_link_or_one_line_evidence}.

I'd value 15 minutes to ask how you're thinking about {topic_specific_to_their_roadmap_or_post}. Happy to send a longer brief first if that's easier.

Best,
{sender_full_name}
{linkedin_url}
{github_or_portfolio_url}
```

If body exceeds 220 words after filling, the validator trims the background paragraph first, never the opener or contribution paragraph.

### Body, Service-pitch variant (revenue / build-for-hire with free POC)

Use when `outreachType = service`. The sender is offering to build / automate / recreate something the poster has explicitly described as a pain. End-goal is paid work; free POC is the wedge that gets the call.

Length target: 140 to 190 words. **Forbidden keywords** (validator hard-rejects if found):
- Job-pitch: `hiring`, `role`, `position`, `opening`, `career`, `join your team`, `interested in your team`
- Pricing in first touch: `$`, `rates`, `quote`, `budget`, `proposal`, `SOW`, `retainer`, `per hour`, `per project`
- Overpromise: `guarantee`, `100%`, `silver bullet`, `transform your business`

```
Hi {first_name},

{service_pitch_opener_hook}

Here's the concrete piece I'd build first, free, to show fit:

{poc_scope_paragraph}

Why I think this'll land:
- {evidence_1: one-line shipped project with measurable outcome}
- {evidence_2: another shipped project, different angle}
- Stack I'd use here: {stack_for_this_poc}

If the POC is useful, we can talk about a full build after. If it isn't, you keep the POC code and we go our ways. Either way you don't pay for the trial.

15 min next week to scope what's actually painful?

Best,
{sender_full_name}
{linkedin_url}
{github_or_portfolio_url}
```

**Mandatory elements** (validator enforces):
- A concrete POC scope (not "I can help with stuff")
- At least one piece of evidence with measurable outcome (numbers if possible)
- The "free POC, you keep the code" wedge explicit
- A single 15-min CTA
- No pricing, no commitment ask

### Body, Partnership variant

Length target: 130 to 170 words. Forbidden keywords: `hiring`, `role`, `position`, `opening`, `career`, `join your team`.

```
Hi {first_name},

{partnership_opener_hook}

There's a real overlap on {specific_overlap_point}. Here's the piece of my work that maps:

{capability_paragraph}

What I'd value out of a 15-minute call: comparing notes on {topic_keyword}, and learning where {company}'s current approach is heading. Not pitching anything, just trying to figure out if it's worth a longer conversation.

Best,
{sender_full_name}
{linkedin_url}
{github_or_portfolio_url}
```

---

## CXO opener hook, construction rules

`{cxo_opener_hook}` is a 1 to 2 sentence opener that **must satisfy all four** of these before the validator clears the draft:

1. **References something concrete about the company.** Pulled from one of (in priority order):
   - The company's recent funding announcement / blog post / launch
   - The CXO's own recent post or interview
   - A specific product feature on the company website
   - Their public technical writing (blog, paper, podcast)
2. **References how the sender can contribute to that specific thing.** Not generic ("I'd love to help"), specific ("the data-quality work you mentioned at <event> overlaps with my MIMIC-III pipeline").
3. **No flattery without substance.** "Your work is amazing" is auto-rejected. "The argument in your <X> post that <Y> is what changed how I think about <Z>" passes.
4. **No claim of expertise the profile can't back.** Validator cross-checks claims against `workHistory` + `skills`.

### Opener hook examples (filled)

Good:
- `Your launch post made the case that eval-driven shipping beats model-bigger shipping, and that's the wall I've been bumping into on the MIMIC-III multimodal work at SMU.`
- `The Verdant Series A note mentioned tabular-plus-text fusion is the bottleneck. I shipped a tabular-plus-text model at Adani for solar plant performance and the same trade-offs showed up.`
- `Saw the Bolt founder post on agent eval, specifically the bit on graded difficulty. I built something adjacent at SMU and there are a few notes I'd want to compare.`

Bad (validator rejects):
- `I came across Acme AI and was very impressed by your work in the AI landscape.` (delving, landscape, generic)
- `Hope this finds you well. I am writing to express my interest in your company.` (banned opener)
- `Your work is amazing and I would love to contribute to your vision.` (flattery without substance)

### Partnership opener hook, same rules but partnership-framed

The hook must reference the *project or topic* in their post, not their *company* abstractly. And it must propose comparing notes, not joining the team.

---

## Personalization slot reference

| Slot | Source | Example |
|------|--------|---------|
| `{first_name}` | Founder/CXO record from `/funded-startup-jobs`, `/feed-scan`, or LinkedIn lookup | "Jane" |
| `{company}` | Company record | "Acme AI" |
| `{funding_stage}` | Company record `fundingStage` | "Series A" |
| `{funding_amount_clause}` | If amount known: ` ($15M)`; else empty | " ($15M)" |
| `{funding_or_topic_hint}` | Short topic for InMail subject | "Series A post" or "agent eval post" |
| `{role_hint}` | Best-match role title | "Founding MLE" |
| `{specific_detail_from_post}` | Up to 6 consecutive words from the post snippet | "graded difficulty in eval design" |
| `{years}` | Computed from profile workHistory | "3.5" |
| `{prev_company_1}`, `{prev_company_1_one_line}` | Profile workHistory[0] | "Adani Green" / "SVM/ANN/LSTM on GCP for ~9 GW renewable portfolio" |
| `{current_focus_one_line}` | Profile current role one line | "multimodal MIMIC-III readmission models on the SuperPOD" |
| `{current_org}` | Profile current company | "SMU" |
| `{top_3_skills}` | 3 from profile.skills matched to company keywords | "PyTorch, LangGraph, LLMs" |
| `{top_3_skills_relevant_to_company}` | Email body version, slightly longer | "PyTorch on long-context inputs, LangGraph for agent orchestration, MIMIC-III ETL" |
| `{strongest_project}`, `{strongest_project_with_link_or_one_line_evidence}` | One project with link or one-line proof | "multimodal MIMIC-III model, github.com/siddarathVats" |
| `{one_specific_skill_match}` | Single skill that strongly matches the role | "LangGraph" |
| `{cxo_opener_hook}` | Generated per Phase 3 of cold-outreach SKILL.md, must satisfy 4 rules above | "Your launch post made the case that eval-driven shipping beats model-bigger shipping..." |
| `{contribution_paragraph}` | 1 to 3 sentences, concrete contribution mapped to the opener hook | "The Adani DSM dashboard work pushed about a 10% penalty-cost reduction by closing the lookahead gap, and that's a similar shape to..." |
| `{topic_specific_to_their_roadmap_or_post}` | Specific thing they've publicly committed to working on | "the move from rule-based eval to learned eval" |
| `{partnership_opener_hook}` | Partnership-flavored hook, same 4 rules | "Your post on graded difficulty in agent eval is exactly the wall I'm hitting on..." |
| `{capability_1}`, `{capability_2}`, `{capability_paragraph}` | Partnership only, concrete things the sender can plug in on | "running ablations on tabular plus text fusion" / "GPU cluster orchestration on SLURM" |
| `{topic_keyword}` | Single keyword tying the two threads | "multimodal eval" |
| `{also_emailed_clause}` | Computed by skill, see logic above | "Also sent a longer note to your inbox in case email is the better channel." or empty |
| `{github_or_portfolio_url}` | Profile.githubUrl or portfolioUrl | "github.com/siddarathVats" |
| `{linkedin_url}` | Profile.linkedInUrl | (link) |
| `{sender_first_name}`, `{sender_full_name}` | Profile.firstName / firstName+lastName | "Siddarath" / "Siddarath Vats" |

### Service-pitch only slots

| Slot | Source | Example |
|------|--------|---------|
| `{pain_or_topic_from_post}` | Service only, the specific pain or workflow the poster complained about | "PV engineers stuck in 2015 tooling" |
| `{specific_pain_quote}` | Service only, up to 8 consecutive words quoted from the post | "productivity ceiling I'm actively trying to break" |
| `{one_relevant_capability_from_stack}` | Service only, the single best-match piece of Siddarath's stack for this pain | "LangGraph multi-agent orchestration with tool-use" |
| `{one_concrete_evidence_with_outcome}` | Service only, a real shipped project with a measurable outcome | "Adani solar PR uplift ~2% via SVM/ANN/LSTM models on GCP" |
| `{narrow_subset_of_their_pain}` | Service only, a thin slice of the pain that can be POC'd in ~1 week | "the input-change re-run problem in PVsyst, scripted in Python" |
| `{service_pitch_opener_hook}` | Service only, 1-2 sentences referencing the pain + how the sender can build for it | "Your post on PV simulation tooling stuck in 2015 lined up with exactly the kind of agentic-pipeline work I just shipped for an alumni RAG platform." |
| `{poc_scope_paragraph}` | Service only, 2-3 sentences describing concrete POC deliverable + timebox | "I'd take one common SAM/PVsyst run, wrap it in a Python pipeline that re-executes on any input change, and ship a tiny dashboard. ~1 week, you keep the code." |
| `{evidence_1}`, `{evidence_2}` | Service only, two distinct shipped projects with outcomes | "Re-architected EZER vector search from OpenSearch to pgvector, sub-second similarity at lower infra cost" |
| `{stack_for_this_poc}` | Service only, the specific stack pieces the sender would use for this POC | "Python, LangGraph, FastAPI, pgvector" |
| `{poc_timebox}` | Service only, the explicit time commitment for the free POC | "~1 week" |

---

## Anti-pattern hard rejects (validator stops the draft, never silently fixes)

These conditions stop the draft and surface the issue to the user, because silently fixing changes meaning:

- Sender claims a skill or domain not in profile
- Sender claims a specific number (e.g., "shipped to 1M users") not present in profile
- Draft mentions compensation in the first touch
- Draft cc's investors, recruiters, or third parties
- Draft proposes an intro to another person ("could you intro me to your CTO?") in the first touch
- Draft attaches the resume as a file (resume must be linked, never attached)
- Same recipient already sent this week per `/outreach-tracker` log (suppression rule)

## Anti-pattern soft fixes (validator rewrites silently)

These get fixed without bothering the user:

- Em-dashes, en-dashes, hyphen-as-break
- Banned phrases from the desperation list
- AI lexical tells
- Three-bullet parallel-verb lists (rewritten to vary structure)
- Closing "I look forward to hearing from you" replaced with topic-specific close
- Multiple CTAs reduced to one
- Over-length drafts trimmed (background paragraph first, never opener or contribution)
