---
description: Compose and send a beautifully formatted HTML email via Gmail. Renders with colored sections, cards, tables, and proper RTL Hebrew layout — exactly like a designed report. Creates a draft by default (safe). Usage: /html-email <recipient> <subject> [--send]
argument-hint: <recipient> <"subject"> [--send]
---

You are composing a rich HTML email using the Gmail MCP tool.

## Parse arguments from $ARGUMENTS

- **Arg 1** — recipient email address (required)
- **Arg 2** — subject line (required; may be quoted)
- **`--send` flag** — if present, send immediately. If absent, create a draft only.

If recipient or subject are missing, ask for them before proceeding.

## Understand the content

Look at the conversation immediately above this invocation. The content to email is whatever data, report, list, summary, or structured information was just discussed or produced. If it's ambiguous, ask: "What should I include in the email?"

## Build the HTML

Write a complete, self-contained HTML document for the `htmlBody` field. Rules Gmail enforces:
- **Inline styles only** — no `<style>` blocks, no `<link>` tags, no external CSS. Every style attribute must be on the element itself.
- **No JavaScript** — Gmail strips it entirely.
- **No external images** — use emoji or Unicode symbols instead of `<img>` tags pointing to URLs.
- **Tables for layout** — Flexbox/Grid may not render consistently across email clients. For complex multi-column layouts, use `<table>`.
- **Max width ~700px** — wider than that gets clipped on mobile.

### Design system (use these consistently)

```
Colors:
  brand:       #1a3a5c   (dark navy — headers, accents)
  brand-light: #e8f0fb   (pale blue — backgrounds, highlights)
  green:       #1e8a4a / bg #e8f5ee
  yellow:      #b8860b / bg #fef9e7
  red:         #c0392b / bg #fdecea
  gray:        #6b7280
  border:      #dde3ed
  body-bg:     #f4f6fa
  text:        #1a2332

Typography:
  font-family: Arial, sans-serif (safe across all email clients)
  base size: 14px, line-height: 1.65

Direction:
  Hebrew content → dir="rtl" on the <html> and relevant containers
  Mixed content → set dir per section
```

### Layout patterns (pick what fits the content)

**Page header** — dark gradient bar with title and subtitle:
```html
<div style="background: linear-gradient(135deg, #1a3a5c 0%, #2d5fa0 100%); color: white; padding: 18px 24px; border-radius: 10px; margin-bottom: 24px;">
  <div style="font-size: 11px; opacity: 0.7; margin-bottom: 6px; letter-spacing: 0.5px; text-transform: uppercase;">label / context</div>
  <div style="font-size: 20px; font-weight: 700;">Title</div>
  <div style="font-size: 12px; opacity: 0.8; margin-top: 6px;">Subtitle or metadata</div>
</div>
```

**Section label / group header** — colored pill above a group of items:
```html
<div style="background: #7c3aed; color: white; display: inline-block; padding: 4px 12px; border-radius: 4px; font-size: 12px; font-weight: 700; margin-bottom: 10px;">Section Name</div>
```
Swap color for: `#1a3a5c` (blue), `#c0392b` (red), `#b8860b` (yellow), `#1e8a4a` (green), `#6b7280` (gray)

**Entry card** — numbered item with colored left border:
```html
<table style="width: 100%; border-collapse: collapse; margin-bottom: 8px;">
  <tr style="background: #f4f6fa;">
    <td style="padding: 6px 10px; font-size: 11px; font-weight: 700; color: #6b7280; width: 50px;">#1</td>
    <td style="padding: 10px 12px; border-right: 3px solid #1a3a5c; background: white; font-size: 14px; line-height: 1.6; border-radius: 0 6px 6px 0;">
      Content here
      <div style="font-size: 11px; color: #6b7280; margin-top: 4px;">Sub-note or context</div>
    </td>
  </tr>
</table>
```

**Callout box** — highlighted note with icon:
```html
<div style="background: #e8f0fb; border-right: 4px solid #1a3a5c; padding: 14px 18px; border-radius: 6px; margin-bottom: 20px; font-size: 13px;">
  <strong>Note title:</strong> content here
</div>
```

**Score/status pill** (inline):
```html
<span style="background: #e8f5ee; color: #1e8a4a; border: 1px solid #a9dfbf; font-size: 11px; font-weight: 700; padding: 2px 8px; border-radius: 20px;">✓ Good</span>
```

**Two-column feedback row**:
```html
<table style="width: 100%; border-collapse: collapse; margin-bottom: 16px;">
  <tr>
    <td style="width: 50%; padding: 14px 18px; vertical-align: top; border-left: 1px solid #dde3ed;">
      <div style="font-size: 11px; font-weight: 700; color: #1a3a5c; text-transform: uppercase; margin-bottom: 6px;">Left column</div>
      Content
    </td>
    <td style="width: 50%; padding: 14px 18px; vertical-align: top;">
      <div style="font-size: 11px; font-weight: 700; color: #e8501a; text-transform: uppercase; margin-bottom: 6px;">Right column</div>
      Content
    </td>
  </tr>
</table>
```

**Footer**:
```html
<div style="border-top: 1px solid #dde3ed; padding-top: 16px; margin-top: 24px; font-size: 12px; color: #6b7280; text-align: center;">
  Prepared by Shai Shalom Hadad, INSAIT · date
</div>
```

## Wrap everything

Outer wrapper:
```html
<!DOCTYPE html>
<html dir="rtl" lang="he">   <!-- change dir/lang to match content -->
<head><meta charset="UTF-8"></head>
<body style="font-family: Arial, sans-serif; font-size: 14px; color: #1a2332; max-width: 700px; margin: 0 auto; padding: 20px; direction: rtl; background: #f4f6fa;">
  <!-- content here -->
</body>
</html>
```

## Send or draft

After building the HTML:

- If `--send` was passed → use `mcp__claude_ai_Gmail__send_email` (or equivalent send tool)
- If no `--send` → use `mcp__claude_ai_Gmail__create_draft`

Pass the full HTML to the `htmlBody` field. Pass a clean plain-text fallback (strip tags) to the `body` field for clients that don't render HTML.

## After sending/drafting

Report back:
- Draft created / Email sent ✓
- Recipient and subject
- One line describing what was formatted (e.g. "32 test questions grouped by failure pattern")

Do not show the full HTML in the conversation — just confirm it's done.
