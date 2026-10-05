# OpenCode Adapter

<!--
Lives in `hangar/blueprints/adapters/`. Defines the adapter for OpenCode — the description of how `construct` adapts the harness's tool-agnostic blueprints to this specific AI coding tool. Read as a rule during compilation; not compiled into an output itself, and not a skill (a skill is invoked directly by the user or an agent — an adapter is read by `construct` in the middle of its own execution). One adapter per supported tool.
-->

## What this adapter does

Tells `construct` exactly how to translate the harness's tool-agnostic
blueprints (`FOUNDATION.md`, `agents/*.md`, `skills/*.md`) into the
structure OpenCode expects to find on disk. Read during Step 3 of the
construct procedure, in place of any hardcoded per-tool logic.

## Outputs

| Blueprint source | OpenCode destination |
| :--- | :--- |
| `FOUNDATION.md` (project root) | `AGENTS.md` (project root) |
| `hangar/blueprints/agents/<name>.md` | `.opencode/agents/<name>.md` |
| `hangar/blueprints/skills/<name>.md` | `.opencode/skills/<name>/SKILL.md` |

Note the shape transformation: agent blueprints stay one file each;
skill blueprints become a folder-per-skill with `SKILL.md` inside. The
blueprint is flat; OpenCode's skill format is not.

## Format — `AGENTS.md` (project root)

A compiled **excerpt** of `FOUNDATION.md`, optimized for OpenCode's
reading context. Markdown, no YAML frontmatter.

OpenCode reads `AGENTS.md` at the project root as its primary context
file. Prioritize: philosophy, structure, agent and skill lists,
conventions, and current status. Omit: lengthy historical reasoning
that does not inform a session. Not a literal copy of the Foundation.

## Format — agent files

`.opencode/agents/<name>.md` uses YAML frontmatter with the following
fields. **All field names and value formats below are verified against
`opencode.ai/docs/agents` (consulted 2026-09-15).**

```yaml
---
description: <one or two sentences — what this agent is for>
mode: <primary | subagent | all>
tools:
  <tool_name>: <true | false>
  ...
---
```

Then the agent body — the blueprint's content, minus the HTML
orientation comment.

### Field details

**`description`** (required) — one or two sentences describing what the
agent does. Take from the blueprint's `## Role` section: the first
sentence or the "Use this agent to" line.

**`mode`** — one of `primary`, `subagent`, or `all`. Meaning, per the
official docs:

- `primary` — the agent appears in the **Tab cycle**, alongside Build
  and Plan. It is a mode of the main conversation.
- `subagent` — the agent is only reachable via **`@mention`** (e.g.,
  `@programmer`) or by delegation from a primary agent. It does **not**
  appear in the Tab cycle.
- `all` — the agent is reachable **both ways**: in the Tab cycle and
  via `@mention`. This is the **default** when the field is omitted.

**Mapping for the Free Wings agents — do not guess, use this table
verbatim:**

| Agent | `mode` | Why |
| :--- | :--- | :--- |
| `the-architect` | `primary` | Entry point of the harness. The first agent the person reaches for when they don't know which agent fits. Belongs in the Tab cycle as the starting mode. |
| `programmer` | `all` | Invoked directly as the main task, or delegated to by `the-architect` / `researcher`. Both ways are normal. |
| `tester` | `all` | Invoked directly after `programmer` just built something, or delegated to by `programmer`. Both ways are normal. |
| `deneir` | `all` | Invoked directly to watch a project, or delegated to by `writer`. Both ways are normal. |
| `writer` | `all` | Invoked directly to shape material, or delegated to by `deneir`. Both ways are normal. |
| `researcher` | `all` | Invoked directly to verify a claim, or delegated to by `programmer` / `the-architect`. Both ways are normal. |

**Take `mode` only from the table above** — never infer it from the
wording of a blueprint or of `FOUNDATION.md`. In OpenCode, `subagent`
is a `mode` value: an agent reachable only via `@mention` or by
delegation from a primary agent. The harness itself has no
"subagent" concept; it only has agents.

**`tools`** — the per-tool enable/disable map. **Format: an object
(record) with tool names as keys and booleans as values.** Never a
string, never a comma-separated list — OpenCode rejects that format
with a hard crash at startup
(`InvalidError: expected record, received string`). The OpenCode schema
is `z.record(z.string(), z.boolean())`.

**Mapping for the Free Wings agents — do not guess, use this table
verbatim:**

| Agent | `read` | `grep` | `glob` | `write` | `edit` | `bash` | Rationale (from the blueprint's `## Write permissions`) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `programmer` | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | Broad read/write/execute — no restriction in the blueprint. |
| `tester` | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | Creates test files **and** runs them — needs `write`, `edit`, `bash`. |
| `deneir` | ✓ | ✓ | ✓ | ✓ | ✗ | ✗ | Read-only about the project; writes only to `docs/observations/`. Needs `write` (new dated files), not `edit` (never modifies existing files), not `bash`. |
| `researcher` | ✓ | ✓ | ✓ | ✓ | ✗ | ✗ | Reads, uses web search/fetch (see below), writes only to `docs/research/`. Needs `write`, not `edit`, not `bash`. |
| `writer` | ✓ | ✓ | ✓ | ✓ | ✓ | ✗ | Shapes material into diary entries / articles. Needs `edit` (revising drafts), not `bash`. |
| `the-architect` | ✓ | ✓ | ✓ | ✗ | ✗ | ✗ | Read-only, never writes. |

For `researcher`, if the target OpenCode version supports `webfetch`
and/or `websearch` as named tools, add them as `true`. If the exact
name is uncertain, **omit those two keys** (the agent will simply lack
the tool, and can note that in its output) rather than guess at a
spelling that might fail validation.

**Important note on deprecation:** OpenCode's documentation states that
the `tools` field is deprecated in favor of `permission`, which uses
`"ask"` / `"allow"` / `"deny"` strings. However, `tools` with boolean
values continues to be accepted and validated. Use `tools` with
booleans, per the mapping above. When OpenCode fully removes `tools`,
update **this adapter file** to emit `permission` instead — no change
to `CONSTRUCT.md`, `FOUNDATION.md`, or any blueprint.

## Format — skill files

`.opencode/skills/<name>/SKILL.md` — folder-per-skill. YAML
frontmatter:

```yaml
---
name: <kebab-case identifier>
description: <one or two sentences>
---
```

Then the skill body. **Skill files do not accept a `tools` field.** The
OpenCode schema does not validate it the way agents do; skills are
loaded on-demand and their tool access is determined by the invoking
agent's permissions. Do not include a `tools` field in skill files.

## Invocation

OpenCode invokes agents by name — via `@<name>` for `subagent` / `all`
agents, and via the Tab cycle for `primary` / `all` agents. Skills are
invoked via their name. The file name (`<name>`, kebab-case) is the
identifier in all cases, so the mapping from blueprint name to
invocation identifier is direct.

## Gitignore

The generated `.opencode/` directory should be gitignored — it is a
regenerable output, not source. Same reasoning as never committing a
`build/` folder. If the target project has already established a
practice of committing tool-specific files, respect that — see the
`FOUNDATION.md` discussion of this choice.

## Known limits

OpenCode's exact field set and validation rules have changed recently —
the `tools` → `permission` transition is ongoing, and the precise
boolean map for `tools` may evolve. This adapter reflects the structure
current as of **2026-09-15**, verified against `opencode.ai/docs/agents`
and `opencode.ai/docs/pt-br/agents`. When OpenCode's conventions change,
update *this adapter file* — not `CONSTRUCT.md`, and not
`FOUNDATION.md`.