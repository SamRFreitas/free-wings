# Claude Code Adapter

<!--
Lives in `hangar/blueprints/adapters/`. Defines the adapter for Claude Code — the description of how `construct` adapts the harness's tool-agnostic blueprints to this specific AI coding tool. Read as a rule during compilation; not compiled into an output itself, and not a skill (a skill is invoked directly by the user or an agent — an adapter is read by `construct` in the middle of its own execution). One adapter per supported tool.
-->

## What this adapter does

Tells `construct` exactly how to translate the harness's tool-agnostic
blueprints (`FOUNDATION.md`, `agents/*.md`, `skills/*.md`) into the
structure Claude Code expects to find on disk. Read during Step 3 of the
construct procedure, in place of any hardcoded per-tool logic.

## Outputs

| Blueprint source | Claude Code destination |
| :--- | :--- |
| `FOUNDATION.md` (project root) | `CLAUDE.md` (project root) |
| `hangar/blueprints/agents/<name>.md` | `.claude/agents/<name>.md` |
| `hangar/blueprints/skills/<name>.md` | `.claude/skills/<name>/SKILL.md` |

Note the shape transformation: agent blueprints stay one file per
agent; skill blueprints become a folder-per-skill with `SKILL.md`
inside. The blueprint is flat; Claude Code's skill format is not.
Converting a flat blueprint into that nested shape is part of what
this adapter describes.

## Format — `CLAUDE.md` (project root)

A compiled **excerpt** of `FOUNDATION.md`, optimized for Claude Code's
reading context at session start. Markdown, no YAML frontmatter.

Not a literal copy — `FOUNDATION.md` is dense and complete by design;
`CLAUDE.md` is what Claude Code actually benefits from as persistent
context. Prioritize: philosophy, structure, agent and skill lists,
conventions, and current status. Omit: lengthy historical reasoning
that does not inform a session.

## Format — subagent files

In Claude Code, a file in `.claude/agents/` defines a **subagent**: a
specialized assistant Claude can delegate a task to, which runs in its
own separate context and returns a summary to the conversation that
called it. Each harness agent blueprint compiles into one subagent
file. The same file can also run as the **main thread** of a session
(see "Invocation" below).

`.claude/agents/<name>.md` uses YAML frontmatter. Only `name` and
`description` are required; all other fields are optional. **All field
names, formats, and semantics below are verified against
`code.claude.com/docs/en/sub-agents` (consulted 2026-10-02).**

```yaml
---
name: <kebab-case identifier>
description: <one or two sentences — what this agent is for>
tools: Read, Grep, Glob
---
```

Then the agent body — the blueprint's content, minus the HTML
orientation comment (harness-internal, it does not travel into the
compiled output) — followed by the content of
`hangar/blueprints/shared/agent-rules.md`, minus its title and its
orientation comment.

### Field details

**`name`** (required) — unique identifier, lowercase letters and
hyphens. Kebab-case. Must not contain `:` (reserved for plugin-scoped
identifiers). The file name doesn't have to match, but keeping them
equal is the harness's convention.

**`description`** (required) — when Claude should delegate to this
subagent. Derive from the blueprint's `## Role` section: the first
sentence plus the "Use this agent to" line.

**`tools`** (optional) — the tools the subagent is allowed to use.
**Format: comma-separated string with a space after each comma** — e.g.,
`tools: Read, Grep, Glob`. A **YAML array** (`tools: [Read, Grep,
Glob]`) is also accepted. **If omitted, the subagent inherits every
tool available to subagents.**

**`tools` for the Free Wings agents** — do not guess, use this table
verbatim. Where an agent may write is the blueprint's rule ("Where this
agent writes"), not this list: `tools` is boolean per tool, not per
path or command.

| Agent | `tools` string | Why |
| :--- | :--- | :--- |
| `programmer` | *(omit — inherits all)* | No restriction. |
| `tester` | *(omit — inherits all)* | Creates test files and runs them. |
| `deneir` | `Read, Grep, Glob, Bash, Write` | `Bash` to read git history, `Write` for new files. |
| `researcher` | `Read, Grep, Glob, WebSearch, WebFetch, Write` | Web search and fetch, `Write` for new files. |
| `writer` | `Read, Grep, Glob, Write, Edit` | `Edit` to revise drafts. |
| `the-architect` | *(omit — inherits all)* | Works as a normal session. Inheriting includes `Agent`, so it can delegate. |

**If the mapping is unclear for a specific agent, ask the user rather
than guessing.** **Safety rule:** an entry in `tools` that does not
resolve to a real Claude Code tool **causes the subagent to fail at
launch**. When in doubt about whether a tool name is valid, omit the
field — inheritance is always valid.

### Optional advanced fields

The following fields exist in Claude Code but are **not required by the
harness by default**. Document them here so that a future need — a
specific model for a specific agent, a tool explicitly denied, a
restricted subagent-spawning list — can be expressed without leaving
this adapter.

**`model`** (optional) — the model the subagent uses. Accepted values:
`sonnet`, `opus`, `haiku`, `fable`, `inherit`, or a full model ID
(e.g. `claude-opus-5-5`). `inherit` means "use
the same model as the parent conversation". The harness does **not**
specify a model — omit this field and let Claude Code decide, unless
the person asks for a specific one.

**`disallowedTools`** (optional) — the inverse of `tools`: a list of
tools to **remove** from the inherited set. Same format as `tools`
(comma-separated string or YAML array). Useful when an agent needs
almost everything but one specific tool. The harness does not use this
by default.

**`permissionMode`** (optional) — controls the subagent's permission
behavior. Accepted values: `default`, `acceptEdits`, `auto`,
`dontAsk`, `bypassPermissions`, `plan`, or `manual` (an alias for
`default`). Omit by default — Claude Code's
`default` is what the harness expects.

**`skills`** (optional) — a list of skill names to preload into the
subagent's context. Each listed skill is loaded as if invoked by the
subagent itself. The harness does not use this by default — skills are
invoked on-demand, not preloaded.

**`Agent` and `Agent(<agent_type>, ...)` inside `tools`** — the entry
that controls whether a subagent can spawn other subagents. Three
cases:

- `tools` **omitted** — the subagent inherits every tool, `Agent`
  included, so it can spawn any subagent.
- `tools` **listed with `Agent(a, b)`** — it can spawn only the named
  subagent types. Example: `tools: Read, Grep, Agent(researcher)`.
- `tools` **listed without `Agent`** — it cannot spawn any subagent.

By default, nesting goes up to three layers below the main
conversation; `CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH` in `settings.json`
lowers it (`1` disables nesting).

**Note on `the-architect` and delegation.** The blueprint lets
`the-architect` delegate to `researcher`, `deneir`, and `writer`. In
Claude Code that takes the `Agent` tool, which it has because its
`tools` field is omitted. `programmer` and `tester` are not delegated
to: they are opened as their own sessions (see "Which way to open each
agent").

## Format — skill files

`.claude/skills/<name>/SKILL.md` — folder-per-skill, file named
`SKILL.md`. YAML frontmatter:

```yaml
---
name: <kebab-case identifier>
description: <one or two sentences>
---
```

Then the skill body.

### Field details

**`name`** (required) — display name and `/slash-command` identifier.

**`description`** (required) — one or two sentences.

**`allowed-tools`** (optional) — tools Claude can use **without asking
permission** when this skill is active. **Note the hyphen — it is
`allowed-tools`, not `allowed_tools`.** This is the skill-level
equivalent of the agent's `tools` field, and it is deliberately named
differently. Format: comma-separated string, space-separated string, or
YAML list — all three are accepted. Example: `allowed-tools: Read,
Grep, Glob`.

**Important semantic difference from the agent `tools` field:** for
agents, `tools` is an **allowlist** (only those tools are available).
For skills, `allowed-tools` **grants permission without prompting** —
it does not restrict the skill to only those tools. Other tools remain
available, they just may require user approval. The grant lasts only
for the turn that invokes the skill; it clears with the next message.

The harness's skills (`write-diary`, `write-article`, `loop-status`)
are not gated by permissions in a way that requires this field. Omit it
unless the blueprint explicitly needs a specific tool allowlist.

## Invocation

A subagent is invoked explicitly with an @-mention: type `@` and pick
it from the typeahead (it appears as `@"<name> (agent)"`), or type
`@agent-<name>` by hand. Claude can also delegate to it on its own,
based on its `description`. A skill is invoked as `/<name>`. Both use
the blueprint's kebab-case name, so no translation step is needed.

A subagent can also run as the **main thread** of a session, via the
CLI flag `--agent <name>` or `"agent": "<name>"` in
`.claude/settings.json`. Its prompt then replaces the default system
prompt, and the main thread takes on its `tools` and `model`. This is
chosen when the session starts, not by a field in the subagent file.

### Which way to open each agent

The two ways behave differently. As a **subagent** (`@agent-<name>`),
the agent works alone and returns one report to the session that called
it; that session rewrites the request on the way in and summarizes the
answer on the way out, and the agent cannot ask the person anything —
Claude Code removes `AskUserQuestion` from every subagent. As the
**main thread** (`claude --agent <name>`), the person talks to the
agent directly, turn by turn, with nothing in between.

So an agent that explains, asks, and waits belongs in the main thread;
an agent that takes a request, produces a file, and finishes works well
as a subagent:

| Agent | Open it with | Why |
| :--- | :--- | :--- |
| `the-architect` | `claude --agent the-architect` | The entry point, and the one the person plans with. |
| `programmer` | `claude --agent programmer` | Teaches and asks while it implements. |
| `tester` | `claude --agent tester` by default; `@agent-tester` only to judge an output that already exists | Its first pass explains and asks, which needs the main thread. A pass that only judges a finished output needs no dialogue. |
| `researcher` | `@agent-researcher`, or delegated by `the-architect` | Takes a request, writes a file in `docs/research/`, finishes. |
| `deneir` | `@agent-deneir`, or delegated by `the-architect` | Takes a request, writes a file in `docs/observations/`, finishes. |
| `writer` | `@agent-writer`, or delegated by `the-architect` | Takes a request, writes a draft, finishes. |

**Sessions do not share a conversation.** What passes from one session
to the next is the message the person copies across, or whatever is
already written in the repository. This is why `the-architect` hands
over a ready-to-paste message with every recommendation.

### Making a session behave as an agent by default

Two documented ways, both chosen by the project, neither generated by
`construct`:

- **Per session**: `claude --agent <name>`.
- **Project default**: `"agent": "<name>"` in `.claude/settings.json`.
  Every session in that project then starts as that agent; `--agent`
  overrides it for one session.

How to open a plain, default session in a project that has `"agent"`
set is **not documented** as of 2026-10-05: the docs say only that the
key is unset by default and that `--agent` overrides it. Do not assume
a built-in agent name that restores the default — test it, or leave
`"agent"` unset and use `--agent` per session.

`construct` writes only the files this adapter lists under "Outputs".
It never deletes or changes anything else in `.claude/` — a project's
own `settings.json`, `settings.local.json`, or extra agents and skills
are left exactly as they are.

## Gitignore

The generated `.claude/` directory should be gitignored — it is a
regenerable output, not source. Same reasoning as never committing a
`build/` folder. If the target project has already established a
practice of committing tool-specific files (an existing project may
reasonably do this), respect that — see the `FOUNDATION.md` discussion
of this choice.

## Known limits

Claude Code's exact field set and validation rules may evolve. This
adapter reflects the structure current as of **2026-10-05** (Claude
Code 2.1.289), verified against `code.claude.com/docs/en/sub-agents`,
`code.claude.com/docs/en/skills`, `code.claude.com/docs/en/permissions`,
`code.claude.com/docs/en/settings-reference`, and `claude --help`. The
following were verified:

- `name`, `description` — required fields, formats as documented.
- `tools` — accepts comma-separated string and YAML array; omitting
  inherits all; unresolvable entries cause launch failure.
- `model`, `permissionMode` — accepted values as listed above.
- `disallowedTools`, `skills`, `Agent(<agent_type>)` — exist as
  optional fields; spawn depth defaults to three layers.
- @-mention syntax and `--agent` / `"agent"` main-thread behavior —
  as described in "Invocation".
- `allowed-tools` in skills — accepts comma-separated string, space-
  separated string, or YAML list; grants permission without prompting
  for the invoking turn only, does not restrict.
- `AskUserQuestion` — removed from every subagent, even when listed in
  `tools`.
- Read-only forms of `git` — run without a permission prompt in every
  mode; permission rules are per session, not per subagent.

**Plan mode.** A Claude Code permission mode (`plan`): Claude reads and
explores but does not edit files, then presents one plan for one
approval. It is not the mode that asks file by file — that is the
default (manual) mode.

- **How it meets the blueprints' "show first, then write" rule**: an
  approved plan that showed the exact path and text of a change is the
  approval for exactly that change. Anything the plan only described is
  still shown before it is written.
- **Starting every session in a mode** is a project choice:
  `defaultMode` in the project's settings file. `construct` does not
  generate it.
- **Delegating from a plan-mode session**: observed on 2026-10-05, a
  `researcher` called this way could not save its file. Per its
  blueprint it then returns the full content and path. The docs say a
  subagent runs in the `permissionMode` set in its own file when the
  main conversation is in plan mode; the harness sets none. Whether
  setting one changes this is not tested.
- **Not yet tested**: `the-architect` running as the main thread in
  plan mode with inherited tools. The earlier limit (no tool to write
  the plan or ask questions) was observed with a restricted `tools`
  list.

When Claude Code's conventions change, update *this adapter file* — not
`CONSTRUCT.md`, and not `FOUNDATION.md`.