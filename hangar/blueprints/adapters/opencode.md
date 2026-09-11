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

Same shape transformation as Claude Code: agent blueprints stay one
file each; skill blueprints become a folder-per-skill with `SKILL.md`
inside.

## Format — `AGENTS.md` (project root)

A compiled **excerpt** of `FOUNDATION.md`, optimized for OpenCode's
reading context. Markdown, no YAML frontmatter.

OpenCode reads `AGENTS.md` at the project root as the primary context
file — the role Claude Code gives `CLAUDE.md`. Same priorities as
described in the Claude adapter: philosophy, structure, agent and
skill lists, conventions, current status. Not a literal copy of the
Foundation.

## Format — agent files

`.opencode/agents/<name>.md` uses YAML frontmatter with the same
fields Claude Code uses:

```yaml
---
name: <kebab-case identifier>
description: <one or two sentences>
tools: <comma-separated list of tools this agent may use>
---
```

Then the agent body — the blueprint's content, minus the HTML
orientation comment.

**Deriving `tools:` from the blueprint.** Same logic as the Claude
adapter, restated here so this file stands on its own: the blueprint's
`## Write permissions` section (when present) tells you what the agent
may write to; combined with the described read behavior, that informs
the list. Agents without that section have broader defaults. If
unclear for a specific agent, ask the user rather than guessing.

## Format — skill files

`.opencode/skills/<name>/SKILL.md` — folder-per-skill. YAML
frontmatter:

```yaml
---
name: <kebab-case identifier>
description: <one or two sentences>
---
```

Then the skill body.

## Invocation

OpenCode invokes agents by name (with the tool's own prefix convention)
and skills by name. The file name (`<name>`, kebab-case) is the
identifier in both cases, so the mapping from blueprint name to
invocation identifier is direct.

## Gitignore

The generated `.opencode/` directory should be gitignored — it is a
regenerable output, not source. Same reasoning as described in the
Claude adapter.

## Known limits

OpenCode's conventions have been observed to track Claude Code's
closely in the areas that matter here, but the exact field set and any
OpenCode-specific quirks should be verified against OpenCode's current
documentation before being relied on as definitive. This adapter
reflects the structure current as of 2026-09-11. When OpenCode's
conventions change, update *this adapter file* — not `CONSTRUCT.md`,
and not `FOUNDATION.md`.