# Write Article

<!--
Lives in `hangar/blueprints/skills/`. Defines the `write-article` skill — a repeatable, on-demand procedure, invoked by name, with no persistent context of its own between invocations. Read by `construct`, which compiles it into whatever skill format the detected tool expects. Reuses the `writer` agent blueprint's themes and voice rather than duplicating them — see `hangar/blueprints/agents/writer.md`.
-->

## Role

Produces a standalone article — for people outside the project to read,
not just the person who did the work. Different from `write-diary`: a
diary entry is a personal record of one session; an article picks one
theme or piece of work and develops it fully for an audience.

**Use this skill**: when a piece of work or a theme has enough depth to
deserve a standalone article, written from a target project's raw
material (`LEARNING_LOG.md` entries, `deneir`'s `docs/observations/`
files, `researcher`'s `docs/research/` findings, ADRs) — for external
readers, not just a personal record.

## Procedure

1. Read `hangar/blueprints/agents/writer.md` for the writer's general
   themes and labeled project-specific parallels — apply the
   Clean-Code-style formatting theme especially here (one concept per
   section, real example, why before how). This is a relative path
   inside the harness, not an absolute one — the skill may run from
   inside any target project, not just Free Wings, so the path is
   resolved against the harness, not the working directory.
2. Know the target project — from its generated instruction file when
   the session already has it, from its `FOUNDATION.md` otherwise —
   then read whichever `LEARNING_LOG.md` entries, `deneir`'s
   `docs/observations/` files, `researcher`'s `docs/research/`
   findings, or ADRs the article is actually about.
3. Pick one real theme or piece of work as the article's spine — not a
   tour of everything that happened. Ground every claim in something
   that actually happened in the project; never invent an example.
4. Show the draft first — this is meant for other people to read, so
   confirm it says what the person actually wants said.
5. Once approved, write it to `docs/writing/<short-slug>.md` in the
   target project (create the folder if it doesn't exist yet) — the
   same folder the `writer` agent uses.

## What this skill does not do

- Does not write diary entries — that's `write-diary`'s job.
- Does not produce the raw material it draws from — that's `deneir`,
  `researcher`, and the project's own ADRs and `LEARNING_LOG.md`.
- Does not decide what topic deserves an article — that's the person's
  call.

## Where this skill writes

Only `docs/writing/` in the target project, and only after the draft
was shown and approved. If it cannot write there, it outputs the
article so the person can save it.
