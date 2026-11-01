---
description: Start-of-session briefing for a project. Reads the latest project memory, Linear status, and Slack channel so you're current before the day's work. Usage: /session-start <project>
argument-hint: <project> (e.g. acme, globex, initech)
---

You are starting today's work session on the project named in $ARGUMENTS. Produce a tight "where we stand" briefing by reading the three sources below. This is READ-ONLY: do not change memory, Linear, the agent, or anything else.

## 1. Resolve the project
Identify its three homes:
- **Memory:** run `find ~/.claude/projects -name "*.md" | grep -i <project>` and read the main `project_*.md` plus its `MEMORY.md` index. (Some projects have their own memory dir, e.g. a large client under its own folder.)
- **Linear project:** get the project ID from `reference_linear.md` in the main projects memory dir; if not there, use `list_projects` filtered by name (client projects are under the CX team).
- **Slack channel:** the channel ID is usually recorded in the project memory; otherwise `slack_search_channels` for the project name.

## 2. Read the latest from each (most recent first)
- **Memory:** the newest dated entries / current-state block, open items, "next" actions, and any watch-outs.
- **Linear:** the latest 1–2 status updates (`get_status_updates`) and the current health flag. Skim the description only if the status updates are thin.
- **Slack:** the last ~15 messages in the project channel — especially the most recent daily update and any manager/client questions or decisions raised since it.

## 3. Brief me
Synthesize into a short, scannable briefing:
- **Where we stand** — current state in 2–3 lines, including the Linear health flag.
- **Open / waiting on** — blockers and who owes what.
- **Needs a reply** — any Slack questions or decisions directed at us since the last update that are still unanswered.
- **Suggested focus for today** — 2–4 concrete next actions, drawn from memory's "next" list and open Slack threads.

Do not narrate the tool calls; just deliver the briefing, and end by asking what I want to tackle first.
