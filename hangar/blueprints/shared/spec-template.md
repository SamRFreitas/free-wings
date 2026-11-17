# Spec Template

<!--
Lives in `hangar/blueprints/shared/`. Defines the anatomy of a spec: what one piece of implementation work will do, written before it is implemented. Read by `construct`, which copies everything after the first `---` line into a new project's `docs/specs/TEMPLATE.md`; the project then owns that copy. Filled with the person by the lead agent through the `write-spec` skill (`hangar/blueprints/skills/write-spec.md`); a worker only executes it. Grounded in `docs/research/2026-11-17-spec-anatomy.md` and `2026-11-17-spec-kit-and-formats.md`.
-->

---

# NNNN — <title>

Status: draft | closed | done · Links: ADR-…, plan-…, docs/research/…

## Goal
1–3 lines: what changes, for whom, and why.

## Context
Only what the worker cannot read from the repository. Files to read.

## Scope
- In:
- Out:

## Slices
P1, P2… Each delivers value alone and lists its requirement IDs.

## Requirements
| ID | Requirement (what, never how) | How to verify | Status |
| :- | :- | :- | :- |
| FR-1 | | | pending / done / failed |

## Assumptions
Defaults chosen instead of asking. The person reviews them.

## Open questions
At most 3. Empty before the status is `closed`.

## Constraints
Must / must not. Files not to touch.

## Design (optional)
"How" decisions made with the person. Tables, UML or C4 only if the
project uses them.

## Run record
Filled by the lead agent when the spec is done.

Size: S | M | L (from the number of slices and requirements)

| Agent | Role | Model / effort | Tokens | Time |
| :- | :- | :- | -: | -: |
