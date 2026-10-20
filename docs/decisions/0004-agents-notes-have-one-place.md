# ADR 0004 — Agents' notes have one place

<!--
Lives in `docs/decisions/` of Free Wings, the folder for ADRs about the
harness itself. This one concerns where agents keep what they record
besides code, specs and research; read `FOUNDATION.md` first.
-->

- **Date**: 2026-10-20
- **Status**: accepted

## Context

In Shadow Glass, from 2026-10-06 to 2026-10-20, `programmer` had
nowhere to record anything but code and specs. It improvised ten
`note-*.md` files inside `docs/specs/`, of six kinds: learning
material, a message for `the-architect`, a proposed change to
`FOUNDATION.md`, a task, vocabulary, and personal records. The message
let a session resume the work two weeks later without losing anything.
Other notes outlived their need.

## Alternatives weighed

1. Learning material into `deneir`'s `docs/observations/`. One source
   for `writer`, but it mixes an outside observer with notes taken from
   inside the work.
2. No learning notes: `programmer` proposes the diary entry at the end
   of each session. One kind of file fewer; the detail is lost.
3. One folder per recipient. A note addressed to nobody has no place,
   and an agent's permission to write is no longer one folder.
4. One folder per author, `docs/notes/<agent>/`, the recipient in the
   file name.

## Decision

Alternative 4. `deneir`'s observations move into `docs/notes/deneir/`.
Specs and research stay where they are: they are permanent records,
notes are temporary. The project's vocabulary has its own file,
`docs/GLOSSARY.md`. The rule is in `hangar/blueprints/shared/agent-rules.md`;
`the-architect` reviews the notes, and the person decides what is
deleted.

## Consequences

- Every agent writes in one more place, its notes folder.
- Free Wings' own observation moved. Shadow Glass's notes are that
  project's own work and were not moved from here.
- What goes in a note for whoever continues is decided case by case,
  not by a fixed format.
