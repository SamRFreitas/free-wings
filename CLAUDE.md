# Harness Journal

> A research/learning diary about harness engineering, AI-assisted
> development, and free access to technology — written while building
> real, separate projects (Shadow Glass first) under it.

## What this is

Harness Journal is not itself a piece of software with a runtime — it's
the **high-level layer** in a two-level system:

- **This repository (Harness Journal)**: general, project-agnostic —
  teaching philosophy, diary/field-notes formats, naming conventions,
  and reusable agents/skills any project can plug into. Never contains
  code specific to one target project.
- **Each target project's own repository** (Shadow Glass is the first):
  keeps its own identity, its own `CLAUDE.md`, its own history — and
  gets one small "helper" agent living *inside that project's own repo*
  that imports this Foundation's principles and adapts them to that
  project's specifics.

This mirrors a pattern already familiar from compilers: one stable
front-end (this repository), many pluggable back-ends (one helper agent
per target project). Adding a new project under this harness later means
writing one new small adapter file in *that* project's repo — not
rewriting anything here.

## Philosophy — why this exists, not just what it does

Two people's ideas shaped this project's spirit, and are worth keeping
visible rather than buried in a single naming decision:

- **Paulo Freire** — dialogic education over "banking" education.
  Knowledge isn't deposited into a passive learner; it's built together,
  in conversation. This is why every project under this harness follows
  a rule of "explain before deciding" (see Pedagogical approach below) —
  it's the same principle Freire described, applied to working with an
  AI instead of a classroom.
- **Alberto Santos Dumont** — open, public, unpatented work, done in
  view of everyone, on the belief that technical progress should
  benefit everyone, not be gatekept. This is why every project under
  this harness defaults to a public repository, a visible decision
  record (ADRs), and a diary of what was actually learned — including
  the dead ends, not just the finished result.

The project's own final name (still pending) is meant to carry this
spirit — directly or indirectly — without needing to spell out "Freire"
or "Dumont" explicitly every time.

## Structure

- `docs/decisions/` — ADRs for decisions about the harness itself (not
  about any one target project — those stay in that project's own
  `docs/decisions/`).
- `docs/LEARNING_LOG.md` — the diary itself: one entry per session,
  chronological, in the same spirit as a target project's own learning
  log, but reflecting on the harness-engineering/meta level, not just
  one project's technical details.
- `.claude/agents/` — general-purpose agents, named after real project
  roles (a *programmer*, an *observer* who documents a project's
  evolution, and more to come — a researcher, a writer, a teacher, a
  designer), reusable by any target project, not tied to one.
- `.claude/skills/` — repeatable, on-demand procedures. `construct` is
  the first one: reads a target project's `CLAUDE.md`/`AGENTS.md` and
  scaffolds that project's own helper agent from it.

## Terminology (kept precise on purpose — corrected once already this session)

- **Skill**: a repeatable, on-demand procedure, invoked directly
  (`/name`). No persistent reasoning of its own between invocations —
  it does one mechanical job and finishes.
- **Agent** (subagent): a delegated worker with its own reasoning and
  context, suited to open-ended or interpretive tasks, not just
  mechanical ones.
- **Field Notes**: the name for visual, standalone HTML explainer pages
  (self-contained files, opened directly in a browser — never published
  via Claude's own `Artifact` tool, which is a different, specific
  feature this project deliberately doesn't use for this purpose).
- **Snapshot**: a dated, visual record of a project's directory
  structure at a specific milestone — the "shape" of a codebase at a
  point in time, not its content.
- Avoid the word **"artifact"** for anything in this harness's own
  vocabulary — it collides with Claude Code's `Artifact` tool (a
  specific claude.ai publishing feature this harness's projects
  explicitly don't use for Field Notes/Snapshots), and would be
  ambiguous every time it came up.

## Pedagogical approach

Same rule as every project under this harness: explain the concept and
the trade-off before asking for a decision, or — when the decision is
low-risk and reversible — just decide and state the reasoning, leaving
room to veto. No code or structure gets built without the person
understanding why first. This is Freire's dialogic principle, applied
literally.

## Bilingual by design

This harness and its diary are meant to work in both Portuguese and
English — not as direct translations of each other, but each carrying
the same meaning naturally in its own language. Names, in particular,
are chosen to work (sound right, mean something real) in both.

## Commit convention

Same as every project under this harness: **Conventional Commits**
(`type(scope): description`, lowercase, imperative mood — `feat`, `fix`,
`refactor`, `docs`, `style`, `test`, `chore`, `perf`, `ci`).

## Status

Just founded (2026-09-05), still using "Harness Journal" as a working
name while the final Freire/Santos-Dumont-inspired name is decided
separately. Shadow Glass is the first (and so far only) project meant to
sit under this harness once its own helper agent is built and the
cross-repository context mechanism (`@/absolute/path/CLAUDE.md` imports)
is confirmed to actually work in practice — not yet tested end to end.
