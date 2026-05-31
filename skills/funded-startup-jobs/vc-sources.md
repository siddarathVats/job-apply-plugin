# VC & Accelerator Sources — Curated List

Used by `/funded-startup-jobs`. Keep this file the single source of truth for VC handles and job-board URLs so the skill stays current without code edits.

---

## Tier 1 — VC Job Boards (web pages, scrape via Playwright/WebFetch)

| VC | Job Board URL | Notes |
|----|---------------|-------|
| a16z (Andreessen Horowitz) | https://portfoliojobs.a16z.com/jobs | Full portfolio jobs board |
| a16z Speedrun | https://a16z.com/speedrun/ | Games/consumer track |
| Y Combinator | https://www.workatastartup.com/jobs | YC company jobs (login optional) |
| Sequoia Capital | https://www.sequoiacap.com/jobs/ | Talent network |
| Index Ventures | https://www.indexventures.com/jobs/ | |
| General Catalyst | https://jobs.generalcatalyst.com/ | |
| Khosla Ventures | https://jobs.khoslaventures.com/ | |
| Greylock Partners | https://jobs.greylock.com/jobs | |
| Kleiner Perkins | https://jobs.kleinerperkins.com/ | |
| BITKRAFT Ventures | https://jobs.bitkraft.vc/jobs | Gaming/esports focus |
| Accel | https://jobs.accel.com/ | |
| Contrary | https://contrary.com/jobs | Younger founders, ML-heavy portfolio |
| Pear VC | https://jobs.pearvc.com/jobs | |
| Battery Ventures | https://jobs.battery.com/ | |
| NEA (New Enterprise Associates) | https://jobs.nea.com/ | |
| Antler | https://jobs.antler.co/ | |
| Lightspeed Venture Partners | https://jobs.lsvp.com/jobs | |
| Bessemer Venture Partners | https://jobs.bvp.com/jobs | |
| Founders Fund | https://foundersfund.com/portfolio-jobs/ | |
| First Round Capital | https://jobs.firstround.com/ | |
| Benchmark | https://www.benchmark.com/jobs/ | |
| Coatue | https://jobs.coatue.com/ | |
| Insight Partners | https://insightpartnersjobs.com/ | |

---

## Tier 2 — VC LinkedIn Handles (scroll feed via Chrome MCP)

Use `https://www.linkedin.com/company/<handle>/posts/` to read recent posts. Look for posts with the patterns: "we just raised", "is hiring", "portfolio company X raised", "join <company>".

| VC | LinkedIn handle |
|----|-----------------|
| a16z | andreessen-horowitz |
| Sequoia Capital | sequoia-capital |
| Index Ventures | index-ventures |
| General Catalyst | general-catalyst-partners |
| Khosla Ventures | khosla-ventures |
| Greylock Partners | greylock-partners |
| Kleiner Perkins | kleiner-perkins |
| BITKRAFT Ventures | bitkraft-ventures |
| Accel | accel-partners |
| Y Combinator | y-combinator |
| Contrary | contrarycap |
| Pear VC | pear-vc |
| Battery Ventures | battery-ventures |
| NEA | new-enterprise-associates |
| Antler | antlerglobal |
| Lightspeed | lightspeed-venture-partners |
| Bessemer | bessemer-venture-partners |
| Founders Fund | founders-fund |
| First Round | first-round-capital |
| Benchmark | benchmark |

---

## Tier 3 — Partner LinkedIn Profiles (often share hiring threads from portfolio cos)

Add as you discover them. Example seeds:

| Name | Firm | Handle |
|------|------|--------|
| Marc Andreessen | a16z | mandreessen |
| Sarah Tavel | Benchmark | sarahtavel |
| Elad Gil | Solo investor | eladgil |
| Garry Tan | YC | garrytan |
| Eric Glyman | Ramp / angel | ericglyman |

Partner handles change. Verify before adding.

---

## Funding-news search patterns (LinkedIn keyword search)

LinkedIn `https://www.linkedin.com/search/results/content/?keywords=<URL-encoded>&datePosted=<range>` with these query templates:

```
"just raised" AND ("Series A" OR "Series B" OR "seed") AND ("hiring" OR "we're hiring" OR "join us")
"announcing our" AND ("Series A" OR "Series B") AND hiring
"we raised" AND ("hiring" OR "engineer" OR "ML")
```

`datePosted` values: `past-24h`, `past-week`, `past-month`.

---

## Crunchbase / WebSearch fallback queries

When LinkedIn rate-limits or returns thin results, fall back to WebSearch:

```
"raised Series A" "AI" "hiring" {month} {year} -site:linkedin.com
"announces $XXM Series B" "machine learning" {month} {year}
site:techcrunch.com "raised" "AI" {month} {year}
site:crunchbase.com "Series A" AI {month} {year}
```

Use these to surface companies, then visit their careers page to find roles.

---

## Accelerators / residencies (portfolio + demo-day = founder sourcing)

From a LinkedIn list of "45 Startup Accelerators" (GRG Gen Nxt, 2026-05). Mine each one's portfolio/demo-day/jobs page for recently-funded startups, then outreach the founders. These are SOURCING channels, not pitch targets.

| Accelerator | Sourcing URL | Notes |
|----|----|----|
| Y Combinator | workatastartup.com / ycombinator.com/companies | Largest; filter by recent batch (W26/S26) |
| Sequoia Arc | sequoiacap.com/arc | Pre-seed/seed residency |
| a16z Speedrun | a16z.com/speedrun | Games/consumer/AI |
| Antler | antler.co/portfolio | Global day-zero |
| Techstars | techstars.com/portfolio | Many city/vertical tracks |
| 500 Global | 500.co/companies | Global seed |
| HAX | hax.co | Hard-tech / robotics (eng-heavy) |
| NFX FAST | nfx.com | Seed; fast-decision program |
| South Park Commons | spc.com | Founder fellowship |

Action: feed these to `/funded-startup-jobs vc-boards` (or paste a specific demo-day post) to pull founder targets, then `/cold-outreach` each.

## Maintenance

- When a VC's job-board URL 404s, update this file before re-running the skill.
- Add new VCs as the user surfaces them via shared LinkedIn posts.
- LinkedIn handles drift — if a feed returns the company page instead of posts, verify the handle still resolves.
