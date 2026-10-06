# Free Wings

*Asas Livres.*

A harness — a reusable structure for working with an AI coding
assistant — built so that any project sitting under it explains itself
through a single source of truth, keeps a public trail of what it
decided and why, and treats mistakes and dead ends as worth writing down,
not hiding.

## The name, and who inspired it

**Wings** is [Alberto Santos Dumont](https://en.wikipedia.org/wiki/Alberto_Santos-Dumont)'s
flight — the Brazilian aviation pioneer who flew his early aircraft in
public rather than in secrecy, and deliberately left many of his
inventions unpatented, believing technical progress should benefit
everyone, not be gatekept by whoever holds the patent.

**Free** is [Paulo Freire](https://en.wikipedia.org/wiki/Paulo_Freire)'s
liberation — the Brazilian educator who distinguished "banking"
education (a teacher depositing knowledge into a passive student) from
*dialogic* education, where understanding is built together, through
real back-and-forth. Every project under this harness follows that
model when an AI assistant is involved: it explains a concept and its
trade-offs *before* asking for a decision, instead of deciding quietly
and reporting afterward.

Neither name needs to be spelled out for the spirit to come through —
"free" and "wings" carry it on their own, in both English and
Portuguese.

## Cloning this repo

`CLAUDE.md`, `AGENTS.md`, `.claude/`, and `.opencode/` aren't
committed — they're 100% regenerable from `FOUNDATION.md` and
`hangar/blueprints/` (which are), so they're gitignored rather than
tracked. After cloning, open the repository in your AI-Assisted Tool
(the coding environment — Claude Code, OpenCode, …) and invoke
`@CONSTRUCT` once to generate them locally, before your tool has any
repo-specific instructions to read.

## What this actually is

Free Wings itself has no runtime — it's a pattern, applied first to
[Shadow Glass](https://github.com/SamRFreitas/shadow-glass) (a Mac →
Windows low-latency remote-access project) and meant to generalize to
whatever comes after it. Every project under this harness:

- keeps one dense, tool-agnostic `FOUNDATION.md` as its single source of
  truth — never read automatically by any AI tool, but compiled into
  whatever each tool actually needs;
- generates `CLAUDE.md` + `.claude/` (Claude Code), `AGENTS.md` +
  `.opencode/` (OpenCode), and any future tool's files from that one
  source, via the `construct` bootstrapper (`CONSTRUCT.md`) — instead
  of hand-maintaining several files that drift apart from each other
  over time. How each tool's files are shaped lives in one **adapter**
  per tool, in `hangar/blueprints/adapters/` (`claudecode.md`,
  `opencode.md`); supporting a new tool means adding one adapter file,
  not editing `CONSTRUCT.md` or `FOUNDATION.md`. Adapters are
  independent of each other, and what `construct` generates for one
  tool never mentions the others;
- keeps three kinds of written record, each with one job: an **ADR**
  for *why* a broad direction was chosen, a **spec** for *what* a piece
  of work is going to do (written before implementing it), and a
  **learning log** for *what actually happened* (written after,
  deviations from the spec included);
- follows **Modular & Self-Sufficient Documentation**: every agent/skill
  file and generated document is written so someone with zero prior
  exposure can understand it from wherever they start reading — where
  they are, what it is, how it connects to the rest (see
  `FOUNDATION.md`'s own section by that name for the full convention,
  and its credited origin).

## Using this harness for a new project

1. Open Free Wings in your AI-Assisted Tool and invoke
   `@CONSTRUCT @HARI-SELDON` with the target project's path (e.g.
   `target: /path/to/project`). If the project already has a
   `FOUNDATION.md`, its tool-specific files are regenerated; if not,
   `construct` first walks you through a dialogue that produces one.
2. `construct` creates only the tool-specific directory and root
   configuration file — never `docs/`. The full contract is in
   [`CONSTRUCT.md`](CONSTRUCT.md).
3. The general agents this harness provides — one line each; each
   blueprint in [`hangar/blueprints/agents/`](hangar/blueprints/agents/)
   is the full description:
   - [**The Architect**](hangar/blueprints/agents/the-architect.md)
     (`the-architect`) — the entry point: decides which agent a task
     belongs to.
   - [`programmer`](hangar/blueprints/agents/programmer.md) — explains
     first, then implements in small pieces.
   - [`tester`](hangar/blueprints/agents/tester.md) — verifies what
     `programmer` just built.
   - [`researcher`](hangar/blueprints/agents/researcher.md) — grounds a
     claim in real, checked sources.
   - [`deneir`](hangar/blueprints/agents/deneir.md) — watches a
     project's evolution and writes observations.
   - [`writer`](hangar/blueprints/agents/writer.md) — shapes material
     into diary entries or articles.

   Its three skills (blueprints in
   [`hangar/blueprints/skills/`](hangar/blueprints/skills/)):
   [`write-diary`](hangar/blueprints/skills/write-diary.md),
   [`write-article`](hangar/blueprints/skills/write-article.md) and
   [`loop-status`](hangar/blueprints/skills/loop-status.md).
   `construct` is not a skill — it is the bootstrapper at the root.

   None of them is wired to any single project: each reads the target
   project's own `FOUNDATION.md` first and follows *that* project's
   conventions.

   Three rules bind every agent — "know the project", "recognize and
   refer" and "show first, then write" — written once in
   [`hangar/blueprints/shared/agent-rules.md`](hangar/blueprints/shared/agent-rules.md).
   How a change to the harness itself is kept consistent across its
   files is **Foundation Sync**, defined in
   [`FOUNDATION.md`](FOUNDATION.md).

## License

[MIT](LICENSE) — chosen deliberately over a copyleft license (like GPL):
Santos Dumont's refusal to patent his work was an unconditional gift, not
an obligation attached to how others used it afterward. MIT matches that
spirit more closely — use this however you want, no strings attached.

## Status

Founded 2026-09-05, named Free Wings on 2026-09-06. Six agents, one
shared rules file, three skills, and two adapters (`claudecode.md`, `opencode.md`) are
built and tested live (see "Using this harness for a new project"
above). Shadow Glass is the first project sitting under
this harness — the second is whichever project you point `construct` at
next. See [`docs/LEARNING_LOG.md`](docs/LEARNING_LOG.md) for the full
story, warts included.
