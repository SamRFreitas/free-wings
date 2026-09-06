# The Foundation

**Project: Free Wings** (*Asas Livres*)

> The source. Not read automatically by any AI tool — compiled into
> whatever each tool actually needs (`CLAUDE.md`, `AGENTS.md`, and
> whatever else shows up later) by the `construct` skill. This file is
> the one place the *reasoning* lives in full; the generated files are
> optimized excerpts of it, not the other way around.

Every project under this harness — this Free Wings repository included
— has exactly one `FOUNDATION.md`, tool-agnostic by design.
Nothing in this file should ever assume a specific AI tool is reading
it; the moment it does, that content has drifted out of the Foundation
and belongs in a generated, tool-specific file instead.

## What this project is

**Free Wings** (*Asas Livres*) is the general, reusable layer of a pattern meant to
apply to *every* project, not just itself: each project keeps its own
`FOUNDATION.md` (dense, complete, tool-agnostic) as its single source of
truth, and generates whatever tool-specific instruction files it
actually needs (`CLAUDE.md` for Claude Code, `AGENTS.md` for OpenCode
and similar tools, more as new tools show up) from that one source —
instead of hand-maintaining multiple files that inevitably drift apart,
which is exactly the maintenance burden Shadow Glass's own `CLAUDE.md`/
`AGENTS.md` pair already accumulated by being edited by hand, twice,
every time something changed.

This repository itself is not software with a runtime. It is the
**general layer** — philosophy, formats, naming conventions, reusable
agents and skills — that any target project (Shadow Glass first) can
sit underneath. A general-purpose agent working inside this harness
(a "programmer," an "observer") reads a target project's own
`FOUNDATION.md` directly, by absolute path, to understand that project —
no bespoke per-project adapter file needs to be hand-written and kept in
sync; every project having its own `FOUNDATION.md` already *is* the
adapter.

The analogy worth keeping in mind whenever this pattern feels
over-engineered: a compiler has one front-end (the part that understands
the source language, written once) and many pluggable back-ends (one per
target architecture). This `FOUNDATION.md`, in any project, plays the
front-end's role — general, written once, describing what's actually
true about that project. Each generated `CLAUDE.md`/`AGENTS.md` plays a
back-end's role — the same truth, compiled for one specific reader.

## Philosophy — why this exists, not just what it does

Two real people's ideas shaped the spirit of this project, and stay
worth naming explicitly rather than dissolving silently into a project
name and being forgotten:

**Paulo Freire**, a Brazilian educator, drew a distinction between
"banking" education — a teacher depositing knowledge into a passive
student, who simply receives it — and *dialogic* education, where
teacher and student build understanding together, through real
back-and-forth. Every project under this harness follows the dialogic
model deliberately when working with an AI assistant: the assistant
explains a concept and its trade-offs *before* asking a person to decide
anything about it, rather than deciding quietly and reporting the
decision afterward. This isn't a style preference — it's the concrete,
practical form Freire's idea takes when the "classroom" is a person and
an AI working through a real technical decision together. See
"Pedagogical approach" below for the literal rule this becomes.

**Alberto Santos Dumont**, a Brazilian aviation pioneer, flew his early
aircraft in public — not in secrecy — and deliberately left many of his
inventions unpatented, believing technical progress should benefit
everyone rather than be gatekept by whoever owns the patent. Every
project under this harness defaults to the same posture: a public
repository, a visible written trail of technical decisions (ADRs), and a
diary recording what was actually learned — including the dead ends and
the wrong turns, not only the polished result. Publishing the process,
not just the outcome, is the point.

This project's name, **Free Wings** (*Asas Livres*), carries both
without needing to spell out either name every time it's invoked: "wings"
is Dumont's flight, unmistakably; "free" is Freire's liberation —
the same freedom "banking" education withholds and dialogic education
gives back. Neither half needed the other person's name attached to be
legible; together, they hold both.

## Structure

- `FOUNDATION.md` (this file) — the one source of truth, tool-agnostic,
  as dense and complete as it needs to be. Never generated; always
  hand-written and hand-edited directly.
- `CLAUDE.md`, `AGENTS.md`, and any future tool-specific file — generated
  from this Foundation by the `construct` skill, each one optimized for
  its specific reader (a weaker model reading `AGENTS.md`, for instance,
  gets more explicit, less-compressed instructions than a stronger one
  reading `CLAUDE.md` — the same underlying truth, different
  compression). **Gitignored, not committed** — same reasoning as never
  committing a `build/` folder: a file that's 100% regenerable from a
  tracked source doesn't belong in version control, and committing it
  would silently assume every future contributor needs every supported
  tool's file, rather than generating only the one they actually use.
  This isn't specific to this repository — any project under this
  harness can make the same call, though an existing project (Shadow
  Glass, at the time of this decision) may reasonably keep them
  committed if that's already the established practice there.
- `docs/decisions/` — ADRs about the harness itself (not about any one
  target project — those live in that project's own `docs/decisions/`).
- `docs/LEARNING_LOG.md` — the diary: one entry per session, at the
  meta level of harness engineering and the learning process, not one
  project's narrow technical details.
- `.claude/agents/` — general-purpose subagents, named after real
  project roles, reusable by any target project, not tied to one:
  **programmer** (explains and shows reasoning first, then implements
  while still teaching — divided into small pieces, one function or one
  command at a time, splitting anything complex further, checking the
  person's own understanding along the way and sometimes letting them
  attempt a piece first), **tester** (tests what `programmer` just
  built, same explain-first and divide-and-conquer style — what will be
  tested, why, how, shown transparently while it runs; provisional name,
  may become a shorter combined "reviewer+tester" name later),
  **observer** (read-only about a target project, watches its evolution,
  writes only to its own `docs/observations/`; captures session identity
  first — which model, from a reliable source only, never by asking the
  model to self-report — plus a quick harness-state note, then a small
  set of research-grounded signals: task shipped, time vs. estimate,
  whether context needed a manual mid-task correction, which
  agents/skills were used and how each went, a subjective note — never a
  scored formula, see its own file's "Session identity" and "Grounded
  signals"), **writer**
  (shapes raw material into diary entries or articles, general themes
  kept separate from explicitly-labeled project/person-specific
  parallels),
  **researcher** (grounds a claim in real, checked sources before it's
  trusted — exists directly because of a real mistake: asserting SDD's
  fit before verifying its actual definition; when validating or
  comparing data/metrics specifically, it stops at a comprehension
  check instead of recommending a next step, and only proceeds once
  the person's understanding is actually confirmed — see its own file's
  "Validation checkpoint" section), and **The Architect**
  (file and invocation identifier both `the-architect` — a subagent's
  `name:` field must be kebab-case, verified against Claude Code's real
  docs; "The Architect" itself isn't valid, but the hyphenated
  `the-architect` is, so file and identifier match — the entry point:
  decides which agent a task belongs to, or says plainly when it fits
  none of them, and also owns Foundation Sync, see below). A teacher and
  a designer are still planned for later. Every agent above follows one
  added standing rule, "recognize and refer": when a request falls
  outside an agent's own scope, say so and name which other agent fits
  better, instead of attempting the work anyway or staying silent about
  the mismatch. This isn't a flat list of equally-distant roles, though
  — the same way a front-end engineer and a back-end engineer share far
  more tools, process, and vocabulary than either shares with a
  designer, some pairs here are genuinely closer to each other than to
  the rest: **observer ↔ writer** (observer's output already exists
  specifically to become writer's raw material), and a three-agent chain
  **researcher → programmer → tester** (grounding a decision is where
  researcher hands off to programmer, before implementing; verifying it
  worked is where programmer hands off to tester, right after —
  programmer has two nearest neighbors, one on each side). A referral
  should go straight to the nearest neighbor when it can resolve the
  request alone, rather than looping back through `the-architect` by
  default — `the-architect` is for the genuinely unclear cases and the
  ones spanning more than one hop, not every mismatch. See
  `the-architect.md`'s "Proximity between agents" for the full
  reasoning.
- `.claude/skills/` — repeatable, on-demand procedures. `construct`
  (name borrowed from *Neuromancer*, where a "construct" is a stored
  recording of a person's skills and knowledge, loaded up when needed)
  reads a project's `FOUNDATION.md` and generates/updates that project's
  `CLAUDE.md`/`AGENTS.md`/etc. from it. For a brand-new project with no
  `FOUNDATION.md` yet, `construct` instead scaffolds it automatically
  from the `hangar/` blueprint (folders created directly, never handed
  to the person to copy by hand) and guides the dialogue that produces
  its first real `FOUNDATION.md` — the dialogue itself still can't be
  automated (see "Pedagogical approach"), only the mechanical scaffolding
  can be.
  `write-diary` and `write-article` (both reuse the `writer` agent's own
  file rather than duplicating its themes) and `loop-status` (see "Loop
  Status" above) are the other skills this harness provides so far.
- `hangar/` — the blueprint `construct` reads from when scaffolding a
  brand-new project: a skeleton `FOUNDATION.md` (section prompts, not
  filled-in content) and empty `docs/decisions/`, `docs/specs/`,
  `docs/observations/`, `docs/LEARNING_LOG.md`. Named for where an
  aircraft is prepared before it flies — nothing in here flies on its
  own; it exists to be read by `construct`, never copied by hand.

## Specs, and how `grilling` feeds them

Following Spec-Driven Development's core idea — a written spec as the
primary artifact, with implementation as a regenerable output derived
from it — applied here to individual pieces of implementation work in a
target project, not just to this harness's own `CLAUDE.md`/`AGENTS.md`
generation (which already follows the same principle at the harness
level).

Every target project gets a `docs/specs/` folder, alongside its
`docs/decisions/` and `docs/LEARNING_LOG.md`. Three documents, three
distinct roles, deliberately not overlapping:

- **ADR** (`docs/decisions/`) — *why* a broad, cross-cutting technical
  direction was chosen. Rare — one per real architectural fork.
- **Spec** (`docs/specs/`) — *what* one specific piece of implementation
  work will do, written *before* implementing it. One per non-trivial
  unit of work, numbered the same way a target project already numbers
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

The `programmer` agent (see `.claude/agents/` below) is expected to
write a spec before implementing any non-trivial piece of work in a
target project, and to reference that spec while implementing —
formalizing what was, until now, a decision that only ever lived in
conversation and then disappeared once the implementation shipped.

## Foundation Sync — keeping the harness itself from drifting

The same drift problem `FOUNDATION.md` exists to solve at the content
level — `CLAUDE.md`/`AGENTS.md` slowly disagreeing with each other, the
way Shadow Glass's own pair did by hand — can also happen at the
*process* level: an agent's behavior changes, a new one gets added, a
new standing rule gets adopted, and only one of the files describing it
actually gets updated in the moment. **Foundation Sync**, owned by
**The Architect** (`the-architect.md`), is the trigger for catching that
before it happens: any dialogue that changes how this harness itself
works — a new/renamed/retooled agent, a new skill, a new standing
rule — fires a fixed cascade, walked in order, not chosen freely:
`FOUNDATION.md` first (the source), then `construct` (regenerating
`CLAUDE.md`/`AGENTS.md`), then `README.md` (the human-facing onboarding
doc), then a `docs/LEARNING_LOG.md` entry, then any `docs/learning-*.html`
page (updated, or created new if the change deserves its own explanation
and none exists yet). The cascade isn't done at the first file — it's
done once every file in it has actually been checked, verified with a
real grep for the old name/count/path rather than trusted from memory,
and updated or explicitly confirmed as not needing a change. A sixth,
judgment-based step sits alongside the fixed five: **The Architect**
also decides whether the change is significant enough — a real
architectural fork, not every small addition — to also deserve its own
new entry in `docs/decisions/`, following this harness's existing "rare,
roughly one per genuine fork" ADR standard (see "Specs, and how
`grilling` feeds them" above). A smaller change doesn't need one; the
`LEARNING_LOG.md` entry already covers it. Full detail in
`the-architect.md`'s own "Foundation Sync" section.

## Loop Status — a visible readout of where a loop actually is

Applies to any AI tool working under this harness, in any session — not
one specific agent's job. Whenever work is genuinely mid-loop (a
Foundation Sync cascade running, a spec-driven implementation broken
into several pieces, any multi-step process with real state), the
person should be able to get a short, honest readout of: what triggered
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
the kind of context bloat `observer`'s own grounded signals exist to
watch for. See `.claude/skills/loop-status/SKILL.md` for the exact
block format and the full reasoning.

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

Applied here: every agent file, skill file, and generated document
should let someone with zero prior exposure to this project understand,
from wherever they start reading, **where they are** (which file, and
what kind of file it is — an agent? a skill? generated output?), **what
it does**, and **how it connects to the rest of the system** (its
nearest neighbors, what reads it, what it reads). In practice, this
means a short orientation note near the top of agent and skill files,
present *from the moment a new one is created*, not added afterward as
a separate cleanup step (see any `.claude/agents/*.md` or
`.claude/skills/*/SKILL.md` file for the actual pattern); it means every
agent applies the same standard when explaining something to a person
who doesn't yet understand it — assume no prior context, orient before
explaining, don't presume the concept has already been introduced; and
it means The Architect's own Foundation Sync verifier (above) checks
that a new or changed file actually carries this orientation note before
treating a cascade as complete.

## Terminology — defined precisely, corrected once already

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
- **Field Notes**: standalone, self-contained HTML pages with visual
  explanations of a concept — opened directly in a browser from the
  local filesystem, never published via Claude Code's own `Artifact`
  tool (a different, specific claude.ai-publishing feature this harness
  deliberately doesn't use for this purpose).
- **Snapshot**: a dated, visual record of a project's directory
  structure at a specific milestone — the *shape* of a codebase at a
  point in time, not its content.
- Avoid the word **"artifact"** anywhere in this harness's own
  vocabulary — it collides with Claude Code's `Artifact` tool, and using
  the same word for two different things would be ambiguous every time
  it came up.

## Pedagogical approach

Every project under this harness, including this one, follows this rule
when an AI assistant works with a person on it: explain the relevant
concept and its trade-offs in plain terms — what the choice is, what
each option changes, why it matters — *before* asking the person to
decide, so they have an actual basis to decide from, rather than being
asked to choose blind. When a decision is both low-risk and easily
reversible, the assistant may instead just make a reasonable choice and
state its reasoning clearly, leaving room to veto, rather than blocking
progress on an answer the person has no way to evaluate confidently yet.
This is Freire's dialogic principle, applied literally to working with
an AI instead of a classroom.

## Bilingual by design

This harness, and the diary written under it, are meant to work
naturally in both Portuguese and English — not as word-for-word
translations of each other, but each carrying the same real meaning in
a way that sounds natural in that specific language. This applies
especially to naming: a good name should sound right and mean something
real in both languages, not merely survive translation.

## Commit message convention

Conventional Commits (`type(scope): description`, lowercase, imperative
mood) across every project under this harness: `feat`, `fix`,
`refactor`, `docs`, `style`, `test`, `chore`, `perf`, `ci`.

## Current status

Founded 2026-09-05, named **Free Wings** (*Asas Livres*) on 2026-09-06.
Six agents (`programmer`, `tester`, `observer`, `writer`, `researcher`,
The Architect) and four skills (`construct`, `write-diary`,
`write-article`, `loop-status`) are built and tested live. Shadow Glass
is the first project sitting under this harness, and now has its own real
`FOUNDATION.md`, generated `CLAUDE.md`/`AGENTS.md` from it.
