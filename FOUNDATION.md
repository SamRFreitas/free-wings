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
- `CONSTRUCT.md` — the bootstrapper itself. Reads this `FOUNDATION.md`
  and the `hangar/blueprints/`, then generates/updates the AI-Assisted-
  Tool-specific files for the environment it detects. This file is the
  orchestrator — not a skill, not a script — and lives at the project
  root. It does not hardcode per-AI-Assisted-Tool logic; that logic
  lives in `hangar/blueprints/adapters/`. Invoked as `@CONSTRUCT` alone,
  it bootstraps Free Wings itself; invoked as `@CONSTRUCT @HARI-SELDON`,
  it scaffolds another project's `FOUNDATION.md` first and then
  bootstraps that project. See `CONSTRUCT.md` for the full invocation
  contract.
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
    reads. It contains blueprints (`agents/`, `skills/`, and
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

    - `hangar/blueprints/agents/` — one markdown file per agent
      (`programmer.md`, `tester.md`, `deneir.md`, `writer.md`,
      `researcher.md`, `the-architect.md`). These are the canonical,
      AI-Assisted-Tool-agnostic descriptions; `construct` compiles each
      into the format the detected AI-Assisted Tool expects, guided by
      that AI-Assisted Tool's adapter in `hangar/blueprints/adapters/`.
      The agents, described in full:

      **programmer** — explains and shows reasoning first, then
      implements while still teaching — divided into small pieces,
      one function or one command at a time, splitting anything
      complex further, checking the person's own understanding along
      the way and sometimes letting them attempt a piece first.

      **tester** — tests what `programmer` just built, same
      explain-first and divide-and-conquer style — what will be
      tested, why, how, shown transparently while it runs;
      provisional name, may become a shorter combined
      "reviewer+tester" name later.

      **deneir** — read-only about a target project, watches its
      evolution, writes only to its own `docs/observations/`;
      captures session identity first — which model, from a reliable
      source only, never by asking the model to self-report — plus a
      quick harness-state note, then a small set of
      research-grounded signals: task shipped, time vs. estimate,
      whether context needed a manual mid-task correction, which
      agents/skills were used and how each went, a subjective note —
      never a scored formula, see its own file's "Session identity"
      and "Grounded signals". Named after Deneir, the god of writing
      and record-keeping in Forgotten Realms, whose central tenet —
      "information that is not recorded and preserved is information
      lost" — is the agent's own functional reason to exist.

      **writer** — shapes raw material into diary entries or
      articles, general themes kept separate from
      explicitly-labeled project/person-specific parallels.

      **researcher** — grounds a claim in real, checked sources
      before it's trusted — exists directly because of a real
      mistake: asserting SDD's fit before verifying its actual
      definition; when validating or comparing data/metrics
      specifically, it stops at a comprehension check instead of
      recommending a next step, and only proceeds once the person's
      understanding is actually confirmed — see its own file's
      "Validation checkpoint" section.

      **The Architect** — file and invocation identifier both
      `the-architect` (this harness keeps identifiers kebab-case, a
      convention that happens to match what some AI-Assisted Tools
      require for a subagent's `name:` field; the exact AI-Assisted-
      Tool-specific validation rule lives in the tool's adapter under
      `hangar/blueprints/adapters/`, not here. "The Architect" itself
      isn't a valid identifier because of the space and capitals, but
      the hyphenated `the-architect` is, so file and identifier
      match) — the entry point: decides which agent a task belongs
      to, or says plainly when it fits none of them, and also owns
      Foundation Sync, see below.

      A teacher and a designer are still planned for later.

      Every agent above follows one added standing rule, "recognize
      and refer": when a request falls outside an agent's own scope,
      say so and name which other agent fits better, instead of
      attempting the work anyway or staying silent about the
      mismatch. This isn't a flat list of equally-distant roles,
      though — the same way a front-end engineer and a back-end
      engineer share far more tools, process, and vocabulary than
      either shares with a designer, some pairs here are genuinely
      closer to each other than to the rest: **deneir ↔ writer**
      (deneir's output already exists specifically to become
      writer's raw material), and a three-agent chain
      **researcher → programmer → tester** (grounding a decision is
      where researcher hands off to programmer, before implementing;
      verifying it worked is where programmer hands off to tester,
      right after — programmer has two nearest neighbors, one on each
      side). A referral should go straight to the nearest neighbor
      when it can resolve the request alone, rather than looping back
      through `the-architect` by default — `the-architect` is for the
      genuinely unclear cases and the ones spanning more than one
      hop, not every mismatch. See `the-architect.md`'s "Proximity
      between agents" for the full reasoning.

    - `hangar/blueprints/skills/` — one `.md` file per skill, each
      defining a repeatable, on-demand procedure. Currently:
      `write-diary.md` (reuses the `writer` agent's own file rather
      than duplicating its themes), `write-article.md` (same reuse),
      and `loop-status.md` (see "Loop Status" above). These are the
      only skills this harness provides. (The `construct` is **not**
      a skill — it is the bootstrapper, located at the root as
      `CONSTRUCT.md`.)

    - `hangar/blueprints/adapters/` — one file per supported
      AI-Assisted Tool, each one an **adapter**: a description of how
      `construct` adapts the harness's AI-Assisted-Tool-agnostic
      blueprints to that specific AI-Assisted Tool — where the
      compiled outputs go, what file names that AI-Assisted Tool
      expects, and any format quirks. Two are provided by default —
      `claude.md` (for Claude Code) and `opencode.md` (for OpenCode)
      — both already part of the repository, serving double duty: as
      active adapters *and* as working demonstrations of how easily a
      new AI-Assisted Tool can be supported. Adding a new adapter is
      a single file, not a change to `CONSTRUCT.md` or to the
      Foundation. This is where AI-Assisted-Tool-specific compilation
      logic lives, deliberately kept out of those two files.

  - `hangar/docs/learnings/` — learnings about the blueprints
    themselves and the harness-engineering process — distinct from
    `docs/LEARNING_LOG.md` at the project root, which is the session
    diary of the Free Wings project.

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

`grilling` (the skill that stress-tests a decision through numbered,
recommendation-attached question rounds until nothing is left open) is
the shared mechanism feeding both ADRs and specs — same process, two
different destinations depending on scale:

- A big, cross-cutting architecture decision → `grilling` → an ADR.
- A specific piece's implementation approach → `grilling` → a spec.

Not every spec needs a full `grilling` session. A piece of work with no
real open branches can just get a short spec written directly — the
same way a low-risk, reversible decision doesn't need a whole question
round of its own. `grilling` earns its place when a spec has genuine
open decisions in it, not as a mandatory ritual for every piece of work
regardless of size.

By default, the `programmer` agent blueprint (see
`hangar/blueprints/agents/` above) writes a spec before implementing
any non-trivial piece of work, and references that spec while
implementing — formalizing what was, until now, a decision that only
ever lived in conversation and then disappeared once the implementation
shipped. A downstream project may adapt or replace this behavior, along
with the rest of the pattern.

## Foundation Sync — keeping the harness itself from drifting

The same drift problem `FOUNDATION.md` exists to solve at the content
level — AI-Assisted-Tool-specific files slowly disagreeing with each
other, the way Shadow Glass's own pair did by hand — can also happen at
the *process* level: an agent's behavior changes, a new one gets added,
a new standing rule gets adopted, and only one of the files describing
it actually gets updated in the moment. **Foundation Sync**, owned by
**The Architect** (`the-architect.md`), is the trigger for catching that
before it happens: any dialogue that changes how this harness itself
works — a new/renamed/retooled agent, a new skill, a new standing rule
— fires a fixed cascade, walked in order, not chosen freely:
`FOUNDATION.md` first (the source), then `construct` (regenerating the
AI-Assisted-Tool-specific configuration files — for example `CLAUDE.md`,
`AGENTS.md`), then `README.md` (the human-facing onboarding doc), then
a `docs/LEARNING_LOG.md` entry, then any `docs/learning-*.html` page
(updated, or created new if the change deserves its own explanation and
none exists yet). The cascade isn't done at the first file — it's done
once every file in it has actually been checked, verified with a real
grep for the old name/count/path rather than trusted from memory, and
updated or explicitly confirmed as not needing a change. A sixth,
judgment-based step sits alongside the fixed five: **The Architect**
also decides whether the change is significant enough — a real
architectural fork, not every small addition — to also deserve its own
new entry in `docs/decisions/`, following this harness's existing "rare,
roughly one per genuine fork" ADR standard (see "Specs, and how
`grilling` feeds them" above). A smaller change doesn't need one; the
`LEARNING_LOG.md` entry already covers it. Full detail in
`the-architect.md`'s own "Foundation Sync" section.

## Loop Status — a visible readout of where a loop actually is

Free Wings itself uses this readout in any session — not one specific
agent's job — and target projects scaffolded by the harness may adopt,
adapt, or replace it. Whenever work is genuinely mid-loop (a Foundation
Sync cascade running, a spec-driven implementation broken into several
pieces, any multi-step process with real state), the person should be
able to get a short, honest readout of: what triggered
this (**trigger**), who/what is acting right now and which agents/skills
were already used (**topology** in progress), which step this is — an
exact "k of n" for a real fixed cascade, an honest qualitative sense for
open-ended dialogic work, never a fabricated number — plain-language
context for what this part is actually about, and what happens next
(the **stop rule** still pending). These map directly onto loop
engineering's own four structural pieces (see "Loop engineering" and
`docs/reading-list.md`) — this is that same anatomy, made visible on
request, not a separate invention.

This is available on demand, always — asking "where are we," "status,"
or invoking the `loop-status` skill directly gets this readout in any
session. It's also shown proactively, but only at real checkpoints (the
start and end of a multi-step cascade or a multi-piece implementation),
not on every message — a status block on every turn would be exactly
the kind of context bloat `deneir`'s own grounded signals exist to
watch for. See `hangar/blueprints/skills/loop-status.md` for the
exact block format and the full reasoning.

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
- **Skill**: a repeatable, on-demand procedure, invoked directly
  (`/name`). No persistent reasoning or memory of its own between
  separate invocations — one mechanical job, then done. (Note: "no
  persistent memory between invocations" doesn't mean "no reasoning at
  all" — a skill still has the full model's judgment available for the
  turn it runs in; `construct` genuinely rewrites/optimizes text, it
  just doesn't carry a separate ongoing context the way an agent does.)
- **Agent** (subagent): a delegated worker with its own reasoning and
  context, suited to open-ended or interpretive work, not just
  mechanical, repeatable procedures.
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
Six agent blueprints (`programmer`, `tester`, `deneir`, `writer`,
`researcher`, `The Architect`), three skill blueprints (`write-diary`,
`write-article`, `loop-status`), and two adapters (`claude.md`,
`opencode.md`) are built and tested live. The `construct` bootstrapper
(`CONSTRUCT.md`) reads `hangar/blueprints/` and compiles AI-Assisted-
Tool-specific outputs, following the per-AI-Assisted-Tool compilation
logic in `hangar/blueprints/adapters/`. Shadow Glass is the first
project sitting under this harness, and now has its own real
`FOUNDATION.md`, with AI-Assisted-Tool-specific configuration files
generated from it.