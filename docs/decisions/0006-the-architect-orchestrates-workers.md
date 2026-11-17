# ADR 0006 — The Architect orchestrates workers, and every certainty points to its research

<!--
Lives in `docs/decisions/` of Free Wings, the folder for ADRs about the
harness itself: one file per real architectural fork, recording why a
broad direction was chosen. This one revises ADR 0002 on whom
`the-architect` may delegate to, and adds specs with a template,
models per agent and traceability. It concerns
`hangar/blueprints/agents/the-architect.md`, `shared/spec-template.md`,
`skills/write-spec.md` and the adapters; read `FOUNDATION.md` first.
-->

- **Date**: 2026-11-17
- **Status**: accepted
- **Revises**: ADR 0002, on whom `the-architect` delegates to

## Context

On Chacal, usage reached 90% of the 5-hour limit and 65% of the weekly
one on a Pro plan; `/usage` attributed 66% to `engenir` and 24% to
`tester`. The cause: `the-architect`, the main session, delegated
`engenir` and `tester` as workers although its blueprint said to
recommend them, not delegate. A rule in text only, and no `model` set,
so every worker inherited the session's model.

The person turned "cut tokens" into "understand first, then change". The
research that grounds this ADR is in `docs/research/`:
`2026-11-11-token-efficiency-claude-code.md`,
`2026-11-16-anatomy-of-claude-code.md`,
`2026-11-16-harness-vocabulary.md`, `2026-11-17-spec-anatomy.md`,
`2026-11-17-spec-kit-and-formats.md` and
`2026-11-17-model-choice-per-agent.md`.

## Alternatives weighed

For the role of `the-architect`:

1. **Router only** (the vocabulary research's table: router /
   coordinator). It recommends; the person opens every agent. Safe, but
   it is what the harness already did, and the person carries every
   message across sessions.
2. **One agent, the orchestrator.** It is the default lead agent,
   decides who does what and delegates self-contained work to workers
   in parallel (a closed spec for implementing and verifying).
3. **Two agents**: a router and a separate orchestrator. Cleaner names,
   but one more agent sent on every request, and two files to keep
   consistent.

## Decision

Alternative 2.

- **`the-architect` is the orchestrator and the default lead agent** of
  every project. It delegates only self-contained work, and
  implementation or verification only with a closed spec; a worker that
  hits a doubt stops and returns the question. It
  does not choose a worker's model when delegating, because the model
  Claude passes on the call overrides the agent file's (verified in the
  Claude Code docs, 2026-11-17).
- **`programmer` is not delegable**: it teaches, so it needs the person.
- **Where the tool allows it, the limit on whom it may call is enforced
  by configuration**, optional per project; otherwise it is text only.
- **Vocabulary:** *lead agent*, *worker*, *orchestrator*. "Subagent" is
  Claude Code's word for a worker and stays in its adapter.
- **Specs** have a template (`shared/spec-template.md`), a skill to
  write one with the person (`/write-spec`) and a run record filled by
  the lead agent. `tester` reports each requirement against the spec.
- **Models per agent** are set in the Claude Code adapter, with a
  default and when to go up. Blueprints describe the need, not the
  model.
- **Traceability:** every certainty a project adopts points to the
  research behind it.

## Consequences

- "Orchestrator" is a choice, **not confirmed by the research**: the
  vocabulary table said router / coordinator, because the agent mostly
  recommended. It fits by Anthropic's definition now that it delegates
  and synthesizes.
- Open, not tested: whether `effort` in an agent file applies when it
  runs as the lead agent; whether a `tools` allowlist on the lead agent
  keeps other session tools. The default models for `the-architect` and
  `programmer` (Opus, medium) are under test; teaching quality per model
  was not found in the sources.
- `omitClaudeMd` and `autoCompactWindow` are not used: the generated
  instruction file is what an agent works from, so omitting it would
  send every worker to read `FOUNDATION.md`.
- ADR 0005 says `tester` "keeps explaining". It no longer asks the
  person before each check, since the work now starts from a closed
  spec.
- The Osmani link in `docs/reading-list.md` pointed to a different
  post; corrected. His primary text does not use the four-part
  "trigger, topology, verifier, stop rule" naming.
- Tracking each task's cost (agents, models, tokens, time, size) is
  recorded in the spec's run record. A reliable way to measure and keep
  it is still to be researched.
