# ADR 0002 — The Architect works as a normal session, and Foundation Sync gets three guards

<!--
Lives in `docs/decisions/` of Free Wings, the folder for ADRs about the
harness itself: one file per real architectural fork, recording why a
broad direction was chosen. This one supersedes part of ADR 0001. It
concerns the agent blueprint `hangar/blueprints/agents/the-architect.md`,
the "Write permissions" section of every agent blueprint, and the
Foundation Sync cascade; read `FOUNDATION.md` first for what those are.
-->

- **Date**: 2026-10-06
- **Status**: accepted
- **Supersedes**: ADR 0001, on three points (see "Decision")

## Context

ADR 0001 let `the-architect` write the records of its own decisions and
kept three limits: never code, read-only version control, and
recommending the next agent without invoking it.

Further sessions on 2026-10-05 loosened all three in the blueprint, and
gave every agent the same "show first, then write" rule. Those changes
were made in the blueprints and the Claude Code adapter first.
`FOUNDATION.md`, `README.md`, ADR 0001 and the OpenCode adapter kept
describing the old agent. The disagreement was only noticed after
Shadow Glass was recompiled from the changed blueprints.

That is the drift Foundation Sync exists to prevent, happening to
Foundation Sync itself: the cascade was a rule in one agent's
blueprint, and a session that was not that agent edited a blueprint
without it.

## Alternatives weighed

For the role of `the-architect`:

1. **Keep ADR 0001's limits.** A pure role, but every one-line fix
   needs a second session opened as `programmer`, and every piece of
   research a hand-pasted message.
2. **Let it work as a normal session, bounded by approval.** It makes a
   small change itself, uses the shell and version control normally,
   and delegates to the agents that need no dialogue.

For keeping the cascade from being skipped:

1. **Leave it as a rule in `the-architect`'s blueprint.** That is what
   failed.
2. **Add guards outside that one blueprint**: an order rule for every
   agent, a commit gate, and a check inside `construct`.

## Decision

Alternative 2 in both.

**`the-architect`** (supersedes ADR 0001 on these three points):

- It may make a small change itself, code or configuration included.
  Work with real design decisions in it still goes to `programmer`.
- It uses version control and the shell as any session does. It commits
  or pushes only when the person asks.
- It may delegate to `researcher`, `deneir` and `writer`, where the
  AI-Assisted Tool lets one agent call another. `programmer` and
  `tester` it still recommends, because they need the person in the
  conversation.

Unchanged from ADR 0001: it records its own decisions, and never
rewrites the original body of an existing ADR.

**Every agent** follows "show first, then write": show the exact path
and text, wait for explicit approval, write only what was approved. A
delegated agent may write its own output in its own place, and nothing
else.

**Foundation Sync** gets three guards:

- **Order.** A change that fires the cascade starts in `FOUNDATION.md`.
  No blueprint, adapter or `CONSTRUCT.md` is edited for it first. This
  binds every agent and every session.
- **Commit gate.** Such a change is committed only once the cascade is
  closed.
- **`construct` checks.** It compares `FOUNDATION.md` with the
  blueprints before compiling and names any disagreement in its report.

An adapter's own format detail does not fire the cascade. A change in
`FOUNDATION.md` does require checking every adapter.

## Consequences

- `the-architect` is less pure as a role: "who decides" and "who
  executes" now overlap for small changes. What "small" means is a
  judgment the agent makes and the person can veto, since every change
  is shown first.
- All of this is still text an agent has to follow. Only the
  `construct` check gives a second chance to catch what the other two
  rules missed, and it reports rather than blocks.
- Not yet tested: delegation from `the-architect` inside OpenCode, and
  the `construct` check itself.
