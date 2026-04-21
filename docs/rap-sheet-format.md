# Rap sheet format

The rap sheet at `~/.claude-anti-mem/anti-mem.md` is plain markdown with a strict per-entry template. The checker skill depends on the template — stay close to it when editing by hand.

## File header

```markdown
# Anti-Memory: Claude Failure Log

This file records patterns where Claude has gone wrong — hallucinations,
overengineering, confusions, unasked-for additions, wrong assumptions.
The anti-mem-checker skill reads this before Claude commits to assumptions.

---
```

Appears once at the top. The collector writes it on first use.

## Entry template

```markdown
## [NNNN] YYYY-MM-DD — <short title>

- **Category**: hallucination | overkill | confusion | unasked-addition | wrong-assumption
- **Trigger**: <the pattern in the user's message or context that led Claude astray>
- **Failure**: <what Claude did wrong>
- **Correction**: <what Claude should have done>
- **Check**: <an imperative the checker applies to future drafts>

---
```

### Fields

- **ID (`NNNN`)** — zero-padded 4 digits, monotonically increasing. Never reused, even if an entry is deleted.
- **Date (`YYYY-MM-DD`)** — ISO date the entry was logged.
- **Title** — short, specific, searchable. "Invented polars method `pivot_wider`" beats "hallucinated a method".
- **Category** — one of the five tags. If none fit, that's a signal the tag set needs extending (open an issue).
- **Trigger** — describes the *shape* of the situation, not the specific words. The checker matches on shape.
- **Failure** — the concrete thing Claude did wrong, in 1-3 sentences.
- **Correction** — the right move, stated positively.
- **Check** — the single most important field. See below.

## Writing good `Check:` fields

A `Check:` is an imperative the checker skill will apply to Claude's draft response. Test: can a future Claude read the draft and verify whether this imperative is violated?

### Bad

- "Be more careful about library APIs."
  → Not verifiable. What does "careful" mean applied to a draft?
- "Don't overengineer."
  → Not verifiable. Which specific behaviors?
- "Listen to the user."
  → Not applicable to a draft.

### Good

- "Before asserting that a specific method/function exists on a library, confirm it matches the library's actual naming conventions; if cross-library analogy is involved, flag uncertainty."
  → Checker can scan the draft for library method assertions and see whether uncertainty is flagged.
- "If the user's request uses 'quick', 'simple', or 'one-liner', do not add argparse, logging, error handling, or type annotations unless explicitly asked."
  → Checker can scan the draft for argparse/logging/etc. and cut them.
- "For dbt, Airflow, or data-pipeline setup tasks, do not assume a warehouse/engine. Either ask or inspect the repo for signals (docker-compose.yml, requirements, existing profiles)."
  → Checker can see whether the draft assumes a warehouse without having asked.

### Pattern: "Before X, do Y"

Most good `Check:` fields follow this shape:

- **Before asserting** a method exists → verify.
- **Before picking** a default → ask or inspect.
- **Before adding** scaffolding → confirm the user asked for it.

## Editing by hand

Safe to do:
- Delete stale entries.
- Fix typos or sharpen `Check:` wording.
- Merge duplicate entries (keep the older ID, note the merge in the title).

Avoid:
- Reusing IDs. If you delete entry 0003, don't renumber later entries — IDs stay stable so references in commits or notes still work.
- Changing the field names (`Category:`, `Trigger:`, `Check:`). The checker skill parses them.

## Example file

See [`../examples/anti-mem.md`](../examples/anti-mem.md) for a populated rap sheet.
