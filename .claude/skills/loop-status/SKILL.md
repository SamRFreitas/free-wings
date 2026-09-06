---
name: loop-status
description: Shows a short, structured readout of where the current work actually is — trigger, current agent, skills used so far, step count (exact for a fixed cascade, honest and qualitative for open-ended work), plain-language context, and next step. Use whenever the person asks "where are we" / "status" / "onde estamos" in any form, or proactively at natural checkpoints (the start and end of a multi-step cascade like Foundation Sync, or a spec-driven implementation with several pieces) — not on every single message.
---

# loop-status

> **Orientation, if this is the first skill file you're reading:** this
> is a **skill** for Claude Code — a repeatable, on-demand procedure,
> invoked directly (`/loop-status`) or triggered by a plain-language
> question like "where are we," not a subagent, and not the target
> project's own code. It's part of the **Free Wings** harness (see
> `FOUNDATION.md` in this repository for the whole picture, under "Loop
> Status" — this skill is the mechanical half of that convention; the
> convention itself, including when to show this unprompted, lives
> there, not here).

Produces one thing: a short block showing where the current work
actually stands, grounded directly in loop engineering's own structural
anatomy (trigger, topology, verifier, stop rule — see
`docs/reading-list.md` and `the-architect.md`, which this skill borrows
its shape from rather than inventing a separate one) so that whoever's
reading — the person, or a future session picking this back up — can
tell what's happening and where without having to reconstruct it from
scrollback.

## The block

```
Loop status
- Trigger: <what actually started this piece of work>
- Now: <who/what is acting right now — the main session, or a named
  agent if one was spawned>
- Used so far: <agents and skills actually invoked this loop, in order,
  or "none yet">
- Step: <exact "k of n" if this is a fixed, known cascade (e.g.
  Foundation Sync's five steps) — otherwise an honest qualitative sense
  ("early," "most of the way through," "wrapping up"), never a
  fabricated number for open-ended dialogic work>
- Context: one or two plain-language lines — what this specific part is
  actually about, no jargon assumed
- Next: what happens after this, or "waiting on you" if it's genuinely
  the person's turn to decide something
```

Every field maps onto one of loop engineering's four pieces: Trigger is
literally the trigger; Now + Used so far is the topology in progress;
Step + Context implicitly carries what's already been verified; Next is
the stop-rule-in-waiting — what still needs to happen before this loop
is actually done.

## When to show it

- **On demand, always** — the person asking "where are we," "status,"
  "onde estamos," or invoking `/loop-status` directly, in any phrasing,
  gets this block.
- **Proactively, at real checkpoints only** — the start of a multi-step
  cascade (a Foundation Sync run, a spec-driven implementation broken
  into several pieces), and again once it wraps up. Not on every
  message in between — a status block on every turn would be exactly
  the kind of context bloat this harness's own `observer` was just built
  to watch out for (see `docs/LEARNING_LOG.md`). The block itself stays
  a handful of lines specifically so that showing it is cheap even when
  it does show up.

## Honesty about the step count

Two genuinely different situations, and the block should never blur
them:

- A **fixed cascade** (Foundation Sync's five steps, a numbered spec
  with known pieces) has a real "step 3 of 5" — say the exact number.
- **Open-ended dialogic work** (a conversation still shaping what it's
  even building) doesn't have a real total to count against — forcing a
  fake "step 2 of 6" here would be manufactured precision, the same
  mistake this project already rejected once in a proposed "Validator"
  agent's invented formulas. Say "early," "getting close," or similar,
  honestly, instead.

## What this skill does not do

- Does not replace the person asking a real question when something is
  actually unclear — this is a status readout, not a substitute for
  checking understanding (see `researcher.md`'s "Validation checkpoint"
  for where a real comprehension check is required instead).
- Does not get shown on every single turn — see "When to show it" above.
