# For the-architect — research still to do: tracking the cost of each task (2026-11-17)

Left open by the 2026-11-17 Foundation Sync. Questions 1, 2 and 4 were
researched on 2026-11-23
(`docs/research/2026-11-23-cost-tracking-per-agent.md`); nothing is
decided yet. The measurement itself is Free Wings work, for later.

## What exists now

The spec template has a **run record** at its end (agents that took
part, model and effort, tokens, time, size S/M/L), filled by the lead
agent when the work is done (ADR 0006). It says *what* to write down,
not *how to get a reliable number*.

## Corrected on 2026-11-23

This note said a worker's report gives tokens, tools used and time,
"reliable, per worker". That was wrong for tokens. The token figure in
a worker's report is the size of its **last request**, not what it
consumed in total: a `deneir` run reported ~28k while its transcript
sums ~96k (2026-11-23 research, section on the subagent report). Tool
count and time are whole-run figures and stay reliable.

So the table below compares how large each worker's context was at the
end, not what it cost. Its rows must not be summed as spend.

| Run | Last-request tokens | Tools | Time |
| :- | -: | -: | -: |
| researcher, token efficiency (11-11) | ~100k | 16 | 3 min |
| deneir, history draft (11-16) | ~28k | 4 | 2.4 min |
| researcher, anatomy (11-16) | ~190k | 34 | 4.8 min |
| researcher, vocabulary (11-16) | ~102k | 49 | 10.5 min |
| researcher, spec anatomy (11-17) | ~70k | 45 | 37 min |
| researcher, model choice (11-17) | ~127k | 24 | 12 min |
| researcher, spec-kit (11-17) | ~68k | 39 | 20 min |
| deneir, orchestrator sync (11-17, Sonnet, medium) | ~28k | 4 | 0.5 min |
| researcher, cost tracking (11-23, Sonnet, high) | ~180k | 50 | 54 min |
| researcher, agent quality (11-23, Sonnet, high) | ~122k | 93 | 2 h 39 min |

The agent-quality run's time includes a change of internet connection
midway, after which it retried the fetches that had failed; it is not a
clean figure.

## What the 2026-11-23 research proposes (not decided)

Sum token usage out of band from the session transcripts (input, cache
write, cache read, output and model per request), via an async
`SubagentStop` hook or a script run when a spec closes. It adds no
tokens to the session. Caveats: the transcript format is undocumented
and may change; deduplicate by message id; output tokens are a lower
bound until checked. OpenTelemetry is the documented alternative, with
a heavy setup. Details and every source are in the research.

The research left open why a subagent's `meta.json` says Opus while its
transcript lines say Sonnet. The person's explanation: `meta.json`
records the main session's model (Opus), and the agent then runs on its
own file's model. Not verified.

## Still open

1. Choose the mechanism and where the record lives: the person's
   decision, as a spec, when the measurement is taken up.
2. Whether the spec template's run record should say "tokens" at all,
   or name which figure (a harness change, so a Foundation Sync).
3. Cut-offs for S / M / L, from real specs.
4. After the mechanism: the experiment of each agent's model and
   effort, one agent at a time (method in
   `docs/research/2026-11-17-model-choice-per-agent.md`, section 5).

## Why it matters

The model defaults per agent (`claudecode.md`, "Model and effort per
agent") are starting points. Changing one means measuring before and
after, one agent at a time. Without a reliable record, that stays a
guess. The Opus/medium defaults of `the-architect` and `programmer` are
under test and are the first to need it.
