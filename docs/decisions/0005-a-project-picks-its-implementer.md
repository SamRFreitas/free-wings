# ADR 0005 — A project picks its implementer

<!--
Lives in `docs/decisions/` of Free Wings, the folder for ADRs about the
harness itself. This one concerns which agent implements in a project,
and the first agent that does not teach; read `FOUNDATION.md` first.
-->

- **Date**: 2026-11-06
- **Status**: accepted

## Context

Until now every agent that implements followed Freire: `programmer`
explains first, asks to check understanding, and lets the person write
pieces. That fits Shadow Glass, a project for learning. The second
project under the harness (working name "seeker", an automated job
search and application tool) has another goal: the person chooses the
technologies and explains what he wants, and the work should get built
as fast as possible. The teaching would only slow it down.

## Alternatives weighed

1. A "direct mode" for `programmer`, switched on by the project's
   `FOUNDATION.md`. One agent fewer, but one agent with two opposite
   behaviors, and every rule in its file would need an "unless".
2. Drop the teaching from `programmer` altogether. Breaks Shadow Glass,
   which is built on it.
3. A new agent, `engenir`, in the same place in the chain
   (`researcher` → implementer → `tester`), with the same spec
   discipline and a different intent. Each project names which of the
   two implements there.

## Decision

Alternative 3. `engenir` builds while the person directs, explains only
what the person must decide, and shows a summary of each change — what
the file will contain and how it will behave — instead of the exact
text. The full text comes only when asked. `tester` keeps explaining,
whichever implementer is in use. Which one a project uses is asked when
it is scaffolded (`HARI-SELDON.md`) and recorded in its `FOUNDATION.md`.

## Consequences

- Seven agents. Wherever the harness said "`programmer`" for the
  implementer's place, it now names both or "the project's implementer".
- The shared "Show first, then write" rule has one exception, which
  points to `engenir`'s own file.
- `programmer` is unchanged for now; a later update to it is expected.
- Freire's principle remains Free Wings' own default; a project now
  opts out of it explicitly, by choosing `engenir`.
