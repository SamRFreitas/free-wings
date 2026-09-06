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

`CLAUDE.md` and `AGENTS.md` aren't committed — they're 100% regenerable
from `FOUNDATION.md` (which is), so they're gitignored rather than
tracked. Run the `construct` skill once after cloning to generate them
locally before your AI tool has any repo-specific instructions to read.

## What this actually is

Free Wings itself has no runtime — it's a pattern, applied first to
[Shadow Glass](https://github.com/SamRFreitas/shadow-glass) (a Mac →
Windows low-latency remote-access project) and meant to generalize to
whatever comes after it. Every project under this harness:

- keeps one dense, tool-agnostic `FOUNDATION.md` as its single source of
  truth — never read automatically by any AI tool, but compiled into
  whatever each tool actually needs;
- generates `CLAUDE.md` (Claude Code), `AGENTS.md` (OpenCode and
  similar tools), and any future tool's file from that one source,
  via the `construct` skill — instead of hand-maintaining several files
  that drift apart from each other over time;
- keeps three kinds of written record, each with one job: an **ADR**
  for *why* a broad direction was chosen, a **spec** for *what* a piece
  of work is going to do (written before implementing it), and a
  **learning log** for *what actually happened* (written after,
  deviations from the spec included).

## Using this harness for a new project

1. Run the `construct` skill against the new project's own directory.
   If it finds no `FOUNDATION.md` there yet, it automatically creates
   `docs/decisions/`, `docs/specs/`, `docs/observations/`, and a
   `docs/LEARNING_LOG.md` skeleton (from [`hangar/`](hangar/), this
   harness's blueprint — nothing there is meant to be copied by hand),
   then starts the real dialogue that produces that project's first
   `FOUNDATION.md`. That dialogue can't be automated — it has to reflect
   what the project actually is, the same way this repository's own
   `FOUNDATION.md` was written — but every mechanical part of setting
   the project up is handled for you.
2. `construct` then generates `CLAUDE.md`/`AGENTS.md` from that
   `FOUNDATION.md`, same as it does for any existing project.
3. The general agents this harness provides —
   [`programmer`](.claude/agents/programmer.md) (explains and shows
   reasoning first, then implements while still teaching),
   [`observer`](.claude/agents/observer.md) (watches a project's
   evolution, read-only about it, writes only to its own
   `docs/observations/`),
   [`writer`](.claude/agents/writer.md) (shapes existing material into
   diary entries or articles),
   [`researcher`](.claude/agents/researcher.md) (checks a claim against
   real, current sources before it's trusted, instead of asserting it
   from memory), and
   [`architect`](.claude/agents/architect.md) (the entry point: decides
   which of the other agents a task belongs to, or says plainly when it
   fits none of them) — and its skills —
   [`construct`](.claude/skills/construct/SKILL.md),
   [`write-diary`](.claude/skills/write-diary/SKILL.md), and
   [`write-article`](.claude/skills/write-article/SKILL.md) — are
   symlinked into `~/.claude/agents/` and `~/.claude/skills/`
   respectively, so they're available from any project directory, not
   just this one — none of them are wired to Shadow Glass, or to any
   other single project, by name. They all work the same way: read the
   target project's own `FOUNDATION.md` first, follow *that* project's
   conventions, never guess at rules that aren't written down anywhere.
   If a request falls outside one agent's own scope, it says so and
   names which other agent fits better ("recognize and refer") instead
   of attempting the work anyway — `architect` is the agent whose whole
   job is making that call up front, and its own design follows loop
   engineering's structure (trigger, topology, verifier, stop rule —
   see [`docs/reading-list.md`](docs/reading-list.md) for the sources
   that grounding was actually checked against, and what's freely
   accessible versus what isn't). These referrals aren't equally likely
   in every direction, either — `observer`↔`writer` and
   `programmer`↔`researcher` are closer neighbor pairs than the rest
   (each pair's own files already describe handing off to the other), so
   an agent refers straight to its nearest neighbor when that alone
   solves the request, rather than looping every mismatch back through
   `architect` — see `architect.md`'s "Proximity between agents."

## License

[MIT](LICENSE) — chosen deliberately over a copyleft license (like GPL):
Santos Dumont's refusal to patent his work was an unconditional gift, not
an obligation attached to how others used it afterward. MIT matches that
spirit more closely — use this however you want, no strings attached.

## Status

Founded 2026-09-05, named Free Wings on 2026-09-06. Five agents and
three skills are built and tested live (see "Using this harness for a
new project" above). Shadow Glass is the first project sitting under
this harness — the second is whichever project you point `construct` at
next. See [`docs/LEARNING_LOG.md`](docs/LEARNING_LOG.md) for the full
story, warts included.
