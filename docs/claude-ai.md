# Using claude-anti-mem on Claude.ai

Claude.ai doesn't have the `/plugin` system Claude Code uses, and there's no persistent `~/.claude-anti-mem/` directory across sessions. You can still use the skills, with two caveats:

## Install

Upload each skill separately through the Claude.ai skills UI:

- `skills/anti-mem-collector/`
- `skills/anti-mem-checker/`

Both skills should now appear in your skill list.

## Where the rap sheet lives

Claude.ai sessions don't share a persistent home directory. The rap sheet's default path (`~/.claude-anti-mem/anti-mem.md`) doesn't survive across sessions the way it does in Claude Code.

Two workable approaches:

### Option 1 — paste the rap sheet into each session (best for casual use)

Keep `anti-mem.md` in a note-taking app or a gist. At the start of a session, paste it in with:

> "Here's my anti-mem. Use anti-mem-checker against it for this session."

Claude holds the rap sheet in conversation context for the session. When the collector logs new entries, Claude shows you the updated entry text so you can paste it back into your note.

### Option 2 — project knowledge (best for teams and long-running work)

If you're using a Claude Project, upload `anti-mem.md` as a project file. Both skills will read/write to it through the project's file system. The rap sheet persists across all sessions in that project.

This is the closest thing to the Claude Code experience on Claude.ai.

## What differs from Claude Code

- **No automatic file creation.** On Claude.ai, if no rap sheet exists yet, the collector's first entry is shown to you in full so you can save it somewhere. On Claude Code, the file is created automatically at `~/.claude-anti-mem/anti-mem.md`.
- **No `CLAUDE_ANTI_MEM_PATH` env var.** Tell Claude the file location in the conversation instead: "Our rap sheet is in the project file `team-rap-sheet.md`."
- **Session isolation.** Without Option 2, the rap sheet resets per session. Treat the exported entries as the source of truth and keep them outside Claude.
