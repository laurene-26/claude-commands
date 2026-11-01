---
description: End-of-day client status update in the team standard format (team standard). Updates the client hub, memory and the csm-hub project's health first, then leaves a Slack draft for review. Linear is read-only. Usage: /eod <project>
argument-hint: <project> (e.g. acme, globex, initech)
---

You are wrapping today's work on the project named in $ARGUMENTS.

First, from THIS conversation, gather the raw material: what actually happened, what moved, what is stuck. If little happened, say so rather than padding.

---

# Part A: the record (hub, memory, csm-hub)

This part is Shai's own project history. It is NOT part of the team's shared EOD skill.
If you ever want only the team standard, delete Part A and keep Part B.

## Resolve the project

- Hub: does `~/Projects/Client Hub Shai/data/clients/<slug>/` exist? If not, skip step 1.
- Memory file: `find ~/.claude/projects -name "*.md" | grep -i <project>`.
- csm-hub project: resolve by name via `projects_get` (id, slug, or an exact/unique name all
  work). This is the write target for A3.
- Linear project, read-only: `reference_linear.md` (or `list_projects` by name, CX team). Used
  only in Part B, to pull R&D blocker context for the Slack draft, never written to. See A3.
- Slack channel: project memory, else `slack_search_channels`.

## A1. The hub (tracked engagements only)

Update only what changed today. Do not invent progress.

- `log.json`: a dated entry per decision and per handover-relevant note.
- `gates.json`: status changes, `passed_on`, signatories, evidence. Use `passed` / `failed`.
- `compliance.json`: status, `actual_date`, `artifact`, `owner_id` when a role gets a name.
- `testing.json`: level sign-offs, the daily defect classification, conduct-standard results.
- `exposure.json`: current step, advance conditions met, freeze state and reason.
- `risks.json` / `stakeholders.json`: new entries, status changes, names.

**Recording slip.** If a dated item moved, set `slip_days`, `slip_cause` (`client` / `delivery` /
`external`), `slip_reason` and `slip_recorded_on`, then move `current_date` by that many business
days. Never touch `planned_date`. Use `src/lib/calendar.mjs` for the arithmetic.

## A2. Memory

Search first so you UPDATE rather than duplicate. Append a dated entry: what got done, decisions,
new IDs and assets, what is next, open items. Absolute dates. Specific enough to resume cold.
Update the parent `MEMORY.md` line if the headline changed.

For tracked engagements, keep memory at the level the hub does not cover: judgement and context,
why a decision went the way it did. Do not duplicate gate and compliance tables into memory.

## A3. csm-hub (project health)

Log today's health on the csm-hub project (`projects_log_status`): `status` `green` / `yellow` /
`red`, plus a `note` carrying the manager-facing write-up, what changed, current state, what is
next, risks or decisions needed. Pull the project's recent health history first (`projects_get`)
for an honest diff rather than repeating yesterday's line.

Derive the flag from state rather than feel: an overdue gate or an unowned blocker is `yellow` at
minimum; a failed gate or frozen exposure is `red`. Say which fact drove it.

**Linear is read-only, as of now.** Part B still reads it (`list_issues`) for R&D blocker context
in the Slack draft. Do not post status updates, comments, or anything else back to Linear until
this changes.

---

# Part B: the Slack draft, team standard

Reproduced from the team EOD skill shared with the FDE squad. **Keep this
section in sync with that skill rather than improving it locally.** Matching the team matters more
than any local preference.

## The template

Repeat the whole block once per flow. "Flow" means a distinct agent or workstream for that client.
A client with a chat bot and an IVR has two flows.

```
*<Client> — EOD <DD.MM>*

*<Flow name>*
- Go-live: <date, or TBD>
- Exposure: <rate, or "not live">
- Deflection / success: <rate, or "not live">
- Blocked — R&D: <one line> — <owner>, ETA <date>
- Blocked — FDE: <one line> — <owner>, ETA <date>
- Blocked — client: <one line> — <owner>, ETA <date>
- Next touchpoint: <meeting or checkpoint, with date>
```

## Rules that make or break this update

**One line per bullet.** The team's rule: "better to pass 80% of the
idea than people reading 20% of what you aimed for." If a bullet needs a paragraph, it needs a
Linear ticket instead.

**R&D blockers link a Linear ticket rather than explaining themselves.** The ticket carries the
detail; the bullet carries the ETA and the owner.

**Every blocker names an owner and an ETA.** A blocker with neither is not a blocker, it is a
complaint. If the ETA is genuinely unknown, write "ETA unknown" and name who owes it. That absence
is the most useful thing in the post.

**Never invent a number.** Exposure and deflection are reported only when the flow is live and the
figure was actually measured. Write "not live" or "first data expected <date>" instead of a guess.
A fabricated deflection rate is worse than a blank one.

**Put the go-live date in writing even when it is uncomfortable.** If there are two dates in play,
a formal one and one the team is pushing for, state both. That gap is what the template exists to
surface.

**Empty blocker lines are good news, not omissions.** Write "none" rather than dropping the bullet,
so the reader can tell the difference between nothing-blocked and not-checked.

**This is a project status, not a diary.** It replaces the older Done / Next / Blocked format for
EOD posts. Do not list what was accomplished today unless it changed a date, a rate, or a blocker.

## How to build it

1. **Establish the client and channel.** If the user did not say, ask. Do not guess which channel
   to draft into. Find the channel id with `slack_search_channels`.
2. **Identify the flows.** Read recent history in the client channel (`slack_read_channel`) and
   check earlier updates to see how flows were named before. Keep the same names so posts are
   comparable week to week.
3. **Gather evidence before writing.** In rough order of reliability: the client channel's last few
   days, for blockers already raised and what moved; Linear (`list_issues`, filtered to the client
   label), read-only, for R&D blockers, their state and ticket ids; the platform, when a flow is live, for real
   conversation counts and outcome rates; the user's own account of the day. Where sources
   disagree, prefer the primary one and say so in your reply.
4. **Ask for what is missing, once.** Exposure, deflection and next touchpoint are often not
   discoverable from tools. Collect the gaps into a single question rather than several.
5. **Write it.** Slack mrkdwn: `*bold*`, `•` bullets, no markdown headers or tables.
6. **Create a draft, never send.** `slack_send_message_draft` with the channel id. Only one attached
   draft per channel exists at a time. If the call returns `draft_already_exists`, tell the user to
   clear or send the existing draft rather than trying to overwrite it.
7. **Report the gaps.** In your reply, list every bullet where a number was unavailable or a date
   was assumed, so the user can fix them before sending. This matters more than the draft itself.

## Worked example

```
*Acme — EOD 25.08*

*Travel / Textual Agent*
- Go-live: 15-Sep formal, pushing for 02-Sep
- Exposure: not live — WhatsApp connected today, first traffic tomorrow
- Deflection / success: not live
- Blocked — R&D: prompt caching returns zero hits in Acme prod — INS-XXXX, <owner>, ETA unknown
- Blocked — FDE: none
- Blocked — client: SF classification set — Odelia via Neta, ETA 26.08
- Next touchpoint: Acme Strategy Discussion, Wed 26.08 11:00
```

Note what the example does: it reports two go-live dates rather than choosing, says "not live" twice
instead of inventing rates, writes "ETA unknown" where the ETA genuinely is, and names a person on
every blocker.

---

## Finish

One line confirming what was written where: hub files touched, memory entry, csm-hub health update
(status + note), and the Slack draft link. Then the gap list from step 7.

For tracked engagements, close with the single most important thing in the next five business days.
