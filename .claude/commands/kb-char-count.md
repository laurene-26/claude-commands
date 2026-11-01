---
description: Count characters in every article across one or more KB export directories. Outputs a per-article table sorted by size, plus totals and averages per KB. Usage: /kb-char-count <kb-dir> [kb-dir2] [...]
argument-hint: <kb-dir> [kb-dir2] [...]
---

Count characters in every `.md` file across the provided KB export directories and report results.

## Arguments

Parse `$ARGUMENTS` as a space-separated list of directory paths. Each path is one KB to analyze. There must be at least one path.

## Steps

1. For each directory, glob all `.md` files.
2. For each file, read it as UTF-8 and count total characters (including whitespace).
3. Sort files within each KB largest-to-smallest.
4. Print results using the format below.
5. After all KBs, print a combined cross-KB comparison table.

## Output format

For each KB:

```
======================================================================
  <KB label>  (<article count> articles)
  <path>
======================================================================
   CHARS  FILENAME
--------  ------------------------------------------------------------
 164,261  some-article.md
  14,546  another-article.md
     126  tiny.md

  Total articles : 901
  Total chars    : 9,313,248
  Average chars  : 10,336
  Largest        : 164,261  (filename)
  Smallest       :     126  (filename)
```

Truncate filenames longer than 75 characters with `...`.

After all KBs, print:

```
======================================================================
  COMPARISON SUMMARY
======================================================================
  KB                  Articles   Total chars   Avg chars
  ------------------  --------   -----------   ---------
  Main KB                  901     9,313,248      10,336
  David Test KB             13        87,797       6,753
```

## Implementation

Use a single Python one-liner piped through Bash — do not write a file to disk. Example pattern:

```bash
python3 - <<'EOF'
import os, sys
dirs = { ... }
# ... logic here
EOF
```

Build the script inline in the Bash tool call with the actual paths from `$ARGUMENTS`. Label each KB by its directory basename.

## Error handling

- If a path does not exist or is not a directory, print a clear error for that KB and skip it.
- If a file cannot be read, show `ERROR` in the chars column and exclude it from totals.
- If no `.md` files are found in a directory, say so explicitly.
