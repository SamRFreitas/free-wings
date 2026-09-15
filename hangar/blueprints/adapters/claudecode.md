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

## Format — agent files

`.claude/agents/<name>.md` uses YAML frontmatter. Only `name` and
`description` are required; all other fields are optional. **All field
names, formats, and semantics below are verified against
`code.claude.com/docs/en/sub-agents` (consulted 2026-09-15).**

```yaml
---
name: <kebab-case identifier>
description: <one or two sentences — what this agent is for>
tools: Read, Grep, Glob
---
```

Then the agent body — the blueprint's content, minus the HTML
orientation comment (that comment is harness-internal and does not
travel into the compiled output).

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

**Deriving `tools:` from the blueprint.** The blueprint's
`## Write permissions` section, when present, tells you what the agent
may write to; combined with the agent's described read behavior, that
informs the list. Mapping for the Free Wings agents — do not guess, use
this table verbatim:

| Agent | `tools` string | Rationale |
| :--- | :--- | :--- |
| `programmer` | *(omit — inherits all)* | Broad read/write/execute — no restriction in the blueprint. |
| `tester` | *(omit — inherits all)* | Creates test files and runs them. |
| `deneir` | `Read, Grep, Glob, Bash, Write` | Read-only about the project; only writes to `docs/observations/`. The behavioral constraint is enforced by the blueprint, not the tool list (Claude Code's `tools` is boolean per tool, not per path). |
| `researcher` | `Read, Grep, Glob, WebSearch, WebFetch, Write` | Reads, uses web search/fetch, writes only to `docs/research/`. |
| `writer` | `Read, Grep, Glob, Write, Edit` | Shapes material into diary entries / articles; edits drafts. |
| `the-architect` | `Read, Grep, Glob` | Read-only; never writes. |

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

**`model`** (optional) — the model the subagent uses. Accepted values
include `sonnet`, `opus`, `haiku`, and `inherit`. `inherit` means "use
the same model as the parent conversation". The harness does **not**
specify a model — omit this field and let Claude Code decide, unless
the person asks for a specific one.

**`disallowedTools`** (optional) — the inverse of `tools`: a list of
tools to **remove** from the inherited set. Same format as `tools`
(comma-separated string or YAML array). Useful when an agent needs
almost everything but one specific tool. The harness does not use this
by default.

**`permissionMode`** (optional) — controls the subagent's permission
behavior. Accepted values include `default`, `acceptEdits`,
`bypassPermissions`, and `plan`. Omit by default — Claude Code's
`default` is what the harness expects.

**`skills`** (optional) — a list of skill names to preload into the
subagent's context. Each listed skill is loaded as if invoked by the
subagent itself. The harness does not use this by default — skills are
invoked on-demand, not preloaded.

**`Agent(<agent_type>)` inside `tools`** — a special entry in the
`tools` list that lets a subagent spawn **specific other subagents**,
restricted to the named type. Example: `tools: Read, Grep,
Agent(researcher)` allows this agent to spawn only the `researcher`
subagent. Without this entry, if a subagent is allowed to spawn others,
it can spawn any registered subagent.

**Note on `the-architect` and subagent spawning.** The blueprint for
`the-architect` carries a "Known open question, stated honestly"
section: whether this agent can directly invoke another agent (chaining
subagents) or can only *recommend* one is not confirmed. Until that is
resolved, do **not** add `Agent(...)` entries to its `tools` — the
harness assumes the safer case (recommend, don't chain). If the person
later confirms that direct chaining is supported and desired, update
**this adapter file** to include the mapping — not the blueprint, not
`CONSTRUCT.md`.

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
available, they just may require user approval.

The harness's skills (`write-diary`, `write-article`, `loop-status`)
are not gated by permissions in a way that requires this field. Omit it
unless the blueprint explicitly needs a specific tool allowlist.

## Invocation

Claude Code invokes agents via `@<name>` and skills via `/<name>`. Both
use the file name (`<name>`, kebab-case) as the invocation identifier,
so the mapping from blueprint name to invocation identifier is direct —
no translation step. A subagent may also be run as the **main thread**
of a conversation via the CLI flag `--agent <name>` — in that case the
subagent *is* the primary conversation, which is a decision made at
invocation time, not a field in the file. Claude Code has no `mode`
field equivalent to OpenCode's `primary` / `subagent` / `all`.

## Gitignore

The generated `.claude/` directory should be gitignored — it is a
regenerable output, not source. Same reasoning as never committing a
`build/` folder. If the target project has already established a
practice of committing tool-specific files (an existing project may
reasonably do this), respect that — see the `FOUNDATION.md` discussion
of this choice.

## Known limits

Claude Code's exact field set and validation rules may evolve. This
adapter reflects the structure current as of **2026-09-15**, verified
against `code.claude.com/docs/en/sub-agents` and
`code.claude.com/docs/en/skills`. The following were verified:

- `name`, `description` — required fields, formats as documented.
- `tools` — accepts comma-separated string and YAML array; omitting
  inherits all; unresolvable entries cause launch failure.
- `model`, `disallowedTools`, `permissionMode`, `skills`,
  `Agent(<agent_type>)` — exist as optional fields; exact accepted
  values for `model` and `permissionMode` should be re-verified against
  the current Claude Code docs before relying on a specific value.
- `allowed-tools` in skills — accepts comma-separated string, space-
  separated string, or YAML list; grants permission without prompting,
  does not restrict.

When Claude Code's conventions change, update *this adapter file* — not
`CONSTRUCT.md`, and not `FOUNDATION.md`.