# Write Spec

<!--
Lives in `hangar/blueprints/skills/`. Defines the `write-spec` skill — a repeatable, on-demand procedure, invoked by name, with no persistent context of its own between invocations. Read by `construct`, which compiles it into whatever skill format the detected tool expects. Fills the project's own `docs/specs/TEMPLATE.md` (copied from `hangar/blueprints/shared/spec-template.md`). Its natural user is the lead agent, with the person.
-->

## Role

Produces one spec for a target project, in the anatomy that project
already has in `docs/specs/TEMPLATE.md`, through a short dialogue with
the person. The spec says what and why, never how; "how" appears only
in its Design section, for decisions made with the person.

**Use this skill**: before any non-trivial piece of work is
implemented or delegated to a worker.

## Procedure

1. Read the project's `docs/specs/TEMPLATE.md`, and the numbering of
   its existing specs, so the new one fits.
2. Ask the person one question at a time, with the recommended option
   first. Do not ask what the repository can answer.
3. Keep at most 3 open questions. Anything else becomes a line in
   Assumptions, for the person to review.
4. Write requirements as what, never how, each with a "How to verify".
5. Mark the spec `closed` only when it has no open questions and every
   requirement has a "How to verify". History of what happened goes to
   the diary, not the spec.
6. Show the spec before writing it. Write to `docs/specs/` only with
   the person's explicit approval.

## What this skill does not do

- Does not implement — that is `programmer` or `engenir`.
- Does not decide for the person: an open decision is asked, an
  inferred default is written as an assumption.
- Does not fill the Run record — the lead agent does that when the
  work is done.

## Where this skill writes

Only `docs/specs/` in the target project, and only with the person's
explicit approval. If it cannot write there, it outputs the spec so
the person can save it.
