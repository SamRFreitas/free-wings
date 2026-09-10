# Write Diary

<!--
Lives in `hangar/blueprints/skills/`. Defines the `write-diary` skill — a repeatable, on-demand procedure, invoked by name, with no persistent context of its own between invocations. Read by `construct`, which compiles it into whatever skill format the detected tool expects. Reuses the `writer` agent blueprint's themes and voice rather than duplicating them — see `hangar/blueprints/agents/writer.md`.
-->

## Role

Produces one `LEARNING_LOG.md` entry for a target project — the same
file and convention that project already uses, not a new kind of file.
Different from `write-article`: a diary entry is a personal record of
one session; an article picks one theme or piece of work and develops
it fully for an audience. Reuses the `writer` agent's own context
rather than duplicating it.

**Use this skill**: when there's enough raw material (a session's work,
`deneir`'s `docs/observations/`) to turn into a proper diary entry.

## Procedure

1. Read `hangar/blueprints/agents/writer.md` for the writer's general
   themes and labeled project-specific parallels. This is a relative
   path inside the harness, not an absolute one — the skill may run
   from inside any target project, not just Free Wings, so the path is
   resolved against the harness, not the working directory.
2. Read the target project's `FOUNDATION.md` for its own conventions
   (language, tone, anything project-specific).
3. Gather the raw material: recent commits, the current conversation,
   any files under that project's `docs/observations/` (produced by the
   `deneir` agent) since the last `LEARNING_LOG.md` entry, and any
   relevant `docs/research/` findings (produced by the `researcher`
   agent) that belong in the reflection.
4. Write one new entry, in the same format the project's existing
   `LEARNING_LOG.md` entries already use — prepended or appended
   consistently with however that file is already ordered (check the
   existing file rather than assuming).
5. Show the entry before treating it as final and asking to commit it —
   this is personal reflection, not just a mechanical log. Only write to
   `LEARNING_LOG.md` with the person's explicit approval, since that
   file is theirs to author; without approval, output the entry as a
   preview so they can persist it themselves.

## What this skill does not do

- Does not write standalone articles or essays — that's `write-article`'s
  job.
- Does not produce the raw material it draws from — that's `deneir`,
  `researcher`, and the project's own commits and current conversation.
- Does not decide what's worth recording — it drafts an entry; the
  person decides whether and how to persist it.

## Write permissions

This skill writes to `docs/LEARNING_LOG.md` in the target project —
only with the person's explicit approval, since that file is theirs to
author. It never writes to a target project's own code, to
`docs/decisions/`, `docs/specs/`, `docs/observations/`,
`docs/research/`, or to `docs/writing/` (that last one belongs to the
`writer` agent and to the `write-article` skill). If it cannot write to
`LEARNING_LOG.md` (permissions, a read-only environment, or any other
restriction), it should say so plainly rather than silently losing the
entry — the draft can be output to the console as a preview so the
person can persist it manually.