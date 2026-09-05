---
name: construct
description: Generate or update a project's CLAUDE.md and AGENTS.md from its FOUNDATION.md — the tool-agnostic source of truth. Use when a project's FOUNDATION.md was just written or changed, or when CLAUDE.md/AGENTS.md need to be brought back in sync with it.
---

# construct

Reads a project's `FOUNDATION.md` and (re)generates the tool-specific
files derived from it — currently `CLAUDE.md` (for Claude Code) and
`AGENTS.md` (for OpenCode and similar tools). Named after the
"construct" in *Neuromancer*: a stored recording of someone's
skills/knowledge, loaded up when needed to help with a task — this
skill does the same thing with a project's own accumulated knowledge
(its `FOUNDATION.md`).

## When to use this

- A project's `FOUNDATION.md` was just written or edited, and
  `CLAUDE.md`/`AGENTS.md` need to reflect that.
- `CLAUDE.md`/`AGENTS.md` were hand-edited directly at some point and
  may have drifted from `FOUNDATION.md` — this brings them back in sync,
  with `FOUNDATION.md` always winning as the source of truth.
- A brand new project wants to adopt this harness's pattern for the
  first time.

## Procedure

1. **Find `FOUNDATION.md`.** Default to the current working directory;
   if the user gave a path as an argument, use that instead. If no
   `FOUNDATION.md` exists there, stop — do not try to generate one from
   nothing. A `FOUNDATION.md` is meant to be written through real
   dialogue between the person and the AI (the same dialogic principle
   `FOUNDATION.md` itself describes under "Pedagogical approach") —
   auto-generating one from guesses would violate the exact principle
   this harness is built on. Instead, offer to help write one through
   conversation, the way Shadow Glass's and this harness's own
   `FOUNDATION.md`/`CLAUDE.md` were written.

2. **Read `FOUNDATION.md` fully.** This is the only source of truth for
   what goes into the generated files — never invent content that isn't
   grounded in it.

3. **Confirm the target tools**, don't silently guess. Ask the person
   which AI tools this project's generated files need to serve. Default
   assumption if not specified: both **Claude Code** (`CLAUDE.md`) and
   **OpenCode** (`AGENTS.md`), since those are this harness's two
   established targets. If the person names a different tool with its
   own expected filename, generate that instead/also.

4. **Generate each file, compressed differently per reader**:
   - `CLAUDE.md`: written for a strong, capable model. Can be denser,
     can assume the reader fills in obvious inferences, doesn't need to
     spell out every implication.
   - `AGENTS.md`: written for a model that may be meaningfully weaker.
     Same content, same meaning, but more explicit — spell out
     implications the `CLAUDE.md` version leaves implicit, avoid
     compressed/idiomatic phrasing, prefer complete sentences over
     terse notes.
   - Both must be faithful to `FOUNDATION.md` — no content should exist
     in a generated file that isn't grounded in the Foundation, and
     nothing load-bearing from the Foundation should be silently
     dropped.

5. **If `CLAUDE.md`/`AGENTS.md` already exist**, treat this as a
   regeneration: show the person what's changing (a diff, or a clear
   summary of what's added/removed/reworded) before overwriting —
   `FOUNDATION.md` wins on conflict, but the person should see the
   change, not have it happen silently underneath them.

6. **Never commit automatically.** Generate the files, report what
   changed and why, and leave the actual `git add`/`git commit` to the
   person (or to an explicit later request) — same standing rule every
   project under this harness follows for git operations.

## Known limitations (as of this writing)

- "Detecting" which AI tool a project uses by scanning its files (e.g.
  presence of a `.claude/` folder) was considered and rejected in favor
  of asking directly — file-sniffing is guessable-wrong in a way a
  direct question isn't, and this harness prefers asking over guessing
  wherever the cost of asking is low.
- This skill only handles `FOUNDATION.md` → `CLAUDE.md`/`AGENTS.md`
  generation. It does not (yet) set up a project to be a "child" of this
  harness in any deeper sense (no cross-project linking is performed
  here) — that remains a separate, not-yet-designed piece.
