# anti-mem-checker

Reads the rap sheet at `~/.claude-anti-mem/anti-mem.md` before Claude commits to a response, and course-corrects if the draft matches a logged failure pattern.

## What it does

At the planning stage of any substantive response, this skill:

1. Reads the rap sheet.
2. Scans `Trigger:` fields for entries whose pattern resembles the current situation.
3. Applies each matching entry's `Check:` imperative against Claude's draft response.
4. Revises the draft before sending if any `Check:` would be violated.

The course-correction is silent by default — you get a better response, not a narrated self-audit.

## What it doesn't do

- **Doesn't write anything.** Read-only. Writes are handled by [`anti-mem-collector`](../anti-mem-collector/).
- **Doesn't replace judgment.** Absence of a logged failure doesn't mean the plan is safe.
- **Doesn't trigger on trivial exchanges.** Greetings, one-line factual answers, and simple acknowledgments skip the check.

## When it fires

Aggressively, by design. The checker should fire before:

- Any coding task producing more than ~20 lines.
- Any assertion that a specific API/method/library exists.
- Any default picked when the user didn't specify one (warehouse, framework, executor).
- Any scaffolding addition (logging, retries, error handling, tests, CLI) the user didn't request.
- Any prompt phrased similarly to a past failure trigger.

False positives are cheap. False negatives repeat mistakes.

## Light vs heavy checks

The checker calibrates effort to risk:

- **Light** — scan titles and categories, proceed if nothing relates.
- **Heavy** — read every entry in a relevant category when the task is high-stakes (e.g., all `hallucination` entries when the user asks about a library API).

## Verifying it's working

Ask Claude: *"Before you answer, tell me what's in the anti-mem."*

Claude will read the rap sheet file and report. If the reply is "no rap sheet yet" and you know entries exist, check `CLAUDE_ANTI_MEM_PATH` and file permissions.
