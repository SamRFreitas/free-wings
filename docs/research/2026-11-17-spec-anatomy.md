# Spec anatomy — what a spec contains, where the formats come from

> Researched 2026-11-17 by `researcher`. Builds on
> `2026-11-16-harness-vocabulary.md` (Free Wings practices *spec-first*,
> per Böckeler); does not repeat it. Verdicts: **documented** (quoted,
> linked), **inference** (my reasoning), **not found**. Pages were read
> through a fetch tool that summarizes; quotes are as returned.

## Why this was asked

Decision of 2026-11-17: the main agent writes and closes a spec with the
person; a worker (a subagent) receives the closed spec and only executes
it, returning a question if blocked. In Chacal, specs ran from 63 to
1,207 lines and mixed requirements with change history — context the
worker pays for. Two questions: does spec-driven development (SDD) have
its own formats, or is it classic systems analysis? And can Free Wings
have a default spec anatomy that each project may replace?

## 1. The half-remembered practice: requirements engineering

"Systems analysts, with requirements, tables and UML" is, by name:

- **Requirements engineering**, whose classic product is the **SRS
  (Software Requirements Specification)**. **Documented**: IEEE 830-1998
  "The content and qualities of a good software requirements
  specification (SRS) are described", with sample SRS outlines; status
  "Superseded Standard", "Superseded by ISO/IEC/IEEE 29148:2011"
  ([IEEE 830](https://standards.ieee.org/ieee/830/1222/)). The current
  standard is **ISO/IEC/IEEE 29148:2018**, "provisions for the processes
  and products related to the engineering of requirements for systems
  and software products and services throughout the life cycle"; Active
  ([IEEE 29148-2018](https://standards.ieee.org/ieee/29148/6937/)). Its
  published table of contents has an SRS example outline (8.5.2) and SRS
  content (9.6: purpose, scope, product perspective…)
  ([iTeh preview](https://standards.iteh.ai/catalog/standards/iso/8cf2bc2b-8b5e-4907-a82a-d1c5676c9e85/iso-iec-ieee-29148-2018));
  full text paywalled, not read. iso.org returned 403.
- **UML** for the models. **Documented**: OMG UML 2.5.1 (Dec 2017), "a
  graphical language for visualizing, specifying, constructing, and
  documenting the artifacts of distributed object systems"
  ([OMG](https://www.omg.org/spec/UML/)).
- **Use cases** (Cockburn, *Writing Effective Use Cases*, 2000): primary
  actor, trigger, main success scenario, extensions, guarantees.
  **Documented only secondarily** (course readings); the book and
  Cockburn's template PDF were not readable here.

That these are what the person remembers is **inference**. Portuguese:
*Engenharia de Requisitos*, *Especificação de Requisitos de Software
(ERS)* — **inference**, not sourced.

## 2. Does SDD have its own formats? Per tool, and borrowed

No shared SDD format exists; each tool defines its own. **Documented**
by Böckeler: the term "isn't very well defined yet" (see the 11-16
file); a spec is "a structured, behavior-oriented artifact - or a set of
related artifacts - written in natural language"
([Böckeler](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html)).

| Tool | Files | Notation | Borrowed from |
| :- | :- | :- | :- |
| **spec-kit** (GitHub) | constitution → `spec.md` → `plan.md` (+ research, data-model, contracts) → `tasks.md` | User stories, prioritized, "INDEPENDENTLY TESTABLE"; "**Given** … **When** … **Then**"; "**FR-001**: System MUST …"; "[NEEDS CLARIFICATION: …]"; success criteria "technology-agnostic and measurable"; tasks `T012 [P] [US1] … in src/models/…` | user stories (agile), Given/When/Then (BDD), "MUST" statements (classic requirement wording) |
| **Kiro** (AWS) | `requirements.md` (or `bugfix.md`), `design.md`, `tasks.md` | requirements "uses EARS … notation", "WHEN [condition/event] THE SYSTEM SHALL [expected behavior]"; design has "technical architecture, sequence diagrams, and implementation considerations" | EARS (requirements engineering), user stories, sequence diagrams (UML) |
| **Tessl** | `*.spec.md`, YAML frontmatter `name`, `description`, `targets` | API block; plain-language requirement with inline `[@test]` link; `[@generate]` / `@describe` | API contracts, test-linked requirements |

Sources, all **documented**: spec-kit
[spec](https://raw.githubusercontent.com/github/spec-kit/main/templates/spec-template.md),
[plan](https://raw.githubusercontent.com/github/spec-kit/main/templates/plan-template.md),
[tasks](https://raw.githubusercontent.com/github/spec-kit/main/templates/tasks-template.md)
templates; Kiro [Specs](https://kiro.dev/docs/specs/),
[Feature Specs](https://kiro.dev/docs/specs/feature-specs/); Tessl
[spec format](https://tessl.io/registry/tessl-labs/spec-driven-development/files/docs/spec-format.md),
[launch post, 2025-09-23](https://tessl.io/blog/tessl-launches-spec-driven-framework-and-registry/).
Böckeler describes Kiro's requirements as user stories "(in "As a…"
format)" with GIVEN/WHEN/THEN; Kiro's current docs say EARS — the format
changed between versions (**inference** from the two dates).

The borrowed notations, **documented** at their origin:

- **EARS** — Mavin, Wilkinson, Harwood, Novak, "Easy Approach to
  Requirements Syntax (EARS)", IEEE RE'09, pp. 317–322, Rolls-Royce
  ([bib](https://bibtex.github.io/RE-2009-MavinWHN.html)). Patterns:
  "The <system name> shall <system response>"; "While <precondition(s)>,
  …"; "When <trigger>, …"; "If <trigger>, then …"; "Where <feature is
  included>, …" ([Mavin](https://alistairmavin.com/ears/)).
- **Gherkin** — "a concrete example that illustrates a business rule";
  Given = initial context, When = event, Then = observable outcome
  ([Cucumber](https://cucumber.io/docs/gherkin/reference/)).
  Agile Alliance dates the "Given-When-Then template, 2006" and the
  "Role-feature-benefit template, 2001"
  ([Agile Alliance](https://www.agilealliance.org/glossary/user-stories/)).

So: SDD is not a new spec discipline; its formats reuse requirements
engineering and agile/BDD notations (**inference**, supported above).
Böckeler draws the parallel herself for spec-as-source and
model-driven development: "MDD never took off for business
applications", and notes large unclear features need "more specialist
product and requirements skills" (**documented**).

## 3. What lets an agent execute without asking — and what stays out

**Documented:**

- The worker starts blank. "Each subagent starts with a fresh, isolated
  context window. It doesn't see your conversation history"; it works
  from "the delegation prompt Claude writes"; `AskUserQuestion` is
  removed from every subagent
  ([Subagents](https://code.claude.com/docs/en/sub-agents)).
- Four elements. "Each subagent needs an objective, an output format,
  guidance on the tools and sources to use, and clear task boundaries";
  vague tasks led to duplicated work and gaps
  ([multi-agent research](https://www.anthropic.com/engineering/multi-agent-research-system)).
- A check to run. "Give Claude something it can run that comes back
  pass or fail"; review "against SPEC.md, check every requirement is
  implemented and the listed edge cases have tests"; interview the
  person "until you could write the spec", then "start a fresh session
  to build from it, so the build begins with a clean context and a
  written definition of done"
  ([Best practices](https://code.claude.com/docs/en/best-practices)).
- Don't guess. spec-kit: "Mark all ambiguities", "Don't guess"; "No
  speculative or 'might need' features"; specs say "WHAT users need and
  WHY", "Avoid HOW to implement"
  ([spec-driven.md](https://raw.githubusercontent.com/github/spec-kit/main/spec-driven.md)).
- Only what the agent can't see. CLAUDE.md "works best when it only
  holds what Claude couldn't work out from the code itself" (Best
  practices).
- Machine-checkable lists resist drift: the feature list is JSON
  because "the model is less likely to inappropriately change or
  overwrite JSON files compared to Markdown files"
  ([long-running harnesses](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)).

**Inference** — what stays out of a spec handed to a worker:

- Change history → `LEARNING_LOG.md` (Free Wings already says
  deviations go there).
- Rationale for a broad direction → ADR. Nygard's ADR holds Context,
  Decision, Status, Consequences
  ([Nygard, 2011](https://www.cognitect.com/blog/2011/11/15/documenting-architecture-decisions),
  **documented**); a spec needs only one line, "why", linking to it.
- The grilling dialogue itself → only its outcomes (decisions) enter
  the spec.
- Unresolved questions → a closed spec has none; the worker returns
  one when blocked, it does not decide.

## 4. Size and context cost

- A documented length for specs: **not found** in any source checked.
- Nearest documented numbers: CLAUDE.md "target under 200 lines …
  Longer files consume more context and reduce adherence"
  ([Memory](https://code.claude.com/docs/en/memory)); an ADR "should be
  one or two pages long" (Nygard); sub-agents return "often
  1,000-2,000 tokens"; principle: "the smallest set of high-signal
  tokens that maximize the likelihood of your desired outcome", and
  "context rot"
  ([context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)).
- Documented complaints about size: spec-kit's files "were repetitive,
  both with each other, and with the code"; Kiro on a small bug like
  "using a sledgehammer to crack a nut"; "I'd rather review code than
  all these markdown files" (Böckeler). spec-kit keeps detail in
  separate files to avoid "specifications becoming unreadable code
  dumps".
- **Inference**: treat ~150 lines as a soft ceiling for a closed spec
  (borrowed from the CLAUDE.md figure, not measured for specs); above
  it, split into a `plan-` plus several specs, which Free Wings already
  provides for.

## 5. Proposal — default anatomy, overridable per project (inference)

Everything in this section is **inference**: a design built on the
sources above, not a sourced result.

### Default Free Wings spec

```
# NNNN — <title>
Status: draft | closed | done   ·   Links: ADR-…, plan-…

## Goal            — 1–3 lines: what changes for whom, and why (one line)
## Context         — only what the worker cannot read from the repo; paths
## Scope           — In: … / Out: …
## Requirements    — R1, R2…; EARS form when behaviour depends on a
                     state or event ("When …, the X shall …")
## Checks          — per requirement: command, Given/When/Then, or a
                     manual step for `tester`; nothing passes on "looks done"
## Constraints     — must / must not; files not to touch
## Design          — optional: interfaces, data table, diagram
## Open questions  — must be empty to mark `closed`
## Tasks           — optional, ordered, each with a file path
```

Closing rule: `closed` = no open questions, and every R has a check.
The worker receives only a `closed` spec; if blocked, it returns the
question and does not edit the spec. History goes to the diary, broad
rationale to an ADR.

### How a project declares its own

In the project's `FOUNDATION.md`, a short section:

```
## Spec anatomy
Default (Free Wings) | Own: docs/specs/TEMPLATE.md
Modules used: EARS requirements · use cases · Gherkin scenarios ·
data tables · UML (sequence, state, class) · C4 (context, container)
```

- The template file is the single home; `FOUNDATION.md` only points to
  it (ADR 0003's one-home rule).
- Modules go in **Design** or **Requirements** only where the project
  needs them: e.g. Chacal's protocol work might want sequence and state
  diagrams; a UI project, Gherkin scenarios. C4 is "Notation
  independent", at four levels: system context, containers, components,
  code ([C4](https://c4model.com/), **documented**); UML is the
  standard notation (OMG, above). Diagrams as text (e.g. Mermaid) keep
  them diffable — **inference**, not checked.
- `HARI-SELDON.md` already asks "Specs and grilling — Adopt, adapt, or
  replace?"; the anatomy question would join it.

Where the default template lives in Free Wings, and whether this fires
Foundation Sync, is `the-architect`'s call, not this file's.

## Confidence

- Confirmed: requirements engineering / SRS / IEEE 830 → ISO/IEC/IEEE
  29148; UML 2.5.1; EARS origin and patterns; Gherkin; the three tools'
  formats; subagent isolation and the four delegation elements; no
  shared SDD format.
- Likely: that the person's memory is requirements engineering + UML +
  use cases (inference); Cockburn's template fields (secondary only).
- Not found: any documented spec length for agents.
- Inference: section 5 entirely, and the ~150-line ceiling.
