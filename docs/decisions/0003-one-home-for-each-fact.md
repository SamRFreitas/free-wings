# ADR 0003 — One home for each fact

<!--
Lives in `docs/decisions/` of Free Wings, the folder for ADRs about the
harness itself: one file per real architectural fork, recording why a
broad direction was chosen. This one concerns how the harness's own
source files (`FOUNDATION.md`, `CONSTRUCT.md`, the blueprints, the
adapters, `README.md`) relate to each other; read `FOUNDATION.md` first
for what those are.
-->

- **Date**: 2026-10-06
- **Status**: accepted

## Context

The same facts were written out in several source files. What
`the-architect` does was described in `FOUNDATION.md`, in its
blueprint, in `README.md` and in both adapters. Foundation Sync was
defined in `FOUNDATION.md` and again in the blueprint. `construct`'s
invocation contract appeared four times inside `CONSTRUCT.md`. A
twenty-line "show first, then write" block was pasted into six
blueprints.

On 2026-10-05 and 2026-10-06 this cost real work. A change to
`the-architect` was made in one copy and not the others; each repair
was itself a new hand-written copy, and one of them (the third rule
protecting Foundation Sync) was written differently in two files on the
same day. The person asked for a consistency check several times and
each answer surfaced another divergence.

Every copy is also loaded into a model's context, where it costs tokens
without adding information.

## Alternatives weighed

1. **Keep the copies and check them harder** (ADR 0002's `construct`
   check, more greps). Detects drift after it exists; does not remove
   its cause.
2. **One home for each fact; every other file points to it.** The
   programming rule against writing the same function twice, applied to
   the harness's own text.

## Decision

Alternative 2.

| Fact | Its one home |
| :--- | :--- |
| What the project is, philosophy, terminology, harness-wide conventions | `FOUNDATION.md` |
| Foundation Sync, whole | `FOUNDATION.md` — it binds every session |
| What one agent or skill does | its blueprint |
| Rules common to all agents | `hangar/blueprints/shared/agent-rules.md`, appended by `construct` to each compiled agent |
| Why agents are near or far from each other | `the-architect`'s blueprint |
| How `construct` works | `CONSTRUCT.md` |
| One AI-Assisted Tool's details | its adapter |
| Why a direction was chosen | an ADR |

`FOUNDATION.md` and `README.md` keep one line per agent and a link.

Foundation Sync's protections, three in ADR 0002 plus a sentence about
adapters, are now stated as four rules in the one place: order,
adapters follow, commit gate, `construct` checks.

`grilling` is named a practice. The harness has no blueprint for it and
lists three skills.

## Consequences

- Compiled outputs are still copies, by design: they are generated, not
  maintained by hand.
- An agent blueprint no longer reads as complete on its own: the common
  rules are in the shared file. Its orientation note says so.
- `the-architect`'s blueprint points to Free Wings' `FOUNDATION.md` for
  Foundation Sync. That is sound because the cascade runs only there.
- `construct` has one more thing to do (append the shared file). First
  exercised the same day by a run made from `CONSTRUCT.md` itself,
  against Shadow Glass: the six compiled agents carry the shared
  rules, and the consistency check named no disagreement. Not yet run
  that way on Free Wings itself.
- A new folder under `hangar/blueprints/` has to reach all three
  pillars. `FOUNDATION.md` and `CONSTRUCT.md` described `shared/`;
  `HARI-SELDON.md`'s orientation note still listed its siblings
  without it, and was corrected the same day.
- ADRs and diary entries are records of their day and are not rewritten
  to match.

## Sources consulted

Search summaries only, read on 2026-10-06; the articles themselves were
not read in full.

- Anthropic, "Effective context engineering for AI agents" — the
  smallest set of high-signal tokens; recall degrades as context grows.
  https://anthropic.com/engineering/effective-context-engineering-for-ai-agents
- Addy Osmani, "Agent Harness Engineering".
  https://addyosmani.com/blog/agent-harness-engineering/
- HumanLayer, "Skill Issue: Harness Engineering for Coding Agents".
  https://humanlayer.dev/blog/skill-issue-harness-engineering-for-coding-agents
- "The Emerging Harness Engineering Playbook" — the root file becomes
  mostly a table of contents as detail moves into separate files.
  https://www.ignorance.ai/p/the-emerging-harness-engineering
- "What Is Progressive Disclosure and Why It Matters for AI Agents".
  https://codemyspec.com/blog/progressive-disclosure
