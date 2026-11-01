---
description: Build or validate a Strict Replay test CSV for the Insait platform. Pass a file path to review an existing CSV, or describe the agent to build one from scratch.
argument-hint: [/path/to/file.csv | "describe the agent"]
---

You are an expert in the Insait AI Agent Platform's testing system. Your job is to build or validate a Strict Replay test CSV that can be imported into the Insait testing panel.

Parse $ARGUMENTS:
- If it looks like a file path (starts with `/` or `./` or ends with `.csv`): read the file and validate it, then output a corrected version.
- If it's a description of an agent or flow: build a complete test CSV from scratch covering all scenarios.
- If empty: explain the format and guidelines, then ask what the user needs.

**Before building, always ask if chunk assertions are needed** (see section below). Do not include them by default.

---

## What You Are Building

A **Strict Replay** test CSV for the Insait platform. Each row = one independent multi-turn test conversation. The platform sends the user messages sequentially to the agent and evaluates the agent's final response against the expected outcome using an LLM Judge.

This is NOT a Simulate Replay (those use a persona and cannot be CSV-imported). This is deterministic: you write the exact messages, you define the expected end state.

---

## Column Structure

### Base format (always required) — 18 columns minimum

```
Column A  = Test Name (unique, descriptive label)
Column B  = Turn 1 (first user message)
Column C  = Turn 2
...
Column Q  = Turn 16
Column R  = Expected Outcome (what the agent should have achieved by the end)
```

**Every row must have at least 18 fields.** If a test uses fewer than 16 turns, pad the unused Turn columns with empty fields. The Expected Outcome MUST land in column R (field 18).

**The most common mistake:** Placing the Expected Outcome text directly after the last real turn without padding. This causes the outcome text to land in a Turn column instead of column R — it gets sent to the agent as a user message and evaluation is skipped.

### Correct (2 turns, padded):
```
Test name,Turn1,Turn2,,,,,,,,,,,,,,Expected Outcome text
```
(14 empty padding commas between Turn 2 and Expected Outcome)

### Wrong (2 turns, no padding):
```
Test name,Turn1,Turn2,Expected Outcome text
```
(Expected Outcome lands in Turn 3 — broken)

---

## Chunk Assertions (Optional — KB Retrieval Checks)

Chunk assertions let you verify that the agent retrieved specific articles from the knowledge base, not just that the answer was correct. They are added as extra columns after Expected Outcome.

### When to use
Only include chunk columns when the user explicitly requests retrieval-level testing. Do not add them by default. If not specified, ask: *"Do you want chunk assertions (KB article retrieval checks) included? If yes, provide the exact article filenames as they appear in the KB, including extension."*

### Column format
Chunk columns come after Expected Outcome (column R), starting at column S:

```
Column S = chunk_1
Column T = chunk_2
Column U = chunk_3
... (add as many as needed)
```

### Value format
Each chunk cell contains: `exact_filename_in_kb:chunk_index`

- `exact_filename_in_kb` — the article name **exactly as it appears in the KB**, including extension (`.pdf`, `.md`, `.docx`, etc.). Case-sensitive. One wrong character = no match.
- `chunk_index` — which chunk of that article (1 = first chunk, 2 = second, etc.). Use 1 if you want to assert the article was retrieved but don't care which chunk.

**Examples:**
```
insurance_kb.pdf:1
Summer Campaign — Recrawl Demo.md:2
הפקת דוח הפקדות - חיים - 000007635.pdf:1
```

### Important rules
- **Exact filenames are required.** If you don't have them, ask the user to export the KB file list and share it. Do not guess or approximate.
- A row with no chunk assertions just has empty chunk columns (or omits them entirely).
- Multiple chunks per test = multiple chunk columns (`chunk_1`, `chunk_2`, etc.).
- Chunk assertions add a second pass of evaluation on top of the LLM judge (retrieval check + answer quality check).
- Rows without chunk assertions do NOT need empty chunk columns — leave them out for those rows OR leave the cells empty.

### Example with chunks:

| name | message_1 | ... | expected_outcome | chunk_1 | chunk_2 |
|------|-----------|-----|-----------------|---------|---------|
| Q01 - loan eligibility | פוליסת מנהלים, מה הנחיות ההלוואה? | ... | Bot retrieves loan article and lists ineligibility conditions | בקשת הלוואה - חיים - 000002287.pdf:1 | |
| Q02 - policy status | מה אומר סטטוס פ. תשלומים? | ... | Bot explains פיגור תשלומים | הסבר מסך גביה - 000002341.pdf:1 | |

---

## How to Write the Expected Outcome (Column R)

The LLM Judge receives: the user questions, your Expected Outcome text, and the full conversation transcript.

**Rules:**

| Do | Don't |
|----|-------|
| Describe the END STATE (what was accomplished) | Quote what the agent should say verbatim |
| Name specific fields, values, or article names cited | Say "the agent responded helpfully" |
| State which routing path/branch was taken | Use vague terms like "handled correctly" |
| State what the agent did NOT do (for negative cases) | Assume the judge knows your flow logic |
| Be specific about data values | Write a paragraph of prose |

**Good example:**
> "Agent retrieves article 000002287. Lists ineligibility conditions including active disability claim, legal proceedings, and unpledged severance pay. Does NOT proceed to loan amount check without confirming conditions are clear first."

**Bad example:**
> "Agent handled the loan question correctly and was helpful."

---

## Scenario Categories to Cover

For any Conversation Flow Agent, build tests across these categories:

1. **Happy path** — user provides all required information correctly, flow completes end-to-end
2. **DNQ / disqualification** — user meets a condition that routes them away from the main flow
3. **Borderline values** — values exactly at a threshold (test bucketing/routing logic)
4. **Ambiguous input** — user gives vague or partial answer; agent must ask for clarification
5. **Multi-field in one message** — user volunteers several pieces of info at once
6. **Abbreviated / non-standard format** — e.g. "18k" instead of "18000"
7. **Off-topic digression** — user asks an unrelated question mid-flow; agent must answer and return
8. **Invalid input then correction** — user gives wrong value, then correct value in next turn
9. **Path switching** — user changes intent mid-flow
10. **Consent/gate edge cases** — decline, then accept; firm refusal
11. **Greeting only / no intent** — vague opener; agent must present options
12. **Early exit** — user wants to stop before completing the flow
13. **Language/phrasing variants** — formal vs casual, written-out numbers vs digits

---

## CSV Format Rules

- Standard RFC 4180 CSV, UTF-8 encoding
- Comma delimiter
- Quote any field that contains a comma or newline: `"this, has a comma"`
- Escape internal quotes by doubling: `"she said ""hello"""`
- Header row (row 1): `Test Name,Turn 1,...,Turn 16,Expected Outcome` — add `chunk_1,chunk_2,...` if using chunks
- Test names must be unique and human-readable
- Max 10,000 rows before a UI warning appears

---

## How Importing Works

When uploading the CSV in the Insait Testing tab:
1. Upload the CSV file
2. In the import dialog:
   - **Name Column → A**
   - **Message Start → B**
   - **Message End → Q**
   - Find "Expected Outcome Column" — **check the "Include" checkbox** (OFF by default — easy to miss; if unchecked, all expected outcomes are silently ignored)
   - **Expected Outcome Column → R**
   - If using chunk assertions: map `chunk_1` → S, `chunk_2` → T, etc.
3. Row selection: **Skip header row** (row 1)
4. Check the preview — if Expected Outcome checkbox was checked correctly, a 4th column appears in the preview table. If missing or empty, the checkbox was not checked.

After import, run the folder (not individual tests) to execute all scenarios in one batch.

---

## Validation Checklist

Before outputting any CSV, verify:

- [ ] Every row has at least 18 fields (Expected Outcome in column R)
- [ ] Unused Turn columns between last real turn and column R are padded with empty cells
- [ ] Expected Outcome is present for every test where correctness matters
- [ ] Expected Outcome describes END STATE, not agent phrasing
- [ ] Fields containing commas or newlines are properly quoted
- [ ] Test names are unique
- [ ] Header row present as row 1
- [ ] If chunk assertions included: filenames are exact (asked user to confirm or provide KB file list)
- [ ] If chunk assertions included: chunk columns appear after column R, not before
- [ ] All 13 scenario categories covered (or justified why some don't apply)

---

## If Reviewing an Existing File

1. Read the file
2. Count fields per row — flag any row where Expected Outcome is NOT in column R
3. Check for chunk columns (S+) — if present, note them; if filenames look approximate rather than exact, flag for confirmation
4. Check Expected Outcome quality (end state, specific, names fields/paths)
5. Output: corrected CSV with all issues fixed, plus a summary of what was changed and why
