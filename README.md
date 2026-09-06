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

1. In the target project's own repository, write a `FOUNDATION.md` by
   hand — through real conversation with an AI assistant, the same way
   this repository's own `FOUNDATION.md` was written. This can't be
   auto-generated from nothing; it has to reflect what the project
   actually is.
2. Run the `construct` skill against that project to generate its
   `CLAUDE.md`/`AGENTS.md` from the `FOUNDATION.md` just written.
3. Create `docs/decisions/`, `docs/specs/`, and `docs/LEARNING_LOG.md`
   in that project if they don't already exist.
4. The general agents this harness provides —
   [`programmer`](.claude/agents/programmer.md),
   [`observer`](.claude/agents/observer.md), and
   [`writer`](.claude/agents/writer.md) — and its skills —
   [`construct`](.claude/skills/construct.md),
   [`write-diary`](.claude/skills/write-diary.md), and
   [`write-article`](.claude/skills/write-article.md) — are
   symlinked into `~/.claude/agents/` and `~/.claude/skills/`
   respectively, so they're available from any project directory, not
   just this one. They all work the same way: read the target project's
   own `FOUNDATION.md` first, follow *that* project's conventions, never
   guess at rules that aren't written down anywhere.

## License

[MIT](LICENSE) — chosen deliberately over a copyleft license (like GPL):
Santos Dumont's refusal to patent his work was an unconditional gift, not
an obligation attached to how others used it afterward. MIT matches that
spirit more closely — use this however you want, no strings attached.

## Status

Founded 2026-09-05, named Free Wings on 2026-09-06. Shadow Glass is the
first project sitting under this harness. See
[`docs/LEARNING_LOG.md`](docs/LEARNING_LOG.md) for the full story,
warts included.
