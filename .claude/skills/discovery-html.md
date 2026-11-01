# Skill: discovery-html

Generate a fully interactive, self-contained HTML discovery/briefing document.

## When to use
- User asks to create a discovery doc, briefing sheet, QA checklist, interview guide, or any structured question list
- User says "make this into an HTML", "create a checklist", "build a discovery page"
- Follow-up sessions where questions are derived from a process diagram, screenshot, or brief

## How to invoke
`/discovery-html <description>`

Examples:
- `/discovery-html onboarding bot for HR - Hebrew`
- `/discovery-html sales call prep checklist - English`
- `/discovery-html קובץ שאלות לפגישת אפיון בוט ביטוח`

---

## What to generate

Create a single `.html` file (no external dependencies) with ALL of the following features:

### Language & Direction
- Detect language from the user's description or content
- Hebrew / Arabic → `dir="rtl"`, `lang="he"` or `lang="ar"`
- English / other → `dir="ltr"`, `lang="en"`
- Mixed content is fine — set direction based on the dominant language

### Structure
- **Header**: dark background, title, subtitle, date (auto from JS)
- **Legend**: explain ★ = critical, ✓ = mark as asked/done
- **Progress bar**: live counter showing X of Y items completed
- **Action buttons row**: פתח הכל / Expand All | סגור הכל / Collapse All | הצג שלא נשאלו / Show Unchecked | אפס הכל / Reset | 🖨️ שמור כ-PDF / Save as PDF
- **Sections**: 5–10 thematic sections, each collapsible, with colored header + item counter badge
- **Each question/item**: checkbox + main text + optional sub-note in italic gray
- **★ Critical items**: add class `priority-high` — shows red star prefix
- **Notes textarea**: at the bottom, full width, auto-resizable
- **Save indicator**: "נשמר בשעה XX:XX" / "Saved at XX:XX" next to export button
- **Export button**: downloads `.txt` summary with notes + checked + unchecked items

### Persistence (localStorage)
- On every checkbox toggle → save all checkbox states to localStorage
- On every keystroke in notes → save notes text to localStorage
- On page load → restore checkbox states + notes from localStorage
- Reset button → clears localStorage too, asks for confirmation first
- Storage key: derived from the document title (slugified), so different docs don't collide

### Print / PDF
- `@media print` CSS: hide all buttons and controls, expand all sections, preserve colored headers (`-webkit-print-color-adjust: exact`), clean typography
- `window.print()` triggered by the PDF button

### Export to TXT
- Header with title and date
- Section: Notes (from textarea)
- Section: Items checked ✓
- Section: Items not yet asked ○
- UTF-8 BOM for Hebrew compatibility (`'﻿' + content`)
- Filename: `<slugified-title>_<date>.txt`

### Visual style
- Clean, modern, no external fonts or libraries
- Each section gets a distinct color (use a palette of 7–9 colors, cycling if more sections needed)
- Hover states on rows, green tint on checked rows, strikethrough on checked text
- Mobile-friendly (works on phone too)
- Suggested color palette for section headers:
  `#4f8ef7, #2ecc71, #e67e22, #e74c3c, #8e44ad, #16a085, #2c3e50, #c0392b, #1abc9c`

---

## Content generation rules

1. **Infer sections** from the use case — don't ask the user, just make smart thematic groupings
2. **Mark ★ critical** the questions that are blocking (can't build without this answer)
3. **Add sub-notes** where context helps (e.g. "נראה בתרשים כ-X" / "seen in diagram as X")
4. **Don't pad** — quality over quantity. 4 sharp questions beat 10 vague ones
5. **If user provides a process diagram or screenshot** — extract specific questions from it (channel names, node types, escalation rules, data sources, edge cases visible in the diagram)
6. **End each section** with at least one question about edge cases or failure modes
7. Aim for **30–50 total items** across all sections for a discovery doc

---

## Output

- Write the file to the current working directory as `<slug>.html`
- Open it in the browser immediately after writing (`open <file>`)
- Tell the user: file path, number of sections, number of questions, and that it auto-saves

## Quality check before finishing
- [ ] localStorage save + restore works (check the JS)
- [ ] Export TXT includes all three sections
- [ ] `@media print` hides all buttons
- [ ] All section count IDs match the JS loop range
- [ ] RTL/LTR direction is correct for the language
- [ ] No external CDN or font links (fully self-contained)
