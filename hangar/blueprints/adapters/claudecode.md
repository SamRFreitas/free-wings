# Claude Code Adapter

<!--
Lives in `hangar/blueprints/adapters/`. Defines the adapter for Claude Code — the description of how `construct` adapts the harness's tool-agnostic blueprints to this specific AI coding tool. Read as a rule during compilation; not compiled into an output itself, and not a skill (a skill is invoked directly by the user or an agent — an adapter is read by `construct` in the middle of its own execution). One adapter per supported tool.
-->

## What this adapter does

Tells `construct` exactly how to translate the harness's tool-agnostic
blueprints (`FOUNDATION.md`, `agents/*.md`, `skills/*.md`) into the
structure Claude Code expects to find on disk. Read during Step 3 of
the construct procedure, in place of any hardcoded per-tool logic.

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

`.claude/agents/<name>.md` requires YAML frontmatter:

```yaml
---
name: <kebab-case identifier>
description: <one or two sentences — what this agent is for>
tools: <comma-separated list of tools this agent may use>
---
```

Then the agent body — the blueprint's content, minus the HTML
orientation comment (that comment is harness-internal and does not
travel into the compiled output).

**Deriving `tools:` from the blueprint.** The blueprint's
`## Write permissions` section, when present, tells you what the agent
may write to; combined with the agent's described read behavior, that
informs the list. Examples:

- `deneir` — reads project state, writes only to `docs/observations/`
- `researcher` — reads project state, uses web search/fetch, writes
  only to `docs/research/`
- `programmer` — broad read/write/execute (no `## Write permissions`
  section is present because the write access is unconstrained)
- `the-architect` — read only (never writes)

If the mapping is unclear for a specific agent, ask the user rather
than guessing.

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

## Invocation

Claude Code invokes agents via `@<name>` and skills via `/<name>`. Both
use the file name (`<name>`, kebab-case) as the invocation identifier,
so the mapping from blueprint name to invocation identifier is direct —
no translation step.

## Gitignore

The generated `.claude/` directory should be gitignored — it is a
regenerable output, not source. Same reasoning as never committing a
`build/` folder. If the target project has already established a
practice of committing tool-specific files (an existing project may
reasonably do this), respect that — see the `FOUNDATION.md` discussion
of this choice.

## Known limits

Claude Code's exact field set and validation rules may evolve. This
adapter reflects the structure current as of 2026-09-11. When Claude
Code's conventions change, update *this adapter file* — not
`CONSTRUCT.md`, and not `FOUNDATION.md`.