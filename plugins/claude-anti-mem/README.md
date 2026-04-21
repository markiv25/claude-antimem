# claude-anti-mem — plugin

Claude Code plugin packaging for `claude-anti-mem`. If you just want to use it, you only need the top-level [README](../README.md) — this file is for people installing the plugin or debugging it.

## Install

```bash
/plugin marketplace add <your-org>/claude-anti-mem
/plugin install claude-anti-mem
```

Verify:

```bash
/plugin list | grep claude-anti-mem
```

You should see both skills registered:

- `anti-mem-collector`
- `anti-mem-checker`

## Data directory

On first use, the plugin creates:

```
~/.claude-anti-mem/
└── anti-mem.md       ← the rap sheet
```

The directory is created lazily — nothing is written until the first failure is logged.

### Overriding the location

Set `CLAUDE_ANTI_MEM_PATH` to a full file path if you want the rap sheet somewhere else:

```bash
export CLAUDE_ANTI_MEM_PATH="$HOME/work/team-rap-sheet.md"
```

Useful if you want to:

- Commit the rap sheet to a project repo so your team shares it.
- Keep separate rap sheets for separate clients.
- Point at a shared network location.

## Uninstall

```bash
/plugin uninstall claude-anti-mem
```

This removes the skills. It does **not** delete `~/.claude-anti-mem/` — your rap sheet is preserved. Delete it manually if you want a clean slate:

```bash
rm -rf ~/.claude-anti-mem
```

## Troubleshooting

**The checker never seems to fire.**
The checker is designed to run silently. To verify it's active, ask Claude: "Before you answer, tell me what's in the anti-mem." If the skill is wired in, Claude will read `~/.claude-anti-mem/anti-mem.md` and report its contents (or say it's empty).

**The collector doesn't log when I correct Claude.**
Two common causes:
1. The correction was too mild — the skill filters out minor style gripes. Be explicit: "log this as overkill" or "add to anti-mem: <failure>".
2. Write permissions. Verify `~/.claude-anti-mem/` is writable.

**The file got huge / messy.**
Prune it manually. The rap sheet is plain markdown — open it in your editor, delete stale entries, done. A curator skill to automate this is on the roadmap; see [`../docs/curation.md`](../docs/curation.md).
