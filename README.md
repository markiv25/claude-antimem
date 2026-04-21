# claude-anti-mem

> A rap sheet for Claude. The opposite of memory — a persistent log of hallucinations, overkill, confusions, and wrong assumptions, consulted *before* Claude commits to an answer so it can course-correct away from its own known failure modes.

Where [claude-mem](https://github.com/thedotmack/claude-mem) remembers what went **right**, `claude-anti-mem` remembers what went **wrong** — and uses that log as a pre-flight check.

## Why

Claude gets things wrong in patterns. It invents library methods by analogy. It wraps a one-liner in argparse + logging. It assumes Snowflake when you said "dbt". It adds retries you never asked for. Each mistake is usually obvious in hindsight — but Claude makes the same *shape* of mistake over and over across sessions because it has no record of them.

`claude-anti-mem` fixes that with two skills that work as a pair:

- **anti-mem-collector** — when you correct Claude ("that's overkill", "you hallucinated that", "I didn't ask for that"), it appends a structured entry to the rap sheet.
- **anti-mem-checker** — before Claude commits to a response, it reads the rap sheet, checks whether the current plan matches a past failure pattern, and revises if so.

The rap sheet lives at `~/.claude-anti-mem/anti-mem.md` as plain markdown. You can read it, edit it, commit it to a repo, share it with a team.

## Install

### Claude Code (plugin)

```bash
/plugin marketplace add <your-org>/claude-anti-mem
/plugin install claude-anti-mem
```

That's it. Both skills are registered, and the rap sheet directory is created on first use.

### Claude.ai (skills only)

Upload each skill from `skills/` individually via the skills UI:

- `skills/anti-mem-collector/`
- `skills/anti-mem-checker/`

The rap sheet in Claude.ai lives in the skill's own working area instead of `~/.claude-anti-mem/` — see [`docs/claude-ai.md`](docs/claude-ai.md) for details.

### Manual (any agent)

```bash
git clone https://github.com/<your-org>/claude-anti-mem ~/.claude-anti-mem/repo
ln -s ~/.claude-anti-mem/repo/skills/anti-mem-collector ~/.claude/skills/anti-mem-collector
ln -s ~/.claude-anti-mem/repo/skills/anti-mem-checker ~/.claude/skills/anti-mem-checker
```

## Quick start

1. Install (above).
2. Work normally with Claude. The checker runs silently before substantive responses.
3. When Claude gets it wrong, **tell it**. Natural phrases are enough:
   - "that's overkill"
   - "you made that function up"
   - "I never said Snowflake"
   - "you confused polars with pandas"
   - "stop adding logging I didn't ask for"
4. The collector logs the failure. Next time Claude hits a similar-shaped task, the checker catches it.

## What the rap sheet looks like

```markdown
# Anti-Memory: Claude Failure Log

## [0001] 2026-04-20 — Overengineered a one-liner dedupe request

- **Category**: overkill
- **Trigger**: User said "quick script" for a task solvable in one expression.
- **Failure**: Produced 60 lines with argparse and logging for what was a single dict-comprehension.
- **Correction**: Match scaffolding to stated scope. "Quick" + obvious one-liner → give the one-liner.
- **Check**: If the user's request uses "quick", "simple", "one-liner", or has an obvious ≤5-line solution, do not add argparse, logging, error handling, or type annotations unless explicitly asked.

---

## [0002] 2026-04-20 — Invented polars method `pivot_wider` by analogy to tidyr

- **Category**: hallucination
- **Trigger**: User asked if library A has a feature from library B.
- **Failure**: Claimed `polars.DataFrame.pivot_wider` exists. Actual method is `.pivot(...)`.
- **Correction**: When asked whether library A has a feature from library B, don't assume naming parity.
- **Check**: Before asserting that a specific method/function exists on a library, confirm it matches the library's actual naming conventions; if cross-library analogy is involved, flag uncertainty.

---
```

## How the two skills interact

```
┌──────────────────────────────────────────────────────────────┐
│  You give Claude a task                                      │
└───────────────────────────┬──────────────────────────────────┘
                            │
                            ▼
              ┌─────────────────────────────┐
              │   anti-mem-checker fires    │
              │  (reads ~/.claude-anti-mem/ │
              │       anti-mem.md)          │
              └─────────────┬───────────────┘
                            │
              ┌─────────────┴───────────────┐
              │                             │
              ▼                             ▼
     draft matches a        draft does NOT match
     logged failure         any logged failure
              │                             │
              ▼                             ▼
     Claude revises          Claude responds
     before responding            normally
              │                             │
              └─────────────┬───────────────┘
                            ▼
              ┌─────────────────────────────┐
              │    Claude's response        │
              └─────────────┬───────────────┘
                            │
                            ▼
               you say "that was wrong / overkill /
                     hallucinated / etc."
                            │
                            ▼
              ┌─────────────────────────────┐
              │  anti-mem-collector fires   │
              │  (appends new entry to      │
              │       anti-mem.md)          │
              └─────────────────────────────┘
```

## Repo layout

```
claude-anti-mem/                          ← repo root = marketplace
├── .claude-plugin/
│   └── marketplace.json                  ← marketplace catalog
├── plugins/
│   └── claude-anti-mem/                  ← the plugin itself
│       ├── .claude-plugin/
│       │   └── plugin.json               ← plugin manifest
│       ├── skills/
│       │   ├── anti-mem-collector/
│       │   │   ├── SKILL.md
│       │   │   └── README.md
│       │   └── anti-mem-checker/
│       │       ├── SKILL.md
│       │       └── README.md
│       └── README.md                     ← plugin install notes
├── README.md                             ← you are here
├── LICENSE
├── docs/
│   ├── design.md                         ← why two skills, not one
│   ├── rap-sheet-format.md               ← entry schema
│   ├── curation.md                       ← cleanup guide
│   └── claude-ai.md                      ← claude.ai install notes
└── examples/
    └── anti-mem.md                       ← sample rap sheet
```

This follows the official Anthropic layout: the repo root is the marketplace, each plugin lives under `plugins/<name>/`, and plugins auto-discover `skills/` inside themselves.

## Design choices

- **Two skills, not one.** Writing happens on user correction; reading happens before every substantive response. Different triggers, different moments in the loop — keeping them separate keeps each SKILL.md focused and reduces false-triggers.
- **Plain markdown rap sheet.** Human-readable, greppable, diffable, commitable. No database, no daemon, no sync protocol.
- **One shared file** at `~/.claude-anti-mem/anti-mem.md` by default. Override with `CLAUDE_ANTI_MEM_PATH` env var if you want per-project rap sheets.
- **Append-only.** The collector only appends. Editing, reorganizing, and retiring entries is a manual (or future-curator-skill) job. This keeps the write path simple and the file auditable.
- **The `Check:` field is the whole game.** Every entry ends with an imperative the checker can apply. Vague entries ("be more careful") are useless — the skill's examples show how to write checks that actually prevent repeats.

See [`docs/design.md`](docs/design.md) for the longer version.

## Contributing

PRs welcome. The most useful contributions are:

- New anti-patterns with well-written `Check:` fields (drop them in `examples/anti-mem.md`).
- Sharper triggering descriptions — if you see the checker under-fire or the collector over-fire, open an issue with the transcript.
- A curator skill (the third skill this project doesn't have yet): periodically dedupe entries, retire stale ones, tag by domain.

## License

MIT. See [LICENSE](LICENSE).
