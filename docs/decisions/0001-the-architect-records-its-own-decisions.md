# ADR 0001 — The Architect records its own decisions

<!--
Lives in `docs/decisions/` of Free Wings, the folder for ADRs about the
harness itself: one file per real architectural fork, recording why a
broad direction was chosen. This is the first one. It concerns the
agent blueprint `hangar/blueprints/agents/the-architect.md`; read
`FOUNDATION.md` first for what the harness and its agents are.
-->

- **Date**: 2026-10-05
- **Status**: accepted

## Context

`the-architect` is the agent that decides which agent a task belongs
to. Until this decision its blueprint said it "doesn't implement, and
doesn't observe, and doesn't write", and it was compiled with read-only
tools.

A working session on 2026-10-05 in Shadow Glass, the first project
under this harness, showed what that cost in practice:

- Nobody could save a multi-step plan, a dated note on an existing ADR,
  or the roadmap in the project's `FOUNDATION.md`. `programmer` refuses
  `FOUNDATION.md`; `writer`, `researcher`, and `deneir` have no
  permission in those places. `the-architect` made those decisions and
  had to return them as text for someone else to save.
- It could not run `git status`, `git log`, or `git diff`. It was
  unable to explain two reverted commits or see what was uncommitted;
  the calling session had to paste that in.

## Alternatives weighed

1. **Keep it read-only; the person saves its output by hand.** Keeps
   the role pure. But the decision and its record are separated by a
   copy-paste step, and a decision that never gets pasted is lost.
2. **Give those writes to another agent** (`writer` for ADRs and plans,
   `programmer` for `FOUNDATION.md`). Keeps `the-architect` read-only,
   but adds a hop through an agent that starts with no context and did
   not make the decision, and it stretches those agents' own scope.
3. **Let `the-architect` write the records of its own decisions, with
   approval.** The one that decides is the one that records.

## Decision

Alternative 3.

- `the-architect` may create and edit ADRs (and dated notes on existing
  ones, never rewriting the original body), multi-step plans
  (`docs/specs/plan-<short-name>.md`), and the target project's
  `FOUNDATION.md`.
- Never code. Code stays with `programmer`.
- Never on its own initiative: show the exact text and path, wait for
  the person's explicit approval, write only what was approved.
- It may read the repository's history with read-only version-control
  commands, and never changes the working tree, index, history, or
  remotes.
- It still recommends the next agent rather than invoking it.

It also observes, which the old wording denied: it follows a project's
evolution as `deneir` does, with a different eye. `deneir` watches to
tell the story; `the-architect` watches the project and how it works,
in order to decide.

## Consequences

- The approval rule and the read-only git rule are **text in the
  blueprint**, not something a tool list enforces: a tool that grants
  "write" or "shell" grants it whole. The agent has to follow the rule
  itself, including where the tool is set to accept edits without
  asking.
- Plans share `docs/specs/` with `programmer`'s specs, told apart only
  by the `plan-` prefix. A separate `docs/plans/` folder was considered
  and not chosen.
- Foundation Sync is unchanged. Its section in the blueprint now states
  its scope: Free Wings, or a project whose `FOUNDATION.md` adopted it.
- Open: the OpenCode adapter still maps `the-architect` to read-only
  tools, so under OpenCode it cannot yet do what this ADR allows.

## Update — 2026-10-05

The open point above is closed in the source: the OpenCode adapter now
enables `write`, `edit` and `bash` for `the-architect`, with the same
two limits kept in the blueprint's text. Not yet tested inside
OpenCode, and that tool's generated files have not been recompiled
since 2026-09-22.
