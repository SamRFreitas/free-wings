# The Foundation

**Project: Free Wings** (*Asas Livres*)

> The source. Not read automatically by any AI-Assisted Tool — compiled
> into whatever each AI-Assisted Tool actually needs (AI-Assisted-Tool-
> specific instruction files such as `CLAUDE.md` or `AGENTS.md`, and
> whatever else shows up later) by the `construct` bootstrapper. This
> file is the one place the *reasoning* lives in full; the generated
> files are optimized excerpts of it, not the other way around.

Every project under this harness — this Free Wings repository included
— has exactly one `FOUNDATION.md`, AI-Assisted-Tool-agnostic by design.
Nothing in this file should ever assume a specific AI-Assisted Tool is
reading it; the moment it does, that content has drifted out of the
Foundation and belongs in a generated, AI-Assisted-Tool-specific file
instead.

## What this project is

**Free Wings** (*Asas Livres*) is the general, reusable layer of a pattern meant to
apply to *every* project, not just itself: each project keeps its own
`FOUNDATION.md` (dense, complete, AI-Assisted-Tool-agnostic) as its
single source of truth, and generates whatever AI-Assisted-Tool-specific
instruction files it actually needs from that one source — instead of
hand-maintaining multiple files that inevitably drift apart, which is
exactly the maintenance burden Shadow Glass's own pair of AI-Assisted-
Tool-specific files (`CLAUDE.md`/`AGENTS.md`, in that specific case)
already accumulated by being edited by hand, twice, every time something
changed. Those two file names are examples of the pattern, not
requirements of it: any AI-Assisted Tool that reads a configuration file
can be supported the same way.

This repository itself is not software with a runtime. It is the
**general layer** — philosophy, formats, naming conventions, reusable
blueprints for agents and skills — that any target project (Shadow
Glass first) can sit underneath. A general-purpose agent working inside
this harness (a "programmer," a "deneir") reads a target project's
own `FOUNDATION.md` directly, by absolute path, to understand that
project — no bespoke per-project adapter file needs to be hand-written
and kept in sync; every project having its own `FOUNDATION.md` already
*is* the adapter.

The analogy worth keeping in mind whenever this pattern feels
over-engineered: a compiler has one front-end (the part that understands
the source language, written once) and many pluggable back-ends (one per
target architecture). This `FOUNDATION.md`, in any project, plays the
front-end's role — general, written once, describing what's actually
true about that project. Each generated AI-Assisted-Tool-specific file
plays a back-end's role — the same truth, compiled for one specific
reader.

## Philosophy — why this exists, not just what it does

Two real people's ideas shaped the spirit of this project, and stay
worth naming explicitly rather than dissolving silently into a project
name and being forgotten:

**Paulo Freire**, a Brazilian educator, drew a distinction between
"banking" education — a teacher depositing knowledge into a passive
student, who simply receives it — and *dialogic* education, where
teacher and student build understanding together, through real
back-and-forth. Free Wings itself, and every fork or contribution to
this harness, follows the dialogic model deliberately when working with
an AI assistant: the assistant explains a concept and its trade-offs
*before* asking a person to decide anything about it, rather than
deciding quietly and reporting the decision afterward. This isn't a
style preference — it's the concrete, practical form Freire's idea takes
when the "classroom" is a person and an AI working through a real
technical decision together. See "Pedagogical approach" below for the
literal rule this becomes.

**Alberto Santos Dumont**, a Brazilian aviation pioneer, flew his early
aircraft in public — not in secrecy — and deliberately left many of his
inventions unpatented, believing technical progress should benefit
everyone rather than be gatekept by whoever owns the patent. Free Wings
itself, and every fork or contribution to this harness, defaults to the
same posture: a public repository, a visible written trail of technical
decisions (ADRs), and a diary recording what was actually learned —
including the dead ends and the wrong turns, not only the polished
result. Publishing the process, not just the outcome, is the point.

This project's name, **Free Wings** (*Asas Livres*), carries both
without needing to spell out either name every time it's invoked: "wings"
is Dumont's flight, unmistakably; "free" is Freire's liberation —
the same freedom "banking" education withholds and dialogic education
gives back. Neither half needed the other person's name attached to be
legible; together, they hold both.

## Structure

Free Wings is carried by three pillars — `FOUNDATION.md`,
`CONSTRUCT.md`, and `hangar/blueprints/HARI-SELDON.md` — described in
full below. None assumes a specific AI-Assisted Tool.

- `FOUNDATION.md` (this file) — the one source of truth, AI-Assisted-
  Tool-agnostic, as dense and complete as it needs to be. Never
  generated; always hand-written and hand-edited directly.
- `CONSTRUCT.md` — the bootstrapper: reads this file and
  `hangar/blueprints/`, then generates the AI-Assisted-Tool-specific
  files for the tool it detects. Not a skill, not a script. How it is
  invoked, and everything else about it, is defined in `CONSTRUCT.md`
  itself.
- AI-Assisted-Tool-specific configuration files — generated from this
  Foundation by the `construct` bootstrapper. `CLAUDE.md` (for Claude
  Code) and `AGENTS.md` (for OpenCode and similar AI-Assisted Tools)
  are the currently-known examples; the list is open-ended and grows as
  new AI-Assisted Tools appear. Each one is optimized for its specific
  reader (a weaker model reading `AGENTS.md`, for instance, gets more
  explicit, less-compressed instructions than a stronger one reading
  `CLAUDE.md` — the same underlying truth, different compression). The
  exact shape each AI-Assisted Tool expects is not defined here; it
  lives in `hangar/blueprints/adapters/`.
  **Gitignored, not committed** — same reasoning as never committing a
  `build/` folder: a file that's 100% regenerable from a tracked source
  doesn't belong in version control, and committing it would silently
  assume every future contributor needs every supported AI-Assisted
  Tool's file, rather than generating only the one they actually use.
  This isn't specific to this repository — any project under this
  harness can make the same call, though an existing project (Shadow
  Glass, at the time of this decision) may reasonably keep them
  committed if that's already the established practice there.
- `docs/decisions/` — ADRs about the harness itself (not about any one
  target project — those live in that project's own `docs/decisions/`).
- `docs/LEARNING_LOG.md` — the diary: one entry per session, at the
  meta level of harness engineering and the learning process, not one
  project's narrow technical details.
- `hangar/` — the workshop. Named for where an aircraft is prepared
  before it flies; nothing inside `hangar/` executes directly. Every
  file here exists to be read by `construct`, which compiles the
  blueprints into the AI-Assisted-Tool-specific outputs that *do* fly.
  The philosophy is uniform: the AI-Assisted Tool's own harness follows
  the blueprint ideas, never the other way around.

  - `hangar/blueprints/` — the schemas and rules that `construct`
    reads. It contains blueprints (`agents/`, `shared/`, `skills/`, and
    `HARI-SELDON.md`) plus `adapters/` (read as compilation rules).
    Most blueprints are compiled into AI-Assisted-Tool-specific outputs;
    `HARI-SELDON.md` is read during scaffolding instead. Each entry
    below describes its own role in full.

    - `hangar/blueprints/HARI-SELDON.md` — the project foundation
      blueprint. Passed as an optional parameter to `construct`
      (`@CONSTRUCT @HARI-SELDON`) when the target is another project.
      `construct` reads it during scaffolding. A section skeleton with
      guidance — prompts for project-specific content and
      optional-inheritance defaults, not filled-in project content.
      Named after Hari Seldon, the fictional creator of the Foundation
      in Asimov's novels: the blueprint that generates foundations.

    - `hangar/blueprints/agents/` — one blueprint per agent. Each file
      is the only full description of its agent; `construct` compiles
      it into the format the detected AI-Assisted Tool expects. One
      line each here, the rest is in the file:

      - `the-architect.md` — **The Architect**, the entry point:
        decides which agent a task belongs to, and owns Foundation
        Sync (below). Its current role is recorded in ADR 0002.
      - `programmer.md` — explains first, then implements in small
        pieces.
      - `tester.md` — verifies what `programmer` just built
        (provisional name).
      - `deneir.md` — watches a project's evolution and writes
        observations. Named after the god of writing and
        record-keeping in Forgotten Realms.
      - `writer.md` — shapes raw material into diary entries or
        articles.
      - `researcher.md` — grounds a claim in real, checked sources
        before it is trusted.

      Identifiers are kebab-case (`the-architect`). A teacher and a
      designer are still planned for later.

    - `hangar/blueprints/shared/agent-rules.md` — the rules every
      agent follows, written once: "recognize and refer" and "show
      first, then write". `construct` appends this file to each
      compiled agent, so no agent blueprint repeats it.

    - `hangar/blueprints/skills/` — one blueprint per skill:
      `write-diary.md`, `write-article.md` and `loop-status.md`. These
      are the only skills this harness provides; `construct` is not
      one of them.

    - `hangar/blueprints/adapters/` — one adapter per supported
      AI-Assisted Tool (see "Terminology"): `claudecode.md` and
      `opencode.md`. Each describes only its own tool, and everything
      `construct` compiles for one tool speaks only of that tool:
      other AI-Assisted Tools may appear in the harness's own files as
      examples, never in a compiled output.

  - `hangar/docs/` — records about the blueprints themselves and the
    harness-engineering process, kept apart from `docs/` at the
    project root, which holds the records of the Free Wings project.

## Specs, and how `grilling` feeds them

Free Wings itself follows this pattern, and target projects scaffolded
by the harness may adopt, adapt, or replace it — see
`hangar/blueprints/HARI-SELDON.md`. The pattern:

Following Spec-Driven Development's core idea — a written spec as the
primary source, with implementation as a regenerable output derived
from it — applied here to individual pieces of implementation work, not
just to this harness's own AI-Assisted-Tool-specific file generation
(which already follows the same principle at the harness level).

A project that adopts the pattern gets a `docs/specs/` folder, alongside
its `docs/decisions/` and `docs/LEARNING_LOG.md`. Three documents, three
distinct roles, deliberately not overlapping:

- **ADR** (`docs/decisions/`) — *why* a broad, cross-cutting technical
  direction was chosen. Rare — one per real architectural fork.
- **Spec** (`docs/specs/`) — *what* one specific piece of implementation
  work will do, written *before* implementing it. One per non-trivial
  unit of work, numbered the same way a project already numbers
  its "pieces" in its own `LEARNING_LOG.md` (e.g. `docs/specs/0012-
  mac-signaling-client.md`), so the two stay easy to cross-reference.
- **`LEARNING_LOG.md`** — *what actually happened*, written after,
  including anywhere the implementation ended up deviating from its own
  spec, and why.

When a piece of work is too large for one spec, a **plan** sits above
the specs: the ordered pieces and which agent takes each one, written
by The Architect as `docs/specs/plan-<short-name>.md`. Same folder as
the specs, told apart by the `plan-` prefix and by having no number —
a plan spans several numbered pieces instead of being one of them.

`grilling` — stress-testing a decision through numbered question
rounds, each with a recommended answer, until nothing is left open — is
the shared mechanism feeding both ADRs and specs. It is a practice, not
one of this harness's skills: there is no blueprint for it, though a
person's AI-Assisted Tool may offer a skill by that name. Same process,
two destinations depending on scale:

- A big, cross-cutting architecture decision → `grilling` → an ADR.
- A specific piece's implementation approach → `grilling` → a spec.

Not every spec needs a full `grilling` session. A piece of work with no
real open branches can just get a short spec written directly — the
same way a low-risk, reversible decision doesn't need a whole question
round of its own. `grilling` earns its place when a spec has genuine
open decisions in it, not as a mandatory ritual for every piece of work
regardless of size.

The `programmer` blueprint is where this is applied: a spec before any
non-trivial piece of work. A downstream project may adapt or replace
it, along with the rest of the pattern.

## Foundation Sync — keeping the harness itself from drifting

The drift `FOUNDATION.md` exists to prevent in content — tool-specific
files slowly disagreeing, the way Shadow Glass's hand-maintained pair
did — can also happen in the *process*: an agent's behavior changes and
only one of the files describing it gets updated. **Foundation Sync** is
the cascade that prevents it. This section is its only full definition;
The Architect owns running it.

**Scope.** Free Wings itself. A target project regulates itself, runs
the cascade only if its own `FOUNDATION.md` adopted it, and keeps that
`FOUNDATION.md` as the record that survives a change of AI-Assisted
Tool.

**Trigger.** Any dialogue that changes how the harness works: a new,
renamed or retooled agent, a change in what an agent does or may do, a
new skill, a new standing rule. Not triggered by work specific to a
target project, nor by a format detail of one AI-Assisted Tool, which
is fixed in that tool's adapter alone.

**The cascade**, walked in this order:

1. `FOUNDATION.md` — the source. Then whatever the change touches among
   the blueprints, `CONSTRUCT.md` and the adapters; every adapter is
   checked against it.
2. `construct` — regenerate the AI-Assisted-Tool-specific files.
3. `README.md` — the human-facing onboarding document.
4. `docs/LEARNING_LOG.md` — one entry: what changed and why.
5. `docs/learning-*.html` — update a page that covers the change, or
   create one if the change deserves its own explanation.

Alongside the five, a judgment call: The Architect decides whether the
change is a real architectural fork that deserves an ADR (rare — see
"Specs, and how `grilling` feeds them"). A smaller change does not; the
diary entry covers it.

**Verifier.** Grep the repository for the old name, count or path —
never trust memory — and confirm every blueprint still carries its
orientation note.

**Stop rule.** Done when every step has been checked and either updated
or explicitly confirmed as needing no change, and the ADR call has been
made — not when the first file is done.

**Four rules keep the cascade from being skipped** (ADR 0002, restated
in ADR 0003). They bind every agent and every session:

- **Order.** A change that fires the cascade starts in this file. No
  blueprint, adapter or `CONSTRUCT.md` is edited for it first. Whoever
  is asked for such a change stops, says the cascade applies, and
  starts at step 1 or refers to The Architect.
- **Adapters follow, they do not lead.** An adapter's own format detail
  does not fire the cascade; a change here does require checking every
  adapter.
- **Commit gate.** Such a change is committed only once the stop rule
  is met.
- **`construct` checks.** Before compiling, `construct` compares this
  file with the blueprints and names any disagreement in its report,
  without correcting it.

## Loop Status — a visible readout of where a loop actually is

Whenever work is genuinely mid-loop, the person can get a short, honest
readout of where it stands: on demand in any session ("where are we",
"status", or the `loop-status` skill), and proactively only at real
checkpoints — never on every message. The step is an exact "k of n" for
a fixed cascade and an honest qualitative sense otherwise, never a
fabricated number. Target projects may adopt, adapt, or replace it. The
block's format and its reasoning live in
`hangar/blueprints/skills/loop-status.md`.
## Modular & Self-Sufficient Documentation

A permanent, standing requirement for every file this harness produces
— past and future alike, not a one-time pass applied only to what
existed when this was written down. Credited honestly to where it
actually came from: a LaTeX tutorial file this harness's first person
built for someone else's genuine first contact with LaTeX/Overleaf,
deliberately written so any section could be opened and understood on
its own, each one carrying a short inline explanation of what it is and
how it connects to the rest, rather than assuming the reader had already
read everything above it.

Applied here: every blueprint file, skill blueprint, and generated
document should let someone with zero prior exposure to this project
understand, from wherever they start reading, **where they are** (which
file, and what kind of file it is — an agent blueprint? a skill
blueprint? generated output?), **what it does**, and **how it connects
to the rest of the system** (its nearest neighbors, what reads it, what
it reads). In practice, this means a short orientation note near the top
of agent and skill blueprints, present *from the moment a new one is
created*, not added afterward as a separate cleanup step (see any
`hangar/blueprints/agents/*.md` or `hangar/blueprints/skills/*.md`
file for the actual pattern); it means every agent applies the same
standard when explaining something to a person who doesn't yet
understand it — assume no prior context, orient before explaining,
don't presume the concept has already been introduced; and it means The
Architect's own Foundation Sync verifier (above) checks that a new or
changed file actually carries this orientation note before treating a
cascade as complete.

## Terminology — defined precisely, corrected once already

- **AI-Assisted Tool**: the coding environment this harness runs
  inside — Claude Code, OpenCode, Cursor, or any similar agentic
  coding tool. Distinct from an *agent's tools* (the instruments an
  agent uses — file reading, web search, shell execution, and so on).
  When this document says "AI-Assisted Tool", it means the coding
  environment.
- **Skill**: a repeatable, on-demand procedure. No persistent
  reasoning or memory of its own between separate invocations — one
  job, then done. It still has the full model's judgment for the turn
  it runs in; it just doesn't carry an ongoing context the way an
  agent does.
- **Agent**: a delegated worker with its own reasoning and context,
  suited to open-ended or interpretive work, not just mechanical,
  repeatable procedures. Each agent has a blueprint in
  `hangar/blueprints/agents/`; `construct` compiles it into the
  agents directory of each AI-Assisted Tool, following that tool's own
  configuration as described in its adapter. (Note: "subagent" is
  **not** part of this harness's vocabulary — it is a concept inside
  specific AI-Assisted Tools, with a meaning defined by each tool, and
  belongs only in that tool's adapter.)
- **Blueprint**: an AI-Assisted-Tool-agnostic schema that `construct`
  reads. Most blueprints are *compiled* into AI-Assisted-Tool-specific
  outputs — every file in `hangar/blueprints/agents/` and
  `hangar/blueprints/skills/`. One blueprint, `HARI-SELDON.md`, is not
  compiled: it is read as guidance during scaffolding of a brand-new
  project's `FOUNDATION.md`. Both are blueprints; the difference is
  what `construct` does with them.
- **Adapter**: a description of how `construct` adapts the harness's
  AI-Assisted-Tool-agnostic blueprints to one specific AI-Assisted
  Tool — where the compiled outputs go, what file names that
  AI-Assisted Tool expects, and any format quirks. One per supported
  AI-Assisted Tool. Lives in `hangar/blueprints/adapters/<tool>.md`.
  Read as a rule *during* compilation; not compiled into an output
  itself, and not a skill (a skill is invoked directly by the user or
  an agent — an adapter is read by `construct` in the middle of its
  own execution). Adding support for a new AI-Assisted Tool means
  adding a new adapter, not editing `CONSTRUCT.md`.
- **Field Notes**: standalone, self-contained HTML pages with visual
  explanations of a concept — opened directly in a browser from the
  local filesystem. Never published through an AI-Assisted-Tool-
  specific publishing feature (Claude Code's `Artifact` tool is the
  current example of such a feature this harness deliberately doesn't
  use for this purpose).
- **Snapshot**: a dated, visual record of a project's directory
  structure at a specific milestone — the *shape* of a codebase at a
  point in time, not its content.
- Avoid the word **"artifact"** anywhere in this harness's own
  vocabulary — it collides with AI-Assisted-Tool-specific features of
  the same name (Claude Code's `Artifact` tool being the current
  example), and using the same word for two different things would be
  ambiguous every time it came up.

## Pedagogical approach

Free Wings itself, and every fork or contribution to this harness,
follows this rule when an AI assistant works with a person on it:
explain the relevant concept and its trade-offs in plain terms — what
the choice is, what each option changes, why it matters — *before*
asking the person to decide, so they have an actual basis to decide
from, rather than being asked to choose blind. When a decision is both
low-risk and easily reversible, the assistant may instead just make a
reasonable choice and state its reasoning clearly, leaving room to
veto, rather than blocking progress on an answer the person has no way
to evaluate confidently yet. This is Freire's dialogic principle,
applied literally to working with an AI instead of a classroom.
Downstream projects scaffolded by this harness may adopt, adapt, or
replace it — see `hangar/blueprints/HARI-SELDON.md`.

## Bilingual by design

This harness, and the diary written under it, are meant to work
naturally in both Portuguese and English — not as word-for-word
translations of each other, but each carrying the same real meaning in
a way that sounds natural in that specific language. This applies
especially to naming: a good name should sound right and mean something
real in both languages, not merely survive translation.

## Commit message convention

Conventional Commits (`type(scope): description`, lowercase, imperative
mood) in Free Wings itself and in any fork or contribution to the
harness: `feat`, `fix`, `refactor`, `docs`, `style`, `test`, `chore`,
`perf`, `ci`.

**Granularity:** Commits should be atomic and focused on a single
logical change. Whenever possible, one commit should correspond to one
file (or to a set of changes that are genuinely inseparable, such as
adding a file and its associated index). Avoid commits that mix
unrelated changes across multiple files. This keeps the history clean,
makes reverts easier, and helps future contributors understand the
evolution of the project. If a change spans multiple files but they are
interdependent (e.g., renaming a function and updating its callers), a
single commit is acceptable — but the preference is for small, focused
commits that are easy to review and understand.

## Current status

Founded 2026-09-05, named **Free Wings** (*Asas Livres*) on 2026-09-06.
Six agent blueprints, one shared rules file, three skill blueprints and
two adapters (`claudecode.md`, `opencode.md`) are built; `construct`
compiles them. Shadow Glass is the first project sitting under this
harness, with its own `FOUNDATION.md` and generated files. Since
2026-10-06 each fact has one home and the other files point to it
(ADR 0003).
generated from it.