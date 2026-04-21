---
name: anti-mem-checker
description: Before Claude commits to an approach, assumption, or non-trivial response, consult the anti-memory file of past failures (hallucinations, overkill, confusions, wrong assumptions) and adjust course if the current plan matches a logged failure pattern. Trigger this skill aggressively — at the start of any substantive task, before picking a library or tool, before asserting a fact Claude is not certain of, before producing code longer than ~20 lines, before making an assumption about the user's stack/warehouse/framework, and any time the user gives a prompt that feels similar to one where Claude previously went wrong. This is Claude's pre-flight check against its own known failure modes.
---

# Anti-Mem Checker

This skill is Claude's pre-flight check. Before committing to a response, Claude reads the rap sheet and verifies the current plan doesn't match a logged failure pattern. If it does, Claude changes course.

The companion skill `anti-mem-collector` writes to the same file.

## Rap sheet location

Default: `~/.claude-anti-mem/anti-mem.md`.

If the environment variable `CLAUDE_ANTI_MEM_PATH` is set, read from that path instead.

If the file doesn't exist, there are no entries yet — skip silently and proceed normally.

## When to consult

Consult the rap sheet at the *planning* stage, before producing the user-facing answer. Specifically:

- **Start of any non-trivial task.** Coding tasks, design questions, data-pipeline work, architecture suggestions, anything multi-step.
- **Before asserting a specific API, method, function, library, or service exists.**
- **Before picking a default** (warehouse, framework, library, pattern) when the user didn't specify one.
- **Before producing code longer than ~20 lines**, or any response with structure (scaffolding, multiple files, configuration).
- **Before adding anything the user didn't explicitly request** — logging, retries, error handling, type hints, tests, CLIs, docstrings.
- **When the user's phrasing echoes a past trigger** — "quick", "simple", cross-library analogy questions, "set up X", etc.

Trivial exchanges (a greeting, a one-line factual question, acknowledging a correction) don't need a check.

## How to consult

1. Read the rap sheet file. If it doesn't exist, skip silently.
2. Scan entries. Each entry has a `Trigger:` and a `Check:`. Read the triggers first to filter for entries whose trigger pattern resembles the current situation.
3. For each matching entry, apply its `Check:` to the response Claude is about to produce.
4. If the draft response would violate any `Check:`, revise before sending. Do not send a response that matches a logged failure pattern.

## What "matching" means

A trigger matches when the *shape* of the current situation is the same, not when the exact words match. Examples:

- Past trigger: "User said 'quick script' for a task solvable in one expression."
  Current situation: User says "just a small helper to flatten this list" and the task is one line.
  **Match.** The shape (casual phrasing + trivially-small task) is the same.

- Past trigger: "User asked if library A has a feature from library B."
  Current situation: "Does duckdb have `.group_by` like pandas?"
  **Match.** Cross-library analogy question.

- Past trigger: "User asked for dbt setup without specifying a warehouse."
  Current situation: "Help me set up Airflow for this project" — user didn't specify executor/backend.
  **Match by analogy.** Same failure shape (tool setup, unspecified backend). Apply the check's spirit: don't assume the backend.

Err toward matching. A false-positive match costs Claude a moment of re-thinking; a false-negative costs a repeat failure.

## How to apply a Check

A `Check:` is an imperative. Read the draft response and ask: "Does this response violate this imperative?"

- If no: proceed.
- If yes: revise. Options:
  - Cut the offending content (the overkill scaffolding, the unverified method name, the assumed default).
  - Ask the user a brief clarifying question instead of assuming.
  - Flag uncertainty explicitly ("I'm not 100% sure `polars` has that exact method name — let me verify" rather than asserting).
  - Downscale scope to match the request.

Claude does not need to announce that it consulted the anti-mem. Silent course-correction is fine and usually preferred. Narrating "I checked my anti-memory and..." is unnecessary process-theater.

## Light vs heavy checking

Not every task needs a full scan. Calibrate:

- **Light check** (default): Glance at the rap sheet's titles and categories. If nothing obviously relates, proceed.
- **Heavy check**: When the task is high-stakes or closely resembles a known failure category — e.g., any dbt/Airflow/warehouse setup triggers a heavy read of every `wrong-assumption` entry involving those tools. Any cross-library "does X have Y" question triggers a heavy read of every `hallucination` entry involving library APIs.

For repeated work within a single session, Claude can internally cache which entries it already considered — no need to re-read the file every turn.

## Examples

**Example 1 — Overkill avoided**

User: "write me a quick function to parse this CSV row"

Claude consults anti-mem, finds entry [0003]:
> Check: If the user's request uses words like "quick", "simple", "one-liner", ... do not add argparse, logging, error handling, or type annotations unless explicitly asked.

"Quick" triggers the check. Draft response had a try/except + logging + type hints + docstring. Revise to the 3-line version.

Final response: the 3-line version, optionally followed by "I can add error handling or type hints if you want."

**Example 2 — Hallucination avoided**

User: "does duckdb have a `pivot_longer` like tidyr?"

Claude consults anti-mem, finds entry [0004] (the polars/tidyr one):
> Check: Before asserting that a specific method/function exists on a library, confirm it matches the library's actual naming conventions; if cross-library analogy is involved, say so explicitly and flag uncertainty.

Cross-library analogy question — match. Draft was about to say "yes, `df.pivot_longer(...)`". Revise: acknowledge uncertainty, describe duckdb's actual pivoting approach (`PIVOT` / `UNPIVOT` SQL), offer to verify against docs.

**Example 3 — Assumption avoided**

User: "help me set up dbt for this project"

Claude consults anti-mem, finds entry [0005]:
> Check: For dbt, Airflow, or data-pipeline setup tasks, do not assume a warehouse/engine. Either ask or inspect the repo for signals.

Match. Instead of writing a Snowflake `profiles.yml`, either (a) ask which warehouse, or (b) look for `docker-compose.yml`, `pyproject.toml`, or existing profiles first.

**Example 4 — No match, proceed**

User: "explain what a columnar storage format is"

Claude consults anti-mem. No entries relate to conceptual explanations of storage formats. Proceed normally.

## Anti-patterns for this skill itself

- Don't over-narrate the check. The user doesn't need a report on which entries Claude considered — they need a better response.
- Don't treat the rap sheet as exhaustive. The absence of a logged failure doesn't mean the current plan is safe; use normal judgment too.
- Don't skip the check on tasks that "feel easy" — many logged failures happened because a task felt easy. The point of this skill is to interrupt that feeling.
- Don't argue with an entry in-session. If a `Check:` seems wrong, still apply it for this turn, then ask the user whether the entry should be edited or retired (a curator task, not this skill's job).
