# INSAIT Claude Commands & Skills

Shared Claude Code slash commands and skills for the INSAIT team.

> **Commands** are simple slash commands.
> **Skills** are richer — they carry full playbook context and handle multiple modes (build, review, audit).
> Both work the same way: drop a `.md` file into the right folder, restart Claude Code, done.

---

## Quick Install

Run this once to install everything:

```bash
# Commands
mkdir -p ~/.claude/commands
curl -o ~/.claude/commands/insait-test-csv.md \
  https://raw.githubusercontent.com/shai-insait/claude-commands/main/.claude/commands/insait-test-csv.md
curl -o ~/.claude/commands/insait-agent-json.md \
  https://raw.githubusercontent.com/shai-insait/claude-commands/main/.claude/commands/insait-agent-json.md
curl -o ~/.claude/commands/summarize.md \
  https://raw.githubusercontent.com/shai-insait/claude-commands/main/.claude/commands/summarize.md
curl -o ~/.claude/commands/feedback-browser.md \
  https://raw.githubusercontent.com/shai-insait/claude-commands/main/.claude/commands/feedback-browser.md
curl -o ~/.claude/commands/html-email.md \
  https://raw.githubusercontent.com/shai-insait/claude-commands/main/.claude/commands/html-email.md
curl -o ~/.claude/commands/session-start.md \
  https://raw.githubusercontent.com/shai-insait/claude-commands/main/.claude/commands/session-start.md
curl -o ~/.claude/commands/eod.md \
  https://raw.githubusercontent.com/shai-insait/claude-commands/main/.claude/commands/eod.md

curl -o ~/.claude/commands/kb-char-count.md \
  https://raw.githubusercontent.com/shai-insait/claude-commands/main/.claude/commands/kb-char-count.md
curl -o ~/.claude/commands/status-update.md \
  https://raw.githubusercontent.com/shai-insait/claude-commands/main/.claude/commands/status-update.md
curl -o ~/.claude/commands/status-update-auto.md \
  https://raw.githubusercontent.com/shai-insait/claude-commands/main/.claude/commands/status-update-auto.md

# Skills
mkdir -p ~/.claude/skills
curl -o ~/.claude/skills/discovery-html.md \
  https://raw.githubusercontent.com/shai-insait/claude-commands/main/.claude/skills/discovery-html.md
curl -o ~/.claude/skills/voice-agent.md \
  https://raw.githubusercontent.com/shai-insait/claude-commands/main/.claude/skills/voice-agent.md
```

Then restart Claude Code. All commands and skills will be available immediately.

---

## Available Commands

Commands go in `~/.claude/commands/`. Invoke with `/command-name`.

| Command | Invoke | What it does |
|---------|--------|--------------|
| INSAIT Agent JSON | `/insait-agent-json` | Build or validate a Conversation Flow Agent JSON for the INSAIT platform. Pass a file path to fix, or describe an agent to build from scratch. Covers full schema, all node types, exits, variables, and every known import failure mode. |
| INSAIT Test CSV | `/insait-test-csv` | Build or validate a Strict Replay test CSV for the INSAIT platform. Pass a file path to review, or describe the agent to generate test cases. |
| Summarize | `/summarize` | Summarize the last N assistant messages in a session. Usage: `/summarize 5` or `/summarize 3 voice agent rules`. |
| Feedback Browser | `/feedback-browser` | Build a filterable, expandable single-file HTML conversation browser from a pilot/QA feedback file (scores + rep feedback + QA comments + full transcripts). Usage: `/feedback-browser matched.md [analysis.md] [output.html]`. |
| HTML Email | `/html-email` | Compose and send a formatted HTML email via Gmail (colored sections, cards, tables, RTL Hebrew). Drafts by default; add `--send` to send. Usage: `/html-email <recipient> <subject> [--send]`. |
| KB Char Count | `/kb-char-count` | Count characters in every article across one or more KB export directories: per-article table sorted by size, plus per-KB totals and averages. Usage: `/kb-char-count <kb-dir> [kb-dir2] ...`. |
| Session Start | `/session-start` | Start-of-session briefing for a project: reads the latest project memory, Linear status update, and Slack channel, then summarizes where things stand plus a suggested focus. Usage: `/session-start acme`. |
| End of Day | `/eod` | End-of-day wrap for a project: updates project memory, logs a health update (green/yellow/red) to the csm-hub project, then drafts the daily update in the project's Slack channel (draft only). Linear is read-only. Usage: `/eod acme`. |
| Status Update | `/status-update` | Brings the csm-hub up to date for a project. Reads the hub first, gathers context since your last update (this session, Slack, email, meetings, Linear, platform numbers), files each fact as health, touchpoint, note, task, risk or integration, checks for stale or conflicting facts, then **shows a preview (every hub write and the Slack message) and waits for your OK**. On your go it writes the hub and posts the health note to the project's Slack channel. Adds only, never deletes. Usage: `/status-update acme`. |
| Status Update (auto) | `/status-update-auto` | Same as `/status-update` but writes the evidence-backed entries straight away and reports exactly what it wrote. Anything unclear or conflicting is listed as "needs your call" instead of written. It never sends to Slack: it leaves the health note as a **draft** in the project's channel for you to review and send. Usage: `/status-update-auto acme`. |

> `/status-update` and `/status-update-auto` were called `/hub-update` and `/hub-update-auto`. They now update the hub and Slack, so the name says what they do. If you installed the old files, delete `~/.claude/commands/hub-update.md` and `hub-update-auto.md` and install the new ones.

### Install individually

```bash
# INSAIT Agent JSON
curl -o ~/.claude/commands/insait-agent-json.md \
  https://raw.githubusercontent.com/shai-insait/claude-commands/main/.claude/commands/insait-agent-json.md

# INSAIT Test CSV
curl -o ~/.claude/commands/insait-test-csv.md \
  https://raw.githubusercontent.com/shai-insait/claude-commands/main/.claude/commands/insait-test-csv.md

# Summarize
curl -o ~/.claude/commands/summarize.md \
  https://raw.githubusercontent.com/shai-insait/claude-commands/main/.claude/commands/summarize.md

# Feedback Browser
curl -o ~/.claude/commands/feedback-browser.md \
  https://raw.githubusercontent.com/shai-insait/claude-commands/main/.claude/commands/feedback-browser.md
curl -o ~/.claude/commands/html-email.md \
  https://raw.githubusercontent.com/shai-insait/claude-commands/main/.claude/commands/html-email.md

# Session Start
curl -o ~/.claude/commands/session-start.md \
  https://raw.githubusercontent.com/shai-insait/claude-commands/main/.claude/commands/session-start.md

# End of Day
curl -o ~/.claude/commands/eod.md \
  https://raw.githubusercontent.com/shai-insait/claude-commands/main/.claude/commands/eod.md

# Hub Update (preview first)
curl -o ~/.claude/commands/status-update.md \
  https://raw.githubusercontent.com/shai-insait/claude-commands/main/.claude/commands/status-update.md

# Hub Update (automatic)
curl -o ~/.claude/commands/status-update-auto.md \
  https://raw.githubusercontent.com/shai-insait/claude-commands/main/.claude/commands/status-update-auto.md

# KB Char Count
curl -o ~/.claude/commands/kb-char-count.md \
  https://raw.githubusercontent.com/shai-insait/claude-commands/main/.claude/commands/kb-char-count.md
```

---

## Available Skills

Skills go in `~/.claude/skills/`. Invoke with `/skill-name`.

| Skill | Invoke | What it does |
|-------|--------|--------------|
| Discovery HTML | `/discovery-html` | Generate a self-contained interactive HTML discovery/briefing doc — checkboxes, progress bar, localStorage, PDF export. Pass a description or topic. |
| Voice Agent | `/voice-agent` | Build or review voice agent prompts and guardrails using the INSAIT Voice Agent Playbook. Pass a brief to generate all artifacts (system prompt, guardrails, node stubs, config checklist), or pass a file/paste to audit against the full playbook ruleset. |

### Install individually

```bash
# Discovery HTML
curl -o ~/.claude/skills/discovery-html.md \
  https://raw.githubusercontent.com/shai-insait/claude-commands/main/.claude/skills/discovery-html.md

# Voice Agent
curl -o ~/.claude/skills/voice-agent.md \
  https://raw.githubusercontent.com/shai-insait/claude-commands/main/.claude/skills/voice-agent.md
```

---

## Updating an existing command or skill

Re-run the same `curl` command — it overwrites the local file. Then restart Claude Code.

---

## Adding a new command or skill

1. Write your `.md` file (see any existing file for format reference)
2. Put it in:
   - `.claude/commands/` for a command
   - `.claude/skills/` for a skill (use this when the instruction set is large or has multiple modes)
3. Add a row to the right table in this README — include the invoke name, and a one-sentence description
4. Add a `curl` install block under "Install individually"
5. Open a PR — once merged, teammates can curl it down
