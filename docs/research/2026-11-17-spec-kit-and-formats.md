# spec-kit, Scrum, and spec formats without user stories

> Researched 2026-11-17 by `researcher`. Builds on
> `2026-11-17-spec-anatomy.md` (what a spec holds; spec-kit, Kiro and
> Tessl formats; EARS; Gherkin), which this file does not repeat.
> Verdicts: **documented** (quoted and linked), **inference** (my own
> reasoning), **not found**. Pages were read through a fetch tool that
> summarizes them, so quotes are as that tool returned them.

## Why this was asked

Free Wings is choosing a default spec format. The person wants one that
is modern, fits spec-driven development (SDD) and comes from agile
practice, but does not want user stories ("As a… I want… so that…").
They are also considering building on GitHub's spec-kit, and they mix
up "tool" and "skill". Four questions follow.

## 1. Does Scrum require user stories? No

The Scrum Guide (2020 edition, Schwaber and Sutherland), at
[scrumguides.org](https://scrumguides.org/scrum-guide.html).
**Documented:**

- "The Product Backlog is an emergent, ordered list of what is needed
  to improve the product."
- Refinement is "an ongoing activity to add details, such as a
  description, order, and size. Attributes often vary with the domain
  of work."
- "The Scrum framework is purposefully incomplete … Various processes,
  techniques and methods can be employed within the framework."
- The phrase "user story" does not appear in the guide, and it sets no
  format for backlog items.

User stories come from Extreme Programming, the agile method written up
by Kent Beck. Agile Alliance dates the "Role-feature-benefit template"
to 2001 ([glossary](https://www.agilealliance.org/glossary/user-stories/),
**documented**). So dropping user stories does not take you outside
Scrum or outside agile (**inference**, based on the quotes above).

## 2. What spec-kit is

**Documented**, from [the repo](https://github.com/github/spec-kit),
[the docs](https://github.github.io/spec-kit/) and
[the releases page](https://github.com/github/spec-kit/releases):

- It describes itself as an "open source toolkit that gives AI coding
  agents structured processes, reusable templates, and documented
  outcomes." MIT licence. About 150k stars and 2,444 commits. Latest
  releases: 1.4.1 (16 Nov), 1.4.0 (10 Nov), 1.3.3 (9 Nov). It is very
  active.
- **The CLI.** `specify` is a Python program (3.11+), installed with
  `uv tool install specify-cli`. `specify init <project> --integration
  claude` creates a new project; `specify integration install claude`
  adds it to an existing one. `specify mcp` starts an "experimental"
  MCP server. 43 integrations: 41 coding agents, plus `generic` and
  `none`.
- **What it puts into Claude Code.** "`.claude/skills/speckit-<command>/SKILL.md`",
  invoked as `/speckit-specify` and so on
  ([Claude integration](https://github.github.io/spec-kit/integrations/claude.html)).
  It also wires "native events" into `.claude/settings.json`, and those
  can be turned off. PR #2051 (merged 2026-04-02) "moves Claude Code
  support from generated slash-command files in `.claude/commands`" to
  native skills ([PR](https://github.com/github/spec-kit/pull/2051)).
- **Files it writes.** The constitution goes to
  `.specify/memory/constitution.md`
  ([command](https://raw.githubusercontent.com/github/spec-kit/main/templates/commands/constitution.md)).
  Each spec goes to `specs/<NNN or timestamp>-<short-name>/spec.md`,
  with at most three `[NEEDS CLARIFICATION]` markers
  ([command](https://raw.githubusercontent.com/github/spec-kit/main/templates/commands/specify.md)).
- **Workflow**
  ([reference](https://github.github.io/spec-kit/reference/agentic-sdd.html)):
  constitution → specify → *clarify* → plan → *checklist* → tasks →
  *analyze* → implement → converge. The three in italics are "optional
  quality gate[s]". `converge` will "Assess implementation against the
  artifacts and append remaining work". Extensions (`bug`, `assess`)
  add whole processes; presets "Adapt templates, commands, terminology,
  or methodology".
- **Its spec format**
  ([template](https://raw.githubusercontent.com/github/spec-kit/main/templates/spec-template.md)).
  "User Scenarios & Testing *(mandatory)*" holds blocks headed "User
  Story 1 - [Brief Title] (Priority: P1)", each with *Why this
  priority*, *Independent Test* and "**Given** … **When** … **Then**"
  scenarios. The other sections are Requirements (mandatory), Success
  Criteria (mandatory) and Assumptions. **It does not use "As a… I
  want…"**: the journey is described "in plain language".

### Tool, skill, command, plugin: Claude Code's own words

From the [Claude Code glossary](https://code.claude.com/docs/en/glossary),
**documented**:

| Term | Claude Code's definition |
| :- | :- |
| **Tool** | "An action Claude can take: read a file, edit code, run a shell command, search the web, spawn a subagent." |
| **Skill** | "A `SKILL.md` file containing instructions, knowledge, or a workflow … invoke it directly with `/skill-name`." Skills are "the recommended successor to custom commands". |
| **Command** | "A reusable instruction you invoke by typing `/name` … define your own commands as files in `.claude/commands/`." The glossary notes "Slash commands" is now just "Commands". |
| **Plugin** | "A bundle of skills, hooks, subagents, and MCP servers packaged as a single installable unit." |

So, in Claude Code's terms (**inference**, from the two sources above):

- spec-kit as a whole is an **external CLI plus templates**, not a tool.
- What it installs into Claude Code is a **set of skills** (before
  April 2026, a set of commands), plus event settings.
- Only `specify mcp` would add real tools, and that is experimental.
- It is not distributed as a Claude Code plugin: no plugin package was
  **found**.

## 3. Fit with Free Wings (all inference)

Where the two overlap:

| spec-kit | Free Wings |
| :- | :- |
| constitution (`.specify/memory/`), "principles every later phase is evaluated against" | `FOUNDATION.md`, the single source |
| `/speckit-clarify`, at most three open markers | `grilling`; a closed spec has no open questions |
| `/speckit-plan`, `/speckit-tasks` | `plan-<name>.md`, the spec's optional Tasks section |
| `/speckit-implement` | `programmer` / `engenir` |
| `/speckit-converge`, `/speckit-analyze` | `tester` |
| `specs/NNN-name/` (spec, plan, tasks, research, data-model…) | `docs/specs/NNNN-name.md`, one file |
| `.claude/skills/speckit-*` | `.claude/skills/` generated by `construct` (gitignored) |

`construct` "touch[es] only the outputs the adapter lists" (CONSTRUCT.md),
so both sets of skills can sit side by side on disk.

Three options:

- **A. Adopt spec-kit as-is.**
  - Gain: a maintained, widely used workflow across 41 agents.
  - Cost: two sources of truth (constitution and `FOUNDATION.md`); two
    competing chains (spec-kit's skills and Free Wings' agents);
    Free Wings' adapter does not generate spec-kit's files, so they
    would be hand-installed and could drift from Free Wings' own
    files; and specs split into several files per feature, which
    Böckeler found "repetitive" (see the 11-17 anatomy file).
  - Breaks ADR 0003's one-home rule unless one of the two is demoted.
- **B. Take only its template and format.** Copy the shape of its spec
  (prioritized, independently testable scenarios; Given/When/Then;
  FR-001 MUST statements; measurable success criteria; a cap on open
  markers) into Free Wings' own default template, then cite it.
  - Gain: one source of truth, no new runtime, MIT allows it.
  - Cost: the copy has to be refreshed by hand when spec-kit changes.
- **C. Take ideas only.** Borrow specific ideas (the cap on open
  questions, `converge` as a check after implementation, "independent
  test" per scenario) into Free Wings' own anatomy.
  - Gain: lowest coupling.
  - Cost: no shared vocabulary with spec-kit users.

B and C fit Free Wings' single-source design; A runs against it.
Choosing between them is the person's call, with `the-architect`. It
probably fires Foundation Sync, since it changes how the harness works.

## 4. Alternatives to user stories

How well an agent can check each one (the right-hand column) is
**inference**.

| Format | One example | Origin (documented) | Agile? | Can an agent check it pass/fail? |
| :- | :- | :- | :- | :- |
| **Job Story** (Jobs-to-be-Done) | "When an order is submitted, I want to see a warning message so I can avoid resubmitting the order." ([Mountain Goat](https://www.mountaingoatsoftware.com/blog/job-stories-offer-a-viable-alternative-to-user-stories)) | Intercom, Paul Adams, 2016-06-28: "[When] [I want to] [So I can]"; "Alan Klement later named it for us"; user stories are "engineering-, rather than customer-driven" ([Intercom](https://www.intercom.com/blog/accidentally-invented-job-stories/)). Klement's original post: **not found** (could not be fetched). | Agile-adjacent: product discovery, not an agile method | **Low on its own.** It states a motive, not a result. It needs examples or checks underneath. |
| **Specification by Example / BDD scenarios** | Given an order was submitted, When the user submits it again, Then a warning is shown and no second order is created. | Fowler, 2004: examples as a way "to describe one of roles of testing in XP" ([bliki](https://martinfowler.com/bliki/SpecificationByExample.html)). North, 2006: "Given some initial context … When an event occurs, Then ensure some outcomes." ([Introducing BDD](https://dannorth.net/blog/introducing-bdd/)). Adzic, Manning 2011: "seven key patterns", "living, reliable documentation" ([Manning](https://www.manning.com/books/specification-by-example)). | Yes: Extreme Programming, BDD | **High.** Each scenario becomes a test or a manual step. |
| **Example Mapping** (a workshop, not a format) | Cards: yellow = story, blue = rules, green = examples, red = open questions | Matt Wynne, Cucumber, 2015-12-08: "a simple, low-tech method"; red cards are "questions that nobody in the room can answer" ([Cucumber](https://cucumber.io/blog/bdd/example-mapping-introduction/)) | Yes: BDD | **High** once done: rules become requirements, examples become checks. No red cards left = Free Wings' "closed". |
| **EARS** | "When “mute” is selected, the laptop shall suppress all audio output." ([Mavin](https://alistairmavin.com/ears/)) | Rolls-Royce, IEEE RE'09 (see the 11-17 anatomy file); used at Airbus, NASA, Bosch and others; Kiro uses it | **No.** Requirements engineering; the page does not mention agile. | **Medium–high.** One trigger and one response per sentence; a check still has to be written for each. |
| **Use-Case 3.0** (use-case slices) | Basic flow plus alternative flows; delivered in "slices" | Jacobson, Spence, de Mendonça, *Use-Case 3.0 – The Definitive Guide*, May 2024; follows Use-Case 2.0 (2011). Slices are each "one or more paths through a use case" with "design, code, and tests". **Partly documented:** only through a search summary of the official PDF ([PDF](https://www.ivarjacobson.com/files/use-case_3.0_v1.0.pdf)); the PDF was unreadable here and the site returned 403. | Presented as agile and scalable (search summary only) | **Medium.** Flows are checkable; each slice carries its own tests. |

A combination that answers the person's wishes (**inference**):

- an optional one-line Job Story as the "why";
- EARS for each requirement;
- Given/When/Then examples as the checks;
- Example Mapping's red cards as the open questions, which must be
  empty before a spec is closed.

That is close to the default anatomy proposed in the 11-17 file, with
"Goal" written as a Job Story. spec-kit's own template already drops
"As a…" while keeping Given/When/Then, so the combination lines up
with spec-kit in practice (B or C above).

## Confidence

- **Confirmed:**
  - The Scrum Guide has no user stories and no item format.
  - spec-kit's CLI, its skills location for Claude Code, workflow,
    licence and releases.
  - Its spec template does not use "As a".
  - The Claude Code glossary terms.
  - Origins of Job Stories (Intercom), Specification by Example, BDD,
    Example Mapping and EARS.
- **Likely:** the details of Use-Case 3.0 (secondary summary only).
- **Not found:**
  - Klement's original post text.
  - A spec-kit Claude Code plugin package.
  - Any agile source for EARS.
- **Inference:** the column on whether an agent can check each format,
  the comparison of options A, B and C, and the suggested combination.

## Sources

- [Scrum Guide 2020](https://scrumguides.org/scrum-guide.html)
- [spec-kit repo](https://github.com/github/spec-kit) · [docs](https://github.github.io/spec-kit/) · [Claude integration](https://github.github.io/spec-kit/integrations/claude.html) · [SDD reference](https://github.github.io/spec-kit/reference/agentic-sdd.html) · [PR #2051](https://github.com/github/spec-kit/pull/2051) · [releases](https://github.com/github/spec-kit/releases) · [spec template](https://raw.githubusercontent.com/github/spec-kit/main/templates/spec-template.md) · [constitution command](https://raw.githubusercontent.com/github/spec-kit/main/templates/commands/constitution.md) · [specify command](https://raw.githubusercontent.com/github/spec-kit/main/templates/commands/specify.md)
- [Claude Code glossary](https://code.claude.com/docs/en/glossary)
- [Intercom, Job Stories](https://www.intercom.com/blog/accidentally-invented-job-stories/) · [Mountain Goat Software](https://www.mountaingoatsoftware.com/blog/job-stories-offer-a-viable-alternative-to-user-stories)
- [Fowler, SpecificationByExample](https://martinfowler.com/bliki/SpecificationByExample.html) · [North, Introducing BDD](https://dannorth.net/blog/introducing-bdd/) · [Manning, Specification by Example](https://www.manning.com/books/specification-by-example)
- [Wynne, Example Mapping](https://cucumber.io/blog/bdd/example-mapping-introduction/)
- [Mavin, EARS](https://alistairmavin.com/ears/)
- [Use-Case 3.0 PDF](https://www.ivarjacobson.com/files/use-case_3.0_v1.0.pdf)
