---
name: outreach-tracker
description: Log sent cold-outreach messages, list pending follow-ups, mark replies, and set reminders. Use when the user wants to see who they've reached out to, who hasn't replied, or wants to log a manually-sent message.
allowed-tools: Read, Write, Bash
---

# Outreach Tracker

Owns the outreach log at `~/.claude-outreach-log.md`. Every send by `/cold-outreach` appends here; the user can also query it directly.

---

## Log file location & schema

**Path**: `~/.claude-outreach-log.md`

**Format**: Markdown with a single table. Each row is one outreach event (one channel per row, a "DM + email" send produces two rows). Append-only; never rewrite history except for the "Reply received" / "Status" columns.

### Schema

```markdown
# Outreach Log

| ID | Date | Company | Recipient | Title | Channel | Role hint | Status | Follow-up | Reply | Notes |
|----|------|---------|-----------|-------|---------|-----------|--------|-----------|-------|-------|
| 001 | 2026-05-27 | Acme AI | Jane Doe | CEO | linkedin-dm | Founding MLE | sent | 2026-06-03 |, | reference: a16z post |
| 002 | 2026-05-27 | Acme AI | Jane Doe | CEO | email | Founding MLE | sent | 2026-06-03 |, | jane@acme.ai |
```

**Status values**: `sent` | `replied` | `bounced` | `no-reply` (auto-set after 14 days from follow-up with no `replied` flag) | `meeting-set` | `dead`

**Channel values**: `linkedin-dm` | `linkedin-inmail` | `linkedin-connect-note` | `email` | `twitter-dm` | `other`

**ID**: zero-padded 3-digit, monotonic.

---

## Commands

### `log`, append a new row

Invoked by `/cold-outreach` after a successful send, or manually by the user.

Required fields: `Date`, `Company`, `Recipient`, `Channel`, `Status`.

Optional fields: `Title`, `Role hint`, `Follow-up` (default: send-date + 7 days), `Notes`.

Compute next ID by counting existing rows + 1.

```
User: /outreach-tracker log Acme AI, Jane Doe, CEO, linkedin-dm, Founding MLE
Claude: Logged row 003. Follow-up: 2026-06-03.
```

### `list`, show pending or all

Default: show rows with `Status = sent` and `Follow-up <= today + 3` (i.e. upcoming/overdue follow-ups).

Filters supported:
- `list all`, every row
- `list pending`, `Status` in {sent, no-reply}
- `list replied`, `Status` in {replied, meeting-set}
- `list overdue`, `Status = sent` and `Follow-up < today`
- `list today`, `Follow-up = today`
- `list company <name>`, all rows for that company

Output format:

```
Outreach, pending / overdue
─────────────────────────────────────────────────────
ID  | Date       | Company        | Recipient   | Channel | Follow-up   | Days
003 | 2026-05-20 | Acme AI        | Jane Doe    | dm      | 2026-05-27  | TODAY
004 | 2026-05-18 | Bolt Robotics  | Sam Ko      | email   | 2026-05-25  | OVERDUE -2
005 | 2026-05-22 | Verdant ML     | Lin Park    | dm+email| 2026-05-29  | +2
─────────────────────────────────────────────────────
3 actions due. Run `/outreach-tracker followup <ID>` to draft a follow-up.
```

### `reply`, mark a row as replied

```
User: /outreach-tracker reply 003
Claude: Mark 003 as replied. Set status to one of: replied / meeting-set / dead?
User: meeting-set, scheduled for Friday
Claude: Updated row 003. Status: meeting-set. Note: "scheduled for Friday".
```

### `followup <ID>`, hand off to /cold-outreach with prior context

1. Read row by ID.
2. Read the original `messageHashes` (stored in a sidecar `~/.claude-outreach-log/messages/<ID>.json`) so the follow-up doesn't repeat content.
3. Hand off to `/cold-outreach` with `follow-up-mode` set: drafts a shorter "bumping this in case it got buried" message that references the original send without quoting it.

### `daily`, yesterday snapshot

The single most useful command for outreach hygiene. Run every morning to see what happened the previous day across all channels.

#### Flow

1. Compute `target_date`. Default: yesterday (today minus 1 day). Override via `daily 2026-05-25` or `daily today`.
2. Read `~/.claude-outreach-log.md`. Filter rows where `Date = target_date`.
3. For each LinkedIn channel row (`linkedin-dm`, `linkedin-connect-note`, `linkedin-inmail`):
   - Determine current status by combining: log row + LinkedIn-side check (next step).
4. **LinkedIn acceptance detection.** For rows with `Channel = linkedin-connect-note` and `Status = sent`:
   - Use Chrome MCP to navigate to `https://www.linkedin.com/mynetwork/invitation-manager/sent/`.
   - Capture every recipient still listed (these are pending or unaccepted).
   - For each `connect-note` row in the log: if the recipient's LinkedIn URL is **NOT** in the still-pending list, AND the recipient now appears in `https://www.linkedin.com/mynetwork/grow/recently-added/` (last 7 days), mark `Status = replied` (accepted = positive reply for invites).
   - If recipient's LinkedIn URL is NOT in still-pending AND NOT in recently-added, it was rejected or withdrawn. Mark `Status = no-reply` with a `Notes` annotation: "vanished from sent + not in recently-added; likely rejected/withdrawn".
5. **Email reply detection (heuristic, best-effort).** For rows with `Channel = email` and `Status = sent`:
   - Use Chrome MCP to read Gmail's Inbox tab.
   - Search for the recipient email address as sender in the last 14 days (`from:<recipient_email>`).
   - If a thread exists where the recipient replied AFTER the send timestamp, mark `Status = replied`. Notes get the reply subject line.
   - This is a heuristic, not exact. The user can override via `reply <ID>`.
6. Write status updates back to `~/.claude-outreach-log.md` (mutate only the `Status`, `Notes` columns).

#### Output

```
Outreach Daily, 2026-05-26 (yesterday)
─────────────────────────────────────────────────────────────────

LinkedIn invitations
  Sent:           4
  Accepted:       1   (Jane Doe, CEO Acme AI)
  Still pending:  2   (Sam Ko, Lin Park)
  Rejected/gone:  1   (Alex Romero, no longer in sent list, not in recently-added)

LinkedIn DMs (1st-degree)
  Sent:           2
  Replied:        1   (Priya Shah, "Let's chat Thursday")
  No reply yet:   1

Cold emails
  Sent:           3
  Replied:        2   (1 positive: Verdant ML; 1 polite no: Bolt Robotics)
  No reply yet:   1   (Foundry CTO)

Aggregate
  Total touches: 9
  Reply rate (so far): 4/9 = 44%
  Follow-up due today: 0
  Follow-up due tomorrow: 3

Suppression notes
  - 2 recipients have pending invites >21 days. Run `withdraw-stale` to clean up.
  - 0 recipients in 7-day same-channel cooldown.
─────────────────────────────────────────────────────────────────
```

#### Variants

- `daily today` → same view but for today's sends (less interesting since nothing has time to respond)
- `daily 2026-05-20` → specific date
- `daily week` → roll up across the last 7 days, group by date

#### Why this matters

This is the data that drives every other tracker decision:
- If yesterday's invite acceptance is 0/4, today's outreach should slow down or change approach.
- If a recipient is logged as "rejected/gone" the suppression rule auto-blocks new touches to that person for 90 days unless overridden.
- If reply rate is trending down week-over-week, the user should reconsider the template.

### `bulk-status`, sweep stale rows

Run on demand. For every row with `Status = sent` and `Follow-up < today, 14 days`, set `Status = no-reply`. Report count.

```
User: /outreach-tracker bulk-status
Claude: 3 rows aged out to no-reply. Run `/outreach-tracker list pending` to see.
```

### `withdraw-stale`, LinkedIn invite hygiene

Sweeps pending LinkedIn connection invitations that have aged out, so your invite acceptance rate doesn't tank.

**Why this matters**: pending invites that sit > ~3 weeks unaccepted (and never explicitly rejected) still count against your LinkedIn acceptance rate. Once that rate drops below ~40-50%, LinkedIn throttles your weekly invite limit and may eventually require email-verified invites, which kills cold outreach entirely.

#### Flow

1. Use Chrome MCP to navigate to `https://www.linkedin.com/mynetwork/invitation-manager/sent/`.
2. Verify logged in. If not, stop and instruct user to log in.
3. Use `read_page` to enumerate every pending sent invitation. For each, capture:
   - Recipient name + LinkedIn URL
   - Recipient headline
   - "Sent X weeks/months ago" → convert to days
4. Filter by `--older-than <N>` flag (default 21 days). Argument can be passed inline:
   - `/outreach-tracker withdraw-stale` → 21 days
   - `/outreach-tracker withdraw-stale 14` → 14 days
   - `/outreach-tracker withdraw-stale 30` → 30 days
5. Show the user a preview table:

```
Sent invitations older than 21 days
─────────────────────────────────────────────────────────────
#  | Sent       | Recipient        | Headline                    | Age
1  | 2026-04-15 | Jane Doe         | CEO at Acme AI              | 42d
2  | 2026-04-22 | Sam Ko           | Founder @ Bolt Robotics     | 35d
3  | 2026-05-01 | Lin Park         | Co-founder, Verdant ML      | 26d
4  | 2026-05-03 | Alex Romero      | Head of AI, Foundry         | 24d
─────────────────────────────────────────────────────────────
4 invites would be withdrawn.

Options:
  a) Withdraw all 4
  b) Withdraw selected (e.g. "1,3" or "1-3")
  c) Skip, review manually
  d) Cancel
```

6. **Wait for explicit user choice.** Never auto-withdraw.
7. On confirmation, for each row to withdraw:
   - Find the "Withdraw" button next to that invite.
   - Click Withdraw.
   - Confirm the LinkedIn modal ("Withdraw invitation? Yes").
   - **1.5-2 second delay** between withdrawals to avoid burst rate-limit.
   - If LinkedIn shows a confirmation toast, count as success; otherwise mark failed and continue.
8. Report:

```
✓ Withdrew 4 stale invitations (0 failed).
Acceptance ratio expected to improve once the dust settles (~24-48h).
```

9. Log the withdrawal sweep to `~/.claude-outreach-log/withdraw-history.md`:

```markdown
| Date       | Threshold | Withdrawn | Failed |
|------------|-----------|-----------|--------|
| 2026-05-27 | 21d       | 4         | 0      |
```

This log is **only** the sweep summary, recipient names are NOT stored (no reason to keep that data once the invite is gone).

#### Safety

- **Hard cap 50 per sweep.** If more than 50 invites match, ask the user to narrow the threshold or do it in batches. LinkedIn rate-limits bulk withdrawals.
- **Never withdraw an invite the user explicitly told you to send.** If the recipient appears in `~/.claude-outreach-log.md` with `Channel = linkedin-connect-note` and `Status = sent`, mark it but require extra confirmation: "This was a tracked outreach send, withdraw anyway?"
- **No auto-mode.** Even with `--auto` flag (don't add one), the skill must show the preview first.

### `check-suppression`, pre-send guard, called by /cold-outreach

Internal command used by `/cold-outreach` before drafting. Returns one of:

- `clear`, no prior outreach to this recipient, proceed.
- `pending-invite:<row_id>`, a LinkedIn connect-note to this person is still pending (sent <= 14 days ago, still in their sent invitations). Block new touches via any channel.
- `recent-touch:<row_id>`, same channel was used in the last 7 days. Block this channel; other channels still open.
- `rejected-recently:<row_id>`, their invite vanished from sent and they're not in recently-added (likely rejected/withdrawn). Block new touches for 90 days unless overridden.
- `auto-no-reply:<row_id>`, they've been marked `no-reply` after the standard 14-day post-followup window. New touches allowed but the user should know.

Input: recipient name + LinkedIn URL + email + channel about to be used.

Returns the status enum plus the conflicting row ID if any. `/cold-outreach` decides what to do (most cases: block, surface row to user, require explicit override).

This command does not modify the log. It's a read-only query.

### `stats`, quick metrics

```
User: /outreach-tracker stats
Claude:
  Total sent: 12 (last 30 days)
  By channel: linkedin-dm 7, email 5
  Reply rate: 25% (3 replied / 12 sent)
  Meeting set: 1
  Pending follow-up: 4
  Overdue: 2
```

---

## Sidecar message storage

For each row, store the actual message text at `~/.claude-outreach-log/messages/<ID>.json`:

```json
{
  "id": "003",
  "channel": "linkedin-dm",
  "to": "linkedin.com/in/janedoe",
  "subject": null,
  "body": "Hi Jane, congrats on...",
  "sentAt": "2026-05-27T15:30:00",
  "referencePost": "https://linkedin.com/posts/...",
  "draftVariant": "A"
}
```

Used by `followup` so the next message doesn't accidentally repeat the same opener.

---

## Safety Rules

1. **Append-only for sends.** Never rewrite a row's `Date`, `Recipient`, or `body`. Only the `Status`, `Follow-up`, `Reply`, and `Notes` columns can be updated post-hoc.
2. **No deletes from chat.** If the user says "delete row 003", refuse and explain: change `Status` to `dead` instead. Manual file edit allowed if they really need it.
3. **Email addresses are stored in plaintext.** Warn the user once that this file is unencrypted; recommend file permissions tightening.
4. **No external sync.** This file never leaves the local machine via this skill.
5. **PII in notes**: the `Notes` column may contain personal info from replies. Treat the file as sensitive.
6. **Acceptance detection is read-only.** The `daily` command navigates LinkedIn but never clicks anything beyond what's needed to read sent-invitations and recently-added pages.
7. **Gmail reading is read-only.** Reply detection reads inbox listings only. Never opens, marks-as-read, or interacts with messages beyond visibility checks.
8. **Suppression overrides require explicit confirmation.** `/cold-outreach` cannot bypass `check-suppression` results without the user typing `yes, override suppression` (exact string match).

---

## Initial setup

On first invocation, if `~/.claude-outreach-log.md` doesn't exist, create it with the header and an empty table. Create `~/.claude-outreach-log/messages/` directory.

---

## Example Invocations

```
User: /outreach-tracker daily
Claude: [Shows yesterday's snapshot: sent / accepted / replied across LinkedIn invites, DMs, and emails]

User: /outreach-tracker daily week
Claude: [Roll-up of the last 7 days grouped by date]

User: /outreach-tracker list
Claude: [Shows 3 pending/overdue rows with action prompts]

User: /outreach-tracker withdraw-stale
Claude: [Sweeps LinkedIn invites > 21 days unaccepted, previews, awaits confirmation]

User: /outreach-tracker stats
Claude: [Prints summary]

User: /outreach-tracker reply 004, they want to schedule a call
Claude: [Marks row 004 as meeting-set, notes the reason]

User: /outreach-tracker followup 002
Claude: [Reads row 002 + sidecar, hands off to /cold-outreach with follow-up framing]
```
