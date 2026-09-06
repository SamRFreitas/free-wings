# Logbook — Free Wings

## 2026-09-06 — `hangar/`: scaffolding a new project without any copy-paste

Real design correction, caught by the person before any code was
written blind: the first version of "what a new project needs to adopt
this harness" was going to be a folder the person manually copied files
out of — called `template/` at first. Two things wrong with that, both
caught in conversation, not found later:

1. **The name.** "Template" said nothing about what the folder actually
   was. Tried "runway" next (an aircraft needs one before it flies) —
   misread as "runaway" (something out of control), which is close to
   the opposite of the intended meaning; a real reminder that a name
   only works if it reads right, not just if the reasoning behind it is
   sound. Landed on **`hangar`** instead — where a plane is prepared
   before it flies — same aviation register as the rest of this
   project's naming, and it read correctly on the first try.
2. **The mechanism.** Manual copy-paste directly contradicts this
   harness's own reason for existing — the entire point of `construct`
   was to stop hand-maintaining things that a skill could keep in sync
   automatically. `hangar/` is a **blueprint construct reads**, not a
   folder a person copies. `construct` was extended: finding no
   `FOUNDATION.md` in a target project no longer just stops and offers
   conversation — it now also scaffolds `docs/decisions/`,
   `docs/specs/`, `docs/observations/`, and a `docs/LEARNING_LOG.md`
   skeleton automatically, using `hangar/`'s own copies as its
   reference, before starting the dialogue that still — correctly,
   deliberately — cannot be automated: a project's actual
   `FOUNDATION.md` content has to come from the person, not be guessed.

Not yet tested end to end (no real new project has gone through this
scaffolding path yet) — `README.md`, `FOUNDATION.md`, the generated
`CLAUDE.md`/`AGENTS.md`, and `construct.md` itself were all updated to
describe the new mechanism; the mechanism's actual first live use is
still pending.

## 2026-09-05/06 — Founding: from "Harness Journal" to Free Wings

Born out of a Shadow Glass session (2026-09-04), during the NVENC-vs-
FFmpeg decision: the back-and-forth pattern that resolved it — propose,
get pushed back on with a real argument, genuinely reconsider, resolve
what's left via `grilling`'s round-based questions — got named
"Architecture Grilling," and the person wanted a place to write about
harness engineering itself, not just Shadow Glass's own technical
progress.

**The core pattern, arrived at through its own real pain, not designed
up front**: Shadow Glass's `CLAUDE.md`/`AGENTS.md` had to be hand-edited,
separately, every time something changed — they drifted. The fix: one
dense, tool-agnostic `FOUNDATION.md` per project, with a `construct`
skill generating `CLAUDE.md`/`AGENTS.md` (and any future tool's file)
from it. Verified twice, for real: once regenerating from a first draft,
once propagating an actual name change (Harness Journal → Free Wings)
through both generated files correctly.

**Named Free Wings (Asas Livres) on 2026-09-06.** Two failed rounds of a
formal naming brief (Freire/Dumont, bilingual, metaphor, no brand
conflicts) before the person just decided on their own: "Wings" for
Santos Dumont's flight, "Free" for Freire's liberation — neither half
needing the other person's name attached to be legible.

**Two real process corrections this session, both worth remembering as
a pattern, not just isolated mistakes:**
1. An `ExitPlanMode` approval got treated as if it meant "the user
   agrees with the recommendation written in the plan" — it didn't;
   approving a plan is a mechanical gate, not endorsement of every open
   question inside it.
2. Spec-Driven Development got asserted as "exactly what we already
   built" without checking the actual current definition first, or
   asking whether the person's own source for the term agreed. Corrected
   by looking it up for real (not from memory) and asking directly
   before writing anything — landed on a more honest conclusion: the
   *principle* matches (spec as primary artifact, output as regenerable),
   but classic SDD drives *code*; this harness applies the same
   principle to *harness configuration* instead, one layer removed from
   the textbook case, not identical to it.

**Built this session**: the `construct` skill; three general-purpose
agents (`programmer`, `observer`, `writer`) — reusable by any project
under this harness, not tied to Shadow Glass — and three writing skills
(`write-diary`, `write-article`, `write-linkedin-post`) that read
`writer.md` as their single source instead of duplicating its themes.
Also added `docs/specs/` (fed by `grilling`, same mechanism ADRs already
use, different destination depending on scale) as a third document type
alongside ADRs and the log.

**Corrections made to the agents after first drafting them, worth
remembering as design lessons**:
- `programmer` originally said explanation and implementation "arrive
  together, neither first" — wrong; the actual intent is explain first,
  then implement while continuing to teach through the code's own
  comments, not a simultaneous, undifferentiated blend.
- `observer` was drafted fully read-only, with no way to persist what it
  observed — meaning an observation would vanish the moment the
  conversation ended. Fixed with a narrowly-scoped `docs/observations/`
  folder it can write to, and nothing else.

**Shadow Glass got its own `FOUNDATION.md`** (2026-09-06) — the first
real target project under this harness, though its helper/cross-project
wiring hasn't been built or tested yet.

**Real gaps identified before closing this session, some resolved
same-day, some deferred:**
- No spec was written for any of today's own work, despite the rule
  just being established — an honest inconsistency, not swept under.
  Specs are planned to actually start once `programmer` is tested
  against Shadow Glass.
- `programmer`/`observer`/`writer` and the three writing skills were
  unused until this entry — only `construct` had real, repeated
  validation before today.
- No `LICENSE` yet, on a public repository.
- No human-facing `README.md` yet — everything written so far
  (`FOUNDATION.md`, `CLAUDE.md`, `AGENTS.md`) is AI-instruction-shaped,
  nothing aimed at a person landing on the GitHub page cold.
- `docs/decisions/` exists as a folder but is still empty — today's real
  decisions (adopting the spec/grilling pattern, the final name) live in
  `FOUNDATION.md`'s running prose, not as their own ADR files.
- The `simstim`-or-similar scaffolding skill for onboarding a brand-new
  AI tool this harness doesn't support yet is still just a name, not
  built.

**Next steps**: license decision, README (with usage instructions for a
project adopting this harness, and a short Freire/Santos Dumont section
for anyone arriving from outside), a directory Snapshot of this
repository once everything above settles, and real tests of the
untested agents/skills against Shadow Glass.
