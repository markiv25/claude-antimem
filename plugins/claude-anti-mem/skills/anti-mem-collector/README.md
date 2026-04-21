# anti-mem-collector

Writes entries to the rap sheet at `~/.claude-anti-mem/anti-mem.md` when Claude fails and you say so.

## What it does

When you correct Claude — "that's overkill", "you made that up", "I never said Snowflake", etc. — this skill:

1. Recognizes the correction as a specific failure category.
2. Extracts the *pattern* that led to the failure (not just what happened).
3. Appends a structured entry to the rap sheet with a concrete `Check:` the checker skill can apply next time.

## What it doesn't do

- **Doesn't read the rap sheet or prevent failures.** That's [`anti-mem-checker`](../anti-mem-checker/).
- **Doesn't reorganize or dedupe.** Append-only by design. A future curator skill will handle cleanup.
- **Doesn't log style gripes.** "I prefer tabs" is a user preference, not a failure mode.

## Trigger phrases

Natural corrections work. Some examples that reliably fire the skill:

| You say | Category logged |
|---|---|
| "that's overkill" / "way too much" / "I just wanted X" | `overkill` |
| "you made that up" / "that function doesn't exist" | `hallucination` |
| "you confused X with Y" / "that's a different thing" | `confusion` |
| "I didn't ask for that" / "why did you add X" | `unasked-addition` |
| "I never said that" / "you assumed wrong" | `wrong-assumption` |
| "log this" / "add to anti-mem" | explicit |

## Entry format

See [`../../docs/rap-sheet-format.md`](../../docs/rap-sheet-format.md) for the full schema and examples.

## When you want finer control

Tell Claude explicitly what to log:

> "Log this as wrong-assumption: you defaulted to Snowflake again. Check: for any dbt task without a named warehouse, ask or inspect the repo first."

Claude will use your exact trigger/correction/check wording rather than inferring them.
