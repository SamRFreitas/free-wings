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
   `target: /path/to/project`; if you forget it, `construct` asks once).
   - **If the target already has a `FOUNDATION.md`** — this is a
     **recompile**: `construct` reads it and regenerates that project's
     tool-specific files from the harness's current blueprints.
   - **If it doesn't** — this is a **scaffold**: `construct` first walks
     you through a real dialogue, guided by
     [`hangar/blueprints/HARI-SELDON.md`](hangar/blueprints/HARI-SELDON.md),
     that produces the project's first `FOUNDATION.md`. That dialogue
     can't be automated — it has to reflect what the project actually
     is — then compilation runs as in a recompile.
2. Either way, `construct` creates **only** the tool-specific directory
   (`.claude/` or `.opencode/`) with the compiled agents and skills, plus
   the root configuration file (`CLAUDE.md` or `AGENTS.md`). It does
   **not** create `docs/` — `docs/decisions/`, `docs/specs/`,
   `docs/LEARNING_LOG.md` and the rest appear later, when the project's
   own work needs them. See [`CONSTRUCT.md`](CONSTRUCT.md) for the full
   invocation contract.
3. The general agents this harness provides (blueprints in
   [`hangar/blueprints/agents/`](hangar/blueprints/agents/)):
   - [`programmer`](hangar/blueprints/agents/programmer.md) — explains
     and shows reasoning first, then implements while still teaching, in
     small divided steps, checking the person's own understanding along
     the way; writes a spec in `docs/specs/` before non-trivial work.
   - [`tester`](hangar/blueprints/agents/tester.md) — tests what
     `programmer` just built, same explain-first and divide-and-conquer
     style (a provisional name, may become a combined "reviewer+tester"
     later).
   - [`deneir`](hangar/blueprints/agents/deneir.md) — watches a
     project's evolution, read-only about it, writes only to its own
     `docs/observations/`; captures which model is running each session
     from a reliable source only, plus a few research-grounded signals,
     never a scored formula.
   - [`writer`](hangar/blueprints/agents/writer.md) — shapes existing
     material into diary entries or articles.
   - [`researcher`](hangar/blueprints/agents/researcher.md) — checks a
     claim against real, current sources before it's trusted; when
     validating or comparing data/metrics, stops at a comprehension
     check until understanding is actually confirmed.
   - [**The Architect**](hangar/blueprints/agents/the-architect.md)
     (`the-architect`) — the entry point: decides which of the other
     agents a task belongs to, or says plainly when it fits none of
     them. It watches how the project works in order to decide, may
     read the repository's history (never change it), and writes only
     the records of its own decisions — ADRs, multi-step plans
     (`docs/specs/plan-*.md`), the project's `FOUNDATION.md` — after
     showing the text and getting explicit approval. Never code.
     Identifiers are kebab-case across the harness; any
     tool-specific naming rule lives in that tool's adapter.

   And its three skills (blueprints in
   [`hangar/blueprints/skills/`](hangar/blueprints/skills/)):
   [`write-diary`](hangar/blueprints/skills/write-diary.md),
   [`write-article`](hangar/blueprints/skills/write-article.md), and
   [`loop-status`](hangar/blueprints/skills/loop-status.md) (a short,
   honest readout of where a loop actually is — on request any time,
   or proactively at real checkpoints only). `construct` itself is
   **not** a skill — it is the bootstrapper at the root.

   None of them is wired to Shadow Glass or to any other single project
   by name. They all work the same way: read the target project's own
   `FOUNDATION.md` first, follow *that* project's conventions, never
   guess at rules that aren't written down anywhere.

   If a request falls outside one agent's scope, it says so and names
   which other agent fits better ("recognize and refer"). Referrals
   aren't equally likely in every direction: `deneir`↔`writer` is a
   closest pair, and `researcher`→`programmer`→`tester` is a
   three-agent chain, so an agent refers straight to its nearest
   neighbor when that alone solves the request, rather than looping
   every mismatch back through `the-architect` — see
   `the-architect.md`'s "Proximity between agents." `the-architect`'s
   own design follows loop engineering's structure (trigger, topology,
   verifier, stop rule — see [`docs/reading-list.md`](docs/reading-list.md)
   for the sources actually checked). It also owns **Foundation Sync**:
   any dialogue that changes how this harness itself works triggers a
   fixed cascade — `FOUNDATION.md` → `construct` → `README.md` → a
   `LEARNING_LOG.md` entry → `docs/learning-*.html`, plus a judgment
   call on whether a new ADR is warranted — checked and updated in
   full, and verified by actually grepping for stale references rather
   than trusted from memory.

## License

[MIT](LICENSE) — chosen deliberately over a copyleft license (like GPL):
Santos Dumont's refusal to patent his work was an unconditional gift, not
an obligation attached to how others used it afterward. MIT matches that
spirit more closely — use this however you want, no strings attached.

## Status

Founded 2026-09-05, named Free Wings on 2026-09-06. Six agents,
three skills, and two adapters (`claudecode.md`, `opencode.md`) are
built and tested live (see "Using this harness for a new project"
above). Shadow Glass is the first project sitting under
this harness — the second is whichever project you point `construct` at
next. See [`docs/LEARNING_LOG.md`](docs/LEARNING_LOG.md) for the full
story, warts included.
