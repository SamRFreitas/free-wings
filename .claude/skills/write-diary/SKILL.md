---
name: write-diary
description: Write or append a reflective diary entry to a target project's LEARNING_LOG.md, in the writer agent's voice. Use when there's enough raw material (a session's work, observer's docs/observations/) to turn into a proper diary entry.
---

# write-diary

> **Orientation, if this is the first skill file you're reading:** this
> is a **skill** for Claude Code — a repeatable, on-demand procedure
> invoked directly (`/write-diary`), not a subagent, and not the target
> project's own code. It's part of the **Free Wings** harness (see
> `FOUNDATION.md` in this repository for the whole picture). This skill
> reuses the `writer` agent's own file for its voice/themes rather than
> duplicating them — see `.claude/agents/writer.md`.

Produces one `LEARNING_LOG.md` entry for a target project — the same
file/convention that project already uses, not a new artifact type.
Reuses the `writer` agent's own context rather than duplicating it.

## Procedure

1. Read `/Users/samrfreitas/Lab/free-wings/.claude/agents/writer.md`
   (absolute path — this skill may run from inside any project, not
   just Free Wings) for the writer's general themes and labeled
   project-specific parallels.
2. Read the target project's `FOUNDATION.md` for its own conventions
   (language, tone, anything project-specific).
3. Gather the raw material: recent commits, the current conversation,
   and any files under that project's `docs/observations/` (produced by
   the `observer` agent) since the last `LEARNING_LOG.md` entry.
4. Write one new entry, in the same format the project's existing
   `LEARNING_LOG.md` entries already use — prepended or appended
   consistently with however that file is already ordered (check the
   existing file rather than assuming).
5. Show the entry before treating it as final and asking to commit it —
   this is personal reflection, not just a mechanical log.
