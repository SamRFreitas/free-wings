# The Foundation

> The source. Not read automatically by any AI tool — compiled into
> whatever each tool actually needs (`CLAUDE.md`, `AGENTS.md`, and
> whatever else shows up later) by the `construct` skill. This file is
> the one place the *reasoning* lives in full; the generated files are
> optimized excerpts of it, not the other way around.

Every project under this harness — this Harness Journal repository
included — has exactly one `Foundation.md`, tool-agnostic by design.
Nothing in this file should ever assume a specific AI tool is reading
it; the moment it does, that content has drifted out of the Foundation
and belongs in a generated, tool-specific file instead.

## What this project is

Harness Journal is the general, reusable layer of a pattern meant to
apply to *every* project, not just itself: each project keeps its own
`Foundation.md` (dense, complete, tool-agnostic) as its single source of
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
`Foundation.md` directly, by absolute path, to understand that project —
no bespoke per-project adapter file needs to be hand-written and kept in
sync; every project having its own `Foundation.md` already *is* the
adapter.

The analogy worth keeping in mind whenever this pattern feels
over-engineered: a compiler has one front-end (the part that understands
the source language, written once) and many pluggable back-ends (one per
target architecture). This `Foundation.md`, in any project, plays the
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

This project's own final proper name is still undecided as of this
writing — the working name "Harness Journal" is used in the meantime,
with the intention that whatever name is eventually chosen carries this
same spirit, directly or indirectly, without needing to spell out
"Freire" or "Santos Dumont" every time it's invoked.

## Structure

- `Foundation.md` (this file) — the one source of truth, tool-agnostic,
  as dense and complete as it needs to be. Never generated; always
  hand-written and hand-edited directly.
- `CLAUDE.md`, `AGENTS.md`, and any future tool-specific file — generated
  from this Foundation by the `construct` skill, each one optimized for
  its specific reader (a weaker model reading `AGENTS.md`, for instance,
  gets more explicit, less-compressed instructions than a stronger one
  reading `CLAUDE.md` — the same underlying truth, different
  compression).
- `docs/decisions/` — ADRs about the harness itself (not about any one
  target project — those live in that project's own `docs/decisions/`).
- `docs/LEARNING_LOG.md` — the diary: one entry per session, at the
  meta level of harness engineering and the learning process, not one
  project's narrow technical details.
- `.claude/agents/` — general-purpose subagents, named after real
  project roles, reusable by any target project: a **programmer**
  (writes code and explains the reasoning and the implementation
  together, not one before the other) and an **observer** (watches a
  target project's evolution over time and documents it, producing raw
  material for diary entries and later publications) are planned first;
  a researcher, a writer, a teacher, a designer are planned for later.
- `.claude/skills/` — repeatable, on-demand procedures. `construct`
  (name borrowed from *Neuromancer*, where a "construct" is a stored
  recording of a person's skills and knowledge, loaded up when needed —
  conceptually close to what this skill does with a project's
  `Foundation.md`) reads a project's `Foundation.md`, detects which
  AI tool(s) are actually in use there, and generates or updates that
  project's `CLAUDE.md`/`AGENTS.md`/etc. from it. Exact design still
  being worked out as of this writing — see the harness's own
  `docs/LEARNING_LOG.md` for the reasoning trail.

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

Founded 2026-09-05. Working name "Harness Journal" pending a final,
Freire/Santos-Dumont-inspired name. Shadow Glass is the first project
intended to sit under this harness — it does not have its own
`Foundation.md` yet, and the `construct` skill described above has not
been built yet, so nothing in the "Structure" section past this file is
confirmed working in practice. The `CLAUDE.md`/`AGENTS.md` written
before this file existed are being treated as a first draft, superseded
by whatever `construct` eventually generates from this Foundation.
