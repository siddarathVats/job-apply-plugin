# Deep-Research Dossier Process (the standard approach)

This is the mandated research method before ANY outreach (user directive 2026-05-29). Depth of research is the single biggest reply-rate driver — the deeply-researched, pain-first messages get instant replies (Geir/DeepXL replied within hours; generic notes get ignored). **Do not rush — be exhaustive.** Build one dossier per target, write it to a file, end with the single strongest hook.

Output: `outreach-dossiers/<lastname-company>.md` (one file per target). Narrate progress so the user can follow.

---

## Phase 1 — Full profile capture (logged-in browser)
Open the profile. Expand every control ("see more" on headline/About, "Show all N experiences/education/skills/recommendations") — or JS-extract the underlying text. Capture:
- Headline, **About text (HIGH PRIORITY — often the clearest first-person pain statement; mine it for the opener)**, every role (title, company, dates, location, description), education, certifications, skills, recommendations, languages, volunteering, awards.
- **Signals:** how they describe themselves, what they emphasize/repeat, their writing voice, any published email in the About.

## Phase 2 — Recent activity (highest-signal — go slow)
Open their Activity tab (`/in/<slug>/recent-activity/all/`, `/comments/`). Review ~20–30 recent items:
- **Posts:** topic, angle, what they promote or are frustrated by.
- **Comments:** what they engage with, who they talk to, their opinions.
- **Likes/reactions:** recurring themes, whose content resonates.
For each notable item: date + 1-line paraphrased gist (don't copy long passages — copyright) + why it matters for outreach. Pull out 2–3 things they care about *right now*.

## Phase 3 — Company & product deep-dive (primary sources, not just LinkedIn)
- What the product actually does, ICP, positioning; pricing/packaging if public; key features; recent launches/changelog.
- Stage signals: funding, headcount trend, hiring, recent press.
- Competitors and where this product sits.
- Read 2–3 of the company's own recent posts/blog/announcements.
- **Verify the person matches the company** (wrong-entity guard — cf. the FluxAI mismatch where a name mapped to the wrong domain).

## Phase 4 — Synthesis → dossier file
Write `outreach-dossiers/<lastname-company>.md` using the template below; fill every field, use "not found" where unverified. End with the **single strongest, most specific hook** (something only true of THEM) + a confidence note (high/med/low), then the **connect note**.

---

## Dossier template
```
# {Name} — {Title}, {Company}
Profile: <url> | Researched: <date> | Confidence: high/med/low

## Snapshot
- One line: who they are + why they matter to {YOUR_OFFER}.

## Profile
- Headline:
- About (key points, paraphrased):
- Current role + what they own:
- Career arc (notable past roles):
- Education / credentials:
- Tells (voice, recurring themes):

## What they care about right now (from recent activity)
- Theme 1 — evidence (post/comment/like, date)
- Theme 2 — evidence
- Theme 3 — evidence
- Public positions:

## Company / product
- What it does:
- ICP:
- Stage & momentum (funding, hiring, launches, press):
- Notable recent moves:
- Competitive context:

## Outreach angle
- Strongest personalized hook (specific to them):
- What NOT to say (sensitivities, things they've criticized):
- Relevance of {YOUR_OFFER}:

## Team members worth reaching (per {ICP})
- Name — title — why them — profile url

## Strongest hook + confidence
- Hook (only-true-of-them):
- Confidence:

## Connect note (pain-first, NO link, soft CTA)
- <draft>
```

---

## The note that comes out of the dossier
- **Pain-first opener** from the research (ideally echo their own About line / their own post words).
- **One specific, only-true-of-them detail** — names their exact problem, not a category.
- **NO portfolio/work URL** in a connection note (save space; link belongs in emails/post-accept DMs).
- **CTA = soft, casual: a coffee chat OR just connect** — "down for a coffee chat about that?", "would love to connect and chat about {their_thing}". Never a hard sales ask. (User directive: include the coffee-chat / just-connect option.)
- Keep within the note cap (~200 free / 300 Premium chars).

## Email addresses
Only email a **published/official** address (the About section, company contact/team page, faculty page). Treat Apify-inferred emails as **unverified guesses** — 5/9 bounced on 2026-05-29 (see `apify-email-confidence-unreliable` memory). Prefer LinkedIn over a bounce.
