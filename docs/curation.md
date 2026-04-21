# Curation

The collector is append-only. Over time the rap sheet accumulates duplicates, stale entries, and lessons Claude has actually internalized. Curation is the periodic cleanup that keeps the file useful.

There's no curator skill yet — this is a manual task. A future `anti-mem-curator` skill will automate it.

## When to curate

- **Monthly**, loosely. Whenever the file feels noisy.
- **When the checker starts feeling slow** or overly conservative — probably too many stale entries firing.
- **When you notice duplicates** — two entries logging the same failure shape with slightly different wording.

## What to look for

### Duplicates

Two entries describing the same failure shape.

Example:
```markdown
## [0012] — Overengineered a CSV parser with argparse
## [0019] — Overengineered a JSON reader with argparse and logging
```

Merge into the older entry with a broader `Check:`, or keep both if the `Trigger:` patterns are genuinely different.

### Stale entries

Entries where Claude no longer makes the mistake. Test: has the checker been catching it? If the same shape hasn't come up in months, the entry may have done its job.

Delete stale entries. Keep the IDs unreused.

### Vague `Check:` fields

Any entry whose `Check:` field isn't a verifiable imperative. Rewrite or delete.

### Over-broad checks

A `Check:` so broad it fires on almost every task ("Before writing code, reconsider whether the user asked for code"). These waste checker attention. Narrow them.

### Category drift

If you see a pattern the five tags can't express, propose a new category in an issue. Don't invent one-off tags in the file — the checker reads categories.

## How to curate

1. Open `~/.claude-anti-mem/anti-mem.md` in your editor.
2. Scan from top to bottom.
3. For each entry: keep / sharpen / merge / delete.
4. Preserve IDs — don't renumber.
5. Commit if the rap sheet lives in a repo.

## Team rap sheets

If your team shares a rap sheet via `CLAUDE_ANTI_MEM_PATH`, treat curation as a lightweight PR workflow: anyone can propose deletions or merges, but land them through review so nobody loses a check they were relying on.
