---
description: Build an interactive HTML conversation browser from a matched feedback MD file. Takes pilot/QA data (scores + rep feedback + Polina/QA comments + transcripts) and produces a filterable, expandable single-file HTML report. Usage: /feedback-browser <matched.md> [analysis.md] [output.html]
argument-hint: <matched.md> [analysis.md] [output.html]
---

You are building a standalone interactive HTML browser from pilot/QA feedback data.

## Arguments

Parse $ARGUMENTS as:
- Arg 1 (required): path to the matched markdown file (scores + feedback + transcripts per entry)
- Arg 2 (optional): path to an analysis/pattern file (failure patterns, fix priorities, etc.)
- Arg 3 (optional): output HTML path. Default: same directory as matched.md, filename = matched.md stem + `_browser.html`

## CRITICAL BUILD STRATEGY — READ THIS FIRST

**Never write the full HTML in one shot.** Large files (35+ entries with transcripts) exceed output limits and produce nothing. Always use this exact sequence:

### Step 1 — Read source files in pages
- Use Read with `limit` and `offset` parameters
- Read the matched file in chunks of ~1000 lines at a time
- Note the structure: each entry has a header (##), scores table, rep feedback, Polina/QA comments, and a full conversation
- Read the analysis file fully (it's usually shorter)
- Extract: entry count, date range, score dimensions, failure patterns if present

### Step 2 — Write the HTML shell
Use the Write tool to create the output file with:
- Full CSS (design system below)
- Filter bar (by date, verdict, pattern)
- Pattern legend (collapsible)
- JS filtering logic
- A placeholder comment `<!-- ENTRIES_PLACEHOLDER -->` where cards will go
- Footer

### Step 3 — Insert entries in batches via Edit
Replace `<!-- ENTRIES_PLACEHOLDER -->` with the first batch, then use Edit to append subsequent batches by replacing the last `<!-- END BATCH N -->` comment.

Each batch = ~10–12 entries. Each batch ends with `<!-- END BATCH N -->`.

**Do NOT spawn an agent to write the HTML.** Do this yourself, directly, using Write + Edit.

---

## HTML Design System

**Direction:** RTL (Hebrew, or match source language). Font: Segoe UI / Arial Hebrew.

**Color palette:**
- brand: #1a3a5c, brand-light: #e8f0fb
- green: #1e8a4a, green-bg: #e8f5ee
- yellow: #b8860b, yellow-bg: #fef9e7
- red: #c0392b, red-bg: #fdecea
- gray: #6b7280, border: #dde3ed
- body-bg: #f4f6fa, card-bg: #fff

**Score pill colors:**
- 100 → green chip `✓ 100`
- 50 → yellow chip `~ 50`
- 0 → red chip `✗ 0`
- "לא רלוונטי" / N/A → gray chip `—`

**Card structure (each entry):**
```
.card[data-date, data-verdict, data-patterns]
  .card-header (onclick toggle open/close)
    entry #, short ID (monospace), date badge
    score pills for the 2 main dimensions
    pattern badges
    toggle icon ▼/▲
  .card-body (hidden until open)
    .feedback-row (2-col grid)
      rep feedback block
      QA/Polina feedback block
    .all-scores-row (all 6 dimensions)
    transcript toggle button
    .transcript (hidden until toggled)
      per-message bubbles:
        .msg-bot → right-aligned, brand-light bg
        .msg-rep → left-aligned, #f0f0f0 bg
      source citation under bot messages
```

**Filter bar:** buttons grouped by date / verdict / pattern. Active = brand bg.
**Count label:** "X / N" updates live.
**Verdict data values:** fail, partial, correct, noise (can be space-separated for multi-label)
**Pattern data values:** space-separated letter codes (A B C etc.)

---

## Scoring the verdict field

Derive from the entry's main dimension scores:
- `fail` — any core dimension scores 0
- `partial` — core dims have 50s but no 0s
- `correct` — both main dims = 100
- `noise` — QA confirmed the bot was right but rep rated wrong (or vice versa)
- Multiple values allowed: `data-verdict="correct noise"`

---

## JS (include inline at bottom of file)

```js
var filters = {date:'all', verdict:'all', pattern:'all'};

function setFilter(type, val, btn) {
  filters[type] = val;
  btn.closest('.filter-group').querySelectorAll('.filter-btn')
    .forEach(b => b.classList.remove('active'));
  btn.classList.add('active');
  applyFilters();
}

function applyFilters() {
  var cards = document.querySelectorAll('.card');
  var shown = 0;
  cards.forEach(function(c) {
    var ok = true;
    if (filters.date !== 'all' && c.dataset.date !== filters.date) ok = false;
    if (filters.verdict !== 'all' && c.dataset.verdict.indexOf(filters.verdict) < 0) ok = false;
    if (filters.pattern !== 'all' && c.dataset.patterns.indexOf(filters.pattern) < 0) ok = false;
    c.classList.toggle('hidden', !ok);
    if (ok) shown++;
  });
  document.getElementById('countLabel').textContent = shown + ' / ' + cards.length;
  document.getElementById('noResults').classList.toggle('hidden', shown > 0);
}

function toggleCard(hdr) {
  hdr.closest('.card').classList.toggle('open');
}

function toggleTranscript(btn) {
  var t = btn.nextElementSibling;
  t.classList.toggle('open');
  btn.textContent = t.classList.contains('open') ? 'הסתר שיחה ▲' : 'הצג שיחה ▼';
}
```

---

## Conversation transcript format

For each turn in the transcript, determine speaker from the line prefix:
- `**[HH:MM:SS] ג'ף:**` → bot message (`.msg-bot`)
- `**[HH:MM:SS] נציג:**` → rep message (`.msg-rep`)
- The line `התשובה הופקה ממקורות המידע הבאים:` → source citation (`.msg-source`, italic, small, shown under the bot bubble)

Preserve Hebrew content exactly. Use `white-space: pre-wrap` on bubbles so line breaks and markdown-style formatting render cleanly.

---

## Pattern legend

If an analysis file was provided, extract the failure patterns (A, B, C... with descriptions) and render a collapsible legend at the top. If no analysis file, omit the legend or use a generic placeholder.

---

## Header stat pills

Extract from the matched file:
- Total entry count
- Unique pilot dates
- Average score on the main accuracy dimension
- Any other top-line stats visible in the file header

---

## After writing

1. Verify the file exists and has reasonable size: `ls -lh <output_path>`
2. Open it: `open <output_path>`
3. Report: output path, entry count, filter dimensions available

---

## What NOT to do

- Do not spawn an agent to write the HTML (it will exceed token limits)
- Do not try to write all 35+ entry cards in a single Write call
- Do not skip the shell-first step
- Do not omit the transcript — that's the core value of this tool
- Do not hardcode client-specific labels — read them from the source file
