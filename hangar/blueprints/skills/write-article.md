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
2. Read the target project's `FOUNDATION.md`, plus whichever
   `LEARNING_LOG.md` entries, `deneir`'s `docs/observations/` files,
   `researcher`'s `docs/research/` findings, or ADRs the article is
   actually about.
3. Pick one real theme or piece of work as the article's spine — not a
   tour of everything that happened. Ground every claim in something
   that actually happened in the project; never invent an example.
4. Write the article to `docs/writing/<short-slug>.md` in the target
   project (create the folder if it doesn't exist yet). This is the
   same `docs/writing/` folder the `writer` agent uses for articles and
   essays — one folder for all longer-form published writing, not two.
5. Show the draft before treating it as final — this is meant for other
   people to read, so confirm it says what the person actually wants
   said before it's considered done.

## What this skill does not do

- Does not write diary entries — that's `write-diary`'s job.
- Does not produce the raw material it draws from — that's `deneir`,
  `researcher`, and the project's own ADRs and `LEARNING_LOG.md`.
- Does not decide what topic deserves an article — that's the person's
  call.

## Write permissions

This skill writes to `docs/writing/` in the target project — the same
destination the `writer` agent uses for articles and essays. It never
writes to a target project's own code, to `docs/decisions/`,
`docs/specs/`, `docs/observations/`, `docs/research/`, or to
`docs/LEARNING_LOG.md` (that last one belongs to the `write-diary` skill
and to the person themselves). If it cannot write (permissions, a
read-only environment, or any other restriction), it should say so
plainly rather than silently losing the draft — the article can be
output to the console as a preview so the person can persist it
manually.