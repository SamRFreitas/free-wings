# Hari Seldon — Foundation Blueprint for New Projects

> This is a blueprint, not the Foundation itself. It is passed as an optional parameter to `construct` — `@CONSTRUCT @HARI-SELDON` — when the target is another project rather than Free Wings itself. In that mode, `construct` reads this blueprint and walks the person through a dialogue that fills each section, producing the target project's real `FOUNDATION.md`. All content below must come from a real conversation between the person and their AI assistant — never filled in by guessing.

<!--
Orientation — where you are: this file lives inside `hangar/blueprints/`, alongside its siblings `agents/`, `skills/`, and `adapters/`. None of them execute on their own; they exist to be read by `construct`.
What this file does: it provides the section skeleton for a new project's `FOUNDATION.md`. It is the parameter that tells `construct` the target is another project — when `construct` is invoked with `@CONSTRUCT @HARI-SELDON`, it reads this blueprint and walks the person through a dialogue that fills each section with real content.
How it connects: passed as a parameter to `construct`. Together, `construct` + `HARI-SELDON` scaffold a new project's `FOUNDATION.md`. Without HARI-SELDON as the parameter, `construct` operates on Free Wings itself and does not read this file. Named after Hari Seldon, the fictional creator of the Foundation in Asimov's novels — the blueprint that generates foundations.
A note on scope: this blueprint deliberately does not carry Free Wings' own identity-level content (its name, its Freire/Dumont philosophy, its bilingual-by-design commitment). Those belong to Free Wings itself. What this blueprint does offer is the harness's standing rules as optional inheritance — see "Harness conventions" below.

Relationship to the other two pillars:
- `FOUNDATION.md` — the source of truth of whichever project I help scaffold. My job is to produce the target project's own `FOUNDATION.md`; I do not produce Free Wings' own.
- `CONSTRUCT.md` — the bootstrapper that receives me as a parameter. It reads me during scaffolding, and together we produce the target project's `FOUNDATION.md`.
- Me (`HARI-SELDON.md`) — the optional parameter that tells `construct` the target is another project.

This is a blueprint, not a file to copy by hand. The actual content in each section below has to come from a real conversation between the person and their AI assistant. Nothing below should ever be filled in by guessing.
-->

## What this project is

[One or two paragraphs: what does this project actually do, and why does it exist? Ask the person directly — don't infer from a name.]

## Purpose / philosophy

[Why does this project work the way it does — any values or principles that shape how decisions get made here, beyond just the technical goal? This is where a project states its own identity, independent of any harness defaults.]

## Structure

[What does this project's repository actually contain? List the real top-level folders/files once they exist, with a one-line reason for each — don't describe a structure that doesn't exist yet.]

## Language convention

[What language does conversation happen in? What language do written files (code comments, docs, commit messages) use? These aren't always the same — ask, don't assume. Not every contributor or AI-Assisted Tool works in English; state the project's choice explicitly.]

## Commit convention

[Free Wings' own default is Conventional Commits (`type(scope): description`, lowercase, imperative). Confirm whether this project adopts that default, adjusts it, or replaces it with its own.]

## Harness conventions — inherit or define your own

[The Free Wings harness defines several standing rules. A project under the harness may choose to inherit each one by default, adapt it to fit, or replace it with its own convention. Walk through each with the person — don't assume inheritance. For the full definition of each, see the Free Wings `FOUNDATION.md`.]

- **Pedagogical approach** — [default: explain a concept and its trade-offs before asking for a decision, unless the decision is low-risk and reversible. Adopt, adapt, or replace?]

- **Terminology** — [default: the harness glossary defines AI-Assisted Tool, Skill, Agent, Blueprint, Adapter, Field Notes, and Snapshot, and avoids the word "artifact". Adopt, adapt, or replace?]

- **Modular & Self-Sufficient Documentation** — [default: every file orients the reader — where they are, what it does, how it connects to the rest. Adopt, adapt, or replace?]

- **Specs and grilling** — [default: a written spec precedes any non-trivial implementation; grilling stress-tests open decisions; ADRs capture architectural forks, specs capture individual pieces. Adopt, adapt, or replace?]

- **Foundation Sync** — [default: a cascade runs when the harness itself changes (FOUNDATION.md → construct → README → LEARNING_LOG → learning pages → optional ADR). Adopt, adapt, or replace?]

- **Loop Status** — [default: a short, honest readout of mid-loop state (trigger, topology, k-of-n, context, stop rule) is available on demand. Adopt, adapt, or replace?]

## Status

[Founding date. What exists so far, what doesn't yet. Update this section honestly as the project actually progresses — it's meant to stay current, not describe an aspiration.]