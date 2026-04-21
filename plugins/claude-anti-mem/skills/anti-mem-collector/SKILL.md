---
name: anti-mem-collector
description: Log Claude's own failures — hallucinations, overkill/overengineering, confusions, wrong assumptions, unasked-for additions — to a persistent anti-memory file so future Claude sessions can avoid repeating them. Trigger this skill whenever the user signals a mistake Claude just made: "you hallucinated that", "that's overkill", "I didn't ask for that", "you confused X with Y", "you made that up", "you're overthinking this", "wrong", "that's not what I meant", or any correction where Claude went off the rails. Also trigger when the user explicitly says "log this", "add to anti-mem", "remember not to do that", or similar. This is the opposite of a memory of successes — it's a rap sheet of failure modes to check against before acting.
---

# Anti-Mem Collector

This skill captures Claude's failure modes to a persistent file. The companion skill `anti-mem-checker` reads that file before Claude commits to assumptions, so these entries directly prevent future repeats.

## When to trigger

Log an entry when the user signals Claude just failed. Common signals:

- **Hallucination**: "you made that up", "that function doesn't exist", "that's not a real library", "you hallucinated"
- **Overkill / overengineering**: "that's overkill", "way too much", "I just wanted X", "you're overthinking", "simpler please" after an elaborate answer
- **Confusion**: "you confused X with Y", "that's a different thing", "wrong library/service/concept"
- **Unasked additions**: "I didn't ask for that", "why did you add X", "stop adding things I didn't request"
- **Wrong assumption**: "I never said that", "you assumed wrong", "that's not what I meant"
- **Explicit logging**: "log this", "add to anti-mem", "remember not to X"

Do NOT log:
- Minor style preferences (those belong in user preferences, not anti-mem)
- One-off typos or formatting gripes
- User changing their mind mid-task (not a Claude failure)

## The rap sheet location

Default: `~/.claude-anti-mem/anti-mem.md` — single markdown file, shared across projects.

If the environment variable `CLAUDE_ANTI_MEM_PATH` is set, use that path instead.

Create the file and the directory if they don't exist:

```bash
RAP_SHEET="${CLAUDE_ANTI_MEM_PATH:-$HOME/.claude-anti-mem/anti-mem.md}"
mkdir -p "$(dirname "$RAP_SHEET")"
touch "$RAP_SHEET"
```

If the file is empty, initialize it with this header (once):

```markdown
# Anti-Memory: Claude Failure Log

This file records patterns where Claude has gone wrong — hallucinations,
overengineering, confusions, unasked-for additions, wrong assumptions.
The anti-mem-checker skill reads this before Claude commits to assumptions.

---
```

## Entry format

Append a new entry to the bottom of the file. Use this exact structure so the checker skill can parse it reliably:

```markdown
## [NNNN] YYYY-MM-DD — <short title>

- **Category**: hallucination | overkill | confusion | unasked-addition | wrong-assumption
- **Trigger**: <what cues in the user's message or context led Claude astray — be concrete>
- **Failure**: <what Claude did wrong, in 1–3 sentences>
- **Correction**: <what Claude should have done instead>
- **Check**: <a short imperative the checker can apply, e.g. "Before suggesting a library, confirm it exists in the user's pyproject.toml">

---
```

Rules for good entries:

- **ID** is zero-padded 4 digits, incremented from the highest existing ID in the file. If the file is empty, start at `0001`.
- **Title** is short and specific — "Invented pandas method `.pivot_wider`" not "hallucinated a method".
- **Trigger** should describe the *pattern* that led to the failure so the checker can recognize it next time. "User asked for a quick one-liner" is better than "user asked a question".
- **Failure** is what actually happened. Be honest and specific.
- **Correction** is the right move. Concrete.
- **Check** is the rule the checker skill will actually apply. It should be an imperative a future Claude can verify against a draft response. "Do not add logging, retries, or error handling unless the user's task explicitly mentions reliability, observability, or production concerns" — not "be less overkill".

## Workflow

1. Recognize the trigger from the user's message.
2. Briefly acknowledge the correction to the user in normal prose — don't make a big show of logging it.
3. Read the rap sheet file to find the highest existing ID.
4. Append a new entry with ID = highest + 1 (or 0001 if empty), filling in each field based on what just happened in the conversation.
5. Confirm to the user in one short sentence: "Logged as [NNNN]: <title>."

## Examples

**Example 1 — Overkill**

User: "write me a quick script to dedupe a list of dicts by the 'id' key"
Claude: [produces 60 lines with argparse, logging, type hints, error handling, docstrings, main guard]
User: "dude that's way overkill, I just wanted `list({d['id']: d for d in xs}.values())`"

Entry:

```markdown
## [0003] 2026-04-20 — Overengineered a one-liner dedupe request

- **Category**: overkill
- **Trigger**: User said "quick script" for a task solvable in one expression. Claude reached for argparse/logging/error-handling scaffolding anyway.
- **Failure**: Produced 60 lines with CLI parsing and logging for what was a single dict-comprehension.
- **Correction**: Match scaffolding to the stated scope. "Quick" + a task with an obvious one-liner → give the one-liner, optionally mention the fuller version as a follow-up.
- **Check**: If the user's request uses words like "quick", "simple", "one-liner", or the task has an obvious ≤5-line solution, do not add argparse, logging, error handling, or type annotations unless explicitly asked.

---
```

**Example 2 — Hallucination**

User: "does polars have a .pivot_wider like tidyr?"
Claude: "Yes, you can use `df.pivot_wider(...)`"
User: "there's no such method, you made that up"

Entry:

```markdown
## [0004] 2026-04-20 — Invented polars method `pivot_wider` by analogy to tidyr

- **Category**: hallucination
- **Trigger**: User asked if library A has a feature from library B. Claude assumed naming parity and invented the method.
- **Failure**: Claimed `polars.DataFrame.pivot_wider` exists. The actual method is `.pivot(...)`.
- **Correction**: When asked whether library A has feature from library B, do not assume naming parity. Verify from the library's actual API (docs or known methods) before asserting a method name.
- **Check**: Before asserting that a specific method/function exists on a library, confirm it matches the library's actual naming conventions; if cross-library analogy is involved, say so explicitly and flag uncertainty.

---
```

**Example 3 — Wrong assumption**

User: "help me set up dbt for this project"
Claude: [assumes Snowflake, writes Snowflake-specific profiles.yml]
User: "we use postgres, I never said Snowflake"

Entry:

```markdown
## [0005] 2026-04-20 — Assumed Snowflake as the dbt warehouse without asking

- **Category**: wrong-assumption
- **Trigger**: User asked for dbt setup without specifying a warehouse. Claude defaulted to Snowflake.
- **Failure**: Generated Snowflake profiles.yml when the project uses Postgres.
- **Correction**: When warehouse is unspecified for dbt/ETL work, either ask or check the project (pyproject, docker-compose, env vars) before picking a default.
- **Check**: For dbt, Airflow, or data-pipeline setup tasks, do not assume a warehouse/engine. Either ask or inspect the repo for signals (docker-compose.yml, requirements, existing profiles).

---
```

## Anti-patterns for this skill itself

- Don't log every minor correction. If the user just fixed a typo or tweaked wording, that's not an anti-mem entry.
- Don't write vague checks like "be more careful". Every `Check:` must be a concrete, verifiable imperative.
- Don't rewrite or reorganize existing entries while adding a new one. Append only. A separate curator pass can reorganize later.
- Don't fabricate a trigger or failure you can't source from the actual conversation. If unsure what went wrong, ask the user briefly before logging.
