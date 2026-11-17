# For the-architect — research still to do: tracking the cost of each task (2026-11-17)

Left open by the 2026-11-17 Foundation Sync. Nothing here is decided;
it is what the next session picks up, as a request to `researcher`.

## What exists now

The spec template has a **run record** at its end (agents that took
part, model and effort, tokens, time, size S/M/L), filled by the lead
agent when the work is done (ADR 0006). It says *what* to write down,
not *how to get a reliable number*.

## What is known (from `docs/research/`)

- A worker's report gives tokens, tools used and time: reliable, per
  worker. Baseline from those reports, all on the session's model (no
  `model` field yet); moved here from the Foundation Sync handoff note
  when that note was deleted:

  | Run | Tokens | Tools | Time |
  | :- | -: | -: | -: |
  | researcher, token efficiency (11-11) | ~100k | 16 | 3 min |
  | deneir, history draft (11-16) | ~28k | 4 | 2.4 min |
  | researcher, anatomy (11-16) | ~190k | 34 | 4.8 min |
  | researcher, vocabulary (11-16) | ~102k | 49 | 10.5 min |
  | researcher, spec anatomy (11-17) | ~70k | 45 | 37 min |
  | researcher, model choice (11-17) | ~127k | 24 | 12 min |
  | researcher, spec-kit (11-17) | ~68k | 39 | 20 min |
  | deneir, orchestrator sync (11-17, Sonnet, medium) | ~28k | 4 | 0.5 min |
- `/usage` attributes by agent only "approximately", from local
  history; its cache line covers the main conversation only
  (`2026-11-17-model-choice-per-agent.md`, section 5).
- Per-agent figures need OpenTelemetry with `OTEL_LOG_TOOL_DETAILS=1`,
  otherwise custom agents show as `"custom"`
  (`2026-11-16-anatomy-of-claude-code.md`, section 5).
- The lead agent's own cost: the session's "Usage by model".
- Task size (S/M/L) is the harness's own convention, an inference, with
  no cut-offs defined.

## To research

1. How to get tokens per agent per task **reliably** in Claude Code
   (hooks, OpenTelemetry, transcripts), and what each really measures.
2. Where the record **lives**: only in the spec's run record, or also
   in a file that can be summed across tasks. Not decided.
3. Cut-offs for S / M / L, from real specs.
4. Whether the measurement is cheap enough not to cost more than it
   saves.

## Why it matters

The model defaults per agent (`claudecode.md`, "Model and effort per
agent") are starting points. Changing one means measuring before and
after, one agent at a time. Without a reliable record, that stays a
guess. The Opus/medium defaults of `the-architect` and `programmer` are
under test and are the first to need it.
