# Harness Journal

> A research/learning diary about harness engineering, AI-assisted
> development, and free access to technology — written while building
> real, separate projects (Shadow Glass first) under it.

**Note for whichever AI tool is reading this file**: this file exists so
that tools other than Claude Code (which reads `CLAUDE.md`) can follow
the exact same instructions. It is kept in sync with `CLAUDE.md` on
purpose — the two files should always say the same thing, just written
here with less assumed context, since a less capable model may be the
one reading it. If you notice this file and `CLAUDE.md` have drifted
apart, that is a bug — point it out rather than picking one over the
other.

## What this is

Harness Journal is not itself a piece of software with a runtime that
gets deployed anywhere — it is the **high-level, general layer** in a
two-level system made of exactly two kinds of thing:

1. **This repository (Harness Journal)** — contains only general,
   project-agnostic material: the teaching philosophy described below,
   the diary/field-notes formats, naming conventions, and reusable
   agents/skills that any target project could plug into. This
   repository must never contain code, decisions, or content specific to
   one single target project (like Shadow Glass) — that content belongs
   in that project's own separate repository instead.
2. **Each target project's own separate repository** (Shadow Glass is
   the first one) — keeps its own identity, its own `CLAUDE.md`, its own
   commit history, completely independent of this repository. Each such
   project additionally gets one small "helper" agent file that lives
   *inside that project's own repository* (not inside this Harness
   Journal repository) — that helper agent's job is to import this
   Harness Journal's general principles (by reading this repository's
   `CLAUDE.md` from an absolute file path) and apply them specifically
   to that one target project.

An analogy that explains why this split exists: a compiler has one
stable front-end (the part that understands the programming language
itself, written once) and many different pluggable back-ends (one per
target CPU architecture, such as x86 or ARM). This Harness Journal
repository plays the role of the front-end — general, written once. Each
target project's own helper agent plays the role of one back-end —
specific to that one project. Adding a brand-new project under this
harness later should mean writing one new small adapter agent file
inside *that new project's own repository* — it should never require
rewriting anything inside this Harness Journal repository itself.

## Philosophy — why this project exists, not just what it mechanically does

Two real historical people's ideas shaped the spirit of this project.
Both are worth remembering explicitly, not just folded silently into a
project name and forgotten:

- **Paulo Freire**, a Brazilian educator, described a difference between
  two models of education: "banking" education (where a teacher deposits
  knowledge into a passive student, who simply receives it) versus
  "dialogic" education (where teacher and student build understanding
  together, through genuine back-and-forth conversation). This Harness
  Journal project, and every project underneath it, deliberately follows
  the dialogic model when working with an AI assistant: the AI must
  explain a concept and its trade-offs *before* asking the human to
  decide anything about it, rather than silently deciding on the human's
  behalf and only informing them afterward. See the "Pedagogical
  approach" section below for the exact rule this becomes in practice.
- **Alberto Santos Dumont**, a Brazilian aviation pioneer, is known for
  demonstrating his early aircraft flights in public view (rather than
  working in secrecy), and for deliberately not patenting many of his
  inventions, on the belief that technical progress should be shared
  openly and benefit everyone rather than being restricted to whoever
  owns the patent. This is why every project under this harness defaults
  to a public code repository, a visible written record of technical
  decisions (see "ADRs" below), and a diary that records what was
  actually learned along the way — including the mistakes and dead ends,
  not only the polished final result.

This project's own final proper name (a name inspired by Freire and/or
Santos Dumont) has not been chosen yet as of this file's writing — the
working name "Harness Journal" is used in the meantime. Once a final
name is chosen, the intention is for that name to carry this same
philosophical spirit — directly or indirectly — even without spelling
out "Freire" or "Santos Dumont" by name every single time it's used.

## Structure of this repository

- `docs/decisions/` — Architecture Decision Records (ADRs), but only for
  decisions **about the harness itself** — for example, how the
  cross-repository helper-agent mechanism should work. Decisions about
  any one specific target project (such as Shadow Glass) must instead be
  recorded in that target project's own `docs/decisions/` folder, not
  here.
- `docs/LEARNING_LOG.md` — the diary itself. One entry per work session,
  written in chronological order (oldest first or newest first,
  whichever convention gets established — check the file itself for the
  actual current order). Written in the same spirit as a target
  project's own learning log, but focused on the meta level of harness
  engineering and the learning process itself, not just one project's
  narrow technical details.
- `.claude/agents/` — general-purpose subagents. Each one is named after
  a real job role that exists on an actual project team, and is written
  so that it can be reused by any target project, not tied to Shadow
  Glass or any other single project specifically. As of this file's
  writing, the planned agents are: a **programmer** (writes code and
  explains the reasoning behind decisions and the implementation
  together, not one before the other), and an **observer** (watches a
  target project's evolution over time and documents it, producing raw
  material that later becomes diary entries or other public writing).
  More roles are planned for later: a researcher, a writer, a teacher, a
  designer.
- `.claude/skills/` — repeatable, on-demand procedures, each one
  invoked directly by name (for example, typing `/construct` in a
  Claude Code session). The first planned skill, named `construct`
  (a deliberate reference to the novel *Neuromancer*, where a
  "construct" is a stored recording of a person's skills and knowledge,
  loaded up when needed to assist with a task — conceptually similar to
  what this skill does with a target project's own `CLAUDE.md`), reads a
  target project's `CLAUDE.md` and `AGENTS.md` files and automatically
  generates that target project's own helper agent file from them.

## Terminology — defined precisely on purpose, because this was a real point of confusion once already

- **Skill**: a repeatable, on-demand procedure. It is invoked directly
  by the person using it (for example, by typing a slash command such
  as `/construct`). It does not carry any reasoning or memory of its own
  between separate invocations — it performs one mechanical job each
  time it's invoked, and then it's done.
- **Agent** (also called a "subagent" in Claude Code's own
  terminology): a delegated worker that has its own reasoning process
  and its own context, better suited to open-ended or interpretive work
  than to purely mechanical, repeatable procedures.
- **Field Notes**: this project's name for a specific kind of visual,
  standalone HTML page — a self-contained `.html` file, meant to be
  opened directly in any web browser from the local filesystem. These
  pages are deliberately **never** published using Claude Code's own
  built-in `Artifact` tool (a specific feature for publishing pages to
  claude.ai) — that tool is explicitly not used for this purpose in this
  harness or in any project underneath it.
- **Snapshot**: a dated, visual record of a project's directory
  structure at one specific point in time or milestone — showing the
  *shape* of a codebase (which files and folders exist and what each is
  for), not the actual content or code inside those files.
- Avoid using the word **"artifact"** anywhere in this harness's own
  vocabulary for the concepts above. That word specifically collides
  with Claude Code's own `Artifact` tool (the claude.ai publishing
  feature mentioned above, which this harness's projects deliberately do
  not use for Field Notes or Snapshots) — using the same word for two
  different things would create real ambiguity every time it came up in
  conversation or documentation.

## Pedagogical approach — the exact rule, spelled out

Every project under this harness, including this Harness Journal
repository itself, follows this same rule when an AI assistant is
working with the person on it: before asking the person to make a
technical decision, the AI assistant must first explain the relevant
concept and its trade-offs in plain terms — what the choice is, what
each option changes, and why it matters — so the person has an actual
basis on which to decide, rather than being asked to choose blindly.
When a particular decision is both low-risk and easily reversible later,
the AI assistant may instead just make a reasonable choice on its own
and clearly state its reasoning for that choice, while leaving room for
the person to veto or change that choice afterward, rather than blocking
progress on an answer the person has no way to evaluate confidently yet.
This is a direct, practical application of Paulo Freire's dialogic
education principle (described above under "Philosophy"), applied to
working with an AI assistant instead of applied to a traditional
classroom.

## Bilingual by design

This harness, and the diary written under it, are meant to work
naturally in both Portuguese and English — not as word-for-word
translations of one another, but each one carrying the same real meaning
in a way that sounds natural in that specific language. This especially
applies to naming choices: a good name for this project (or for any
piece within it) should sound right and mean something real in both
languages, not merely translate acceptably from one into the other.

## Commit message convention

Every project under this harness, including this Harness Journal
repository itself, uses the **Conventional Commits** format for git
commit messages: `type(scope): description`, written in lowercase, in
the imperative mood (for example: "add X" rather than "added X" or
"adds X"). The allowed types are: `feat` (new functionality), `fix` (bug
fix), `refactor` (a change with no behavior change), `docs`
(documentation only), `style` (formatting only, no logic change), `test`
(adding or changing tests), `chore` (maintenance work, such as updating
a dependency), `perf` (a performance improvement), and `ci` (continuous
integration configuration).

## Current status of this project

This Harness Journal repository was just founded on 2026-09-05, and is
still using "Harness Journal" as a working name while its final,
Freire/Santos-Dumont-inspired proper name is decided in a separate,
dedicated conversation. Shadow Glass (a separate, pre-existing project —
a low-latency remote-access system, Mac to Windows) is the first, and so
far the only, project intended to sit underneath this harness, once that
project's own helper agent file has been written inside Shadow Glass's
own repository, and once the underlying cross-repository context-sharing
mechanism (importing another repository's `CLAUDE.md` via an absolute
file path, using the `@/absolute/path/to/CLAUDE.md` import syntax) has
actually been confirmed to work correctly in practice — as of this
file's writing, that mechanism has not yet been tested end to end.
