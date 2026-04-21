# Design

## Why two skills, not one

The writing path and the reading path have almost nothing in common:

| | anti-mem-collector | anti-mem-checker |
|---|---|---|
| **Fires on** | user correction | Claude's own planning moment |
| **Frequency** | rare (only when Claude fails) | constant (before every substantive response) |
| **Operation** | append to file | read file + match + revise |
| **Failure mode if over-triggered** | rap sheet fills with noise | every response gets slower |
| **Failure mode if under-triggered** | failures go unrecorded | failures repeat |

A single skill's description would have to cover both cases, which makes it harder for Claude to decide when to invoke the skill. Splitting them means each SKILL.md has one job, one set of triggers, and one set of anti-patterns — the skill-triggering mechanism works better when skills are narrow.

## Why a plain markdown file

Alternatives considered:

- **SQLite.** Rejected — can't be eyeballed, diffed, or committed as plain text.
- **JSON/YAML entries.** Rejected — structure gets in the way for a file humans also edit.
- **Per-entry files in a directory.** Rejected — more filesystem churn, harder to scan the whole rap sheet in one read.

Markdown with a strict entry template wins because:

- Claude can read the whole file in one pass (rap sheets stay small — hundreds of entries, not millions).
- Humans can edit, dedupe, and curate by hand.
- Git-friendly.
- No runtime dependencies, no daemon, no sync.

## Why one shared file

The default is `~/.claude-anti-mem/anti-mem.md` — one file for all projects. Rationale:

- Most failure patterns (hallucinated APIs, overkill scaffolding, assumption defaults) are project-agnostic.
- A global rap sheet compounds — the more Claude screws up across your work, the smarter it gets everywhere.
- For project-specific patterns, override `CLAUDE_ANTI_MEM_PATH` and commit the file to that repo.

## Why append-only

The collector never edits existing entries. Reasons:

- The write path stays trivially simple and auditable.
- Concurrent sessions (if Claude runs in multiple shells) can't clobber each other.
- Editing is a judgment call — which entry is a duplicate? which one is stale? — and judgment calls belong to the human (or a dedicated curator skill), not the collector.

## Why the `Check:` field is the whole game

An entry is only useful if the checker can actually apply it. A vague `Check:` like "be more careful" can't be verified against a draft response. A concrete one like "Do not add argparse, logging, error handling, or type annotations unless explicitly asked" *can* — the checker can look at the draft, see argparse, and cut it.

Writing good `Check:` fields is the hardest part of using this system. The collector's SKILL.md includes three worked examples; match their specificity when logging new entries.

## What's explicitly out of scope

- **Curation / dedup / retirement.** Manual for now. A `anti-mem-curator` skill is on the roadmap.
- **Multi-user sync.** Override `CLAUDE_ANTI_MEM_PATH` and put the file somewhere shared if you want team-wide rap sheets.
- **Categorization beyond the five tags.** `hallucination | overkill | confusion | unasked-addition | wrong-assumption` covers the observed failure modes. Add categories only if you see a pattern the existing tags can't express.
- **Automatic detection of Claude failures.** The collector fires on user correction, not on Claude self-assessing. Self-detection is a much harder problem and would dramatically increase false positives.
