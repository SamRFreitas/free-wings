# Logbook — Free Wings

## 2026-09-06 — A "Model Adaptation Layer" proposal, declined; `observer` gets grounded signals instead

A second proposal arrived, "MAL" — a dynamic system that would detect
which LLM is running (by asking the model its own name/version and
parsing the answer) and have The Architect adjust context size, query
complexity, and iteration count per model, with "The Validator"
reporting on it. Declined, for reasons distinct from the earlier PDF's
problem (this one wasn't fabricated citations — it's an engineering
proposal with real, checkable flaws):

- Self-reported model identification is a known-unreliable pattern —
  models frequently don't know their own exact version, or the harness
  hosting them doesn't expose it. Building adaptation logic on top of an
  unreliable detection step is building on sand.
- It assumes "The Validator" exists to report into — the same agent
  already declined as originally specified.
- The actual problem it's solving — different models need different
  treatment — already has a working, much cheaper answer in this
  harness: `CLAUDE.md` (dense, for a strong model) versus `AGENTS.md`
  (explicit, for a possibly weaker one), compiled once by `construct`,
  not detected at runtime.
- No evidence surfaced that this project actually juggles multiple,
  wildly different-capability models within a single working session —
  the real trigger MAL assumes. Building for a hypothetical, unobserved
  problem is exactly what this harness's own principles (smallest
  correct change, no abstraction the project doesn't need yet) already
  push back on. If it becomes a real, observed problem later, real prior
  art already exists to reuse (LiteLLM, OpenRouter-style model routing)
  rather than reinventing it from scratch.

Instead of a new agent, `observer` picked up the actual, grounded
responsibility: a small set of research-backed signals (task shipped,
time vs. gut-feel estimate, whether context needed a manual mid-task
correction — the honest low-infrastructure proxy for context rot the
research actually supports — and a short subjective note), folded into
the same observation account it already produces. Deliberately not a
score or a formula — the exact trap the earlier "Validator" proposal
fell into. Reusing an existing agent's scope instead of adding a new one
for every new concern is itself the point: this harness already worried,
out loud, about validation work turning into bureaucracy for its own
sake, and multiplying agents is how that actually happens.

## 2026-09-06 — A pseudo-scientific PDF, and a new checkpoint rule for `researcher`

A long document arrived proposing "The Validator" — a new agent built to
implement ten formula-heavy metrics (an "Índice de Reversibilidade," a
"Medida de Impacto Cognitivo," etc.), presented as grounded in academic
literature with a full numbered bibliography. Checked before building
anything on it, not assumed: several numbered references in that
bibliography (brain-computer interfaces, quantum circuit decoherence,
machine unlearning) aren't about software architecture at all and don't
appear cited anywhere in the document's own body text — a strong sign of
citation padding from an under-curated "deep research" tool output, not
genuine grounding. The reversibility formula's own cited sources turned
out to be about literal building demolition/reuse (circular-economy
architecture, the physical kind), with the formula itself invented, not
derived from anything in those sources. Flagged plainly instead of
building "The Validator" as specified — the same "verify before
building" discipline this project already learned from the SDD mistake,
applied here before a much larger investment, not after.

The response to this was to hand it to `researcher`, not to design an
agent from an unverified source. But a new requirement came with it:
when `researcher`'s job is *validating or comparing data or metrics* —
not just checking one fact — it must not hand off or recommend a next
step in the same report as its findings. It has to end with a real
comprehension check first, and wait for the person to actually confirm,
in their own words, that they understand, before anything moves forward.
This is Freire's dialogic principle again, but tightened into a hard
gate specifically for this kind of task, because the risk this time
wasn't "banking" a wrong implementation decision — it was banking a
*metric that sounds authoritative without being genuinely understood by
anyone*, which is arguably worse, since a false sense of "this is
scientifically validated" is harder to walk back later than a plain
wrong guess. Documented honestly in `researcher.md` itself: a subagent
can't literally pause mid-execution for a live reply, so the rule is
implemented as "the report stops short of recommending next steps" — the
actual gate is enforced by whoever relays the report back to the person,
not inside the subagent's own single execution.

A first real research brief was then handed to `researcher`, covering
three threads: which parts of the PDF's academic grounding are real
versus fabricated; what current, legitimate literature actually says
about context/token efficiency for LLM coding agents (a real, active
research area — "lost in the middle," context rot — relevant to a
harness whose own `LEARNING_LOG.md`/observations/ADRs only grow larger
over a project's lifetime); and whether real prior art exists for a solo
developer validating whether an AI-assisted harness is actually helping.

**Result of that run**, independently re-verified via live search, not
from the flagged PDF: the earlier suspicion held up. Cognitive Load
Theory, Shannon entropy, DORA, and SPACE are all real, established —
but none of them actually derive the specific formulas the PDF built on
their names (a percentage-reduction cognitive-load formula, a
reversibility formula); the math was invented, the names borrowed for
credibility. "Reversibility" as a formal, measurable ADR metric wasn't
found anywhere in real software-engineering literature — only as an
informal heuristic ("defer irreversible decisions") and in the
mismatched physical-building sources already flagged. ADR-as-graph
contradiction detection turned out to be real but genuinely young
(2024-2026 papers, one explicitly a "vision" paper, not an established
technique) — worth knowing about, not worth treating as settled.

The context/token thread came back much stronger: "lost in the middle"
and "context rot" are well-documented and current, and real,
low-infrastructure, embedding-free heuristics already exist in practice
— budget-by-category with alert thresholds, recency-based pruning,
"observation masking" (dropping stale tool output wholesale) preferred
over LLM summarization specifically because naive summarization has a
documented failure mode of smoothing over how stuck an agent actually
is. Real prior art exists for pruning a growing log for LLM context
specifically (not invented for this project) — with the same
summarization-smooths-over-severity caveat applying directly to this
project's own `LEARNING_LOG.md` if it's ever auto-summarized.

For whether AI assistance actually helps a solo developer: the closest
real research (GitHub Copilot studies) is genuinely mixed — real gains
in some controlled studies, no significant effect in at least one — so
there's no single citable number to anchor a productivity claim on. The
honest minimal check `researcher` synthesized from what the literature
actually supports: track a small number of directly observable things
(did the task ship, rough time vs. gut estimate, did context need a
manual correction mid-task, a periodic subjective note) rather than a
composite formula the underlying research doesn't support.

Per `researcher`'s own new checkpoint rule, its report ended with a
plain-terms summary and a direct question back to the person, instead of
recommending what to build next — no context-checker agent designed yet
as a result of this entry; that's still an open next step, deliberately
left open here rather than closed.

## 2026-09-06 — Verifying a second time, an ADR judgment call, and dividing work in two agents

Follow-up to the previous entry, three real refinements:

**The identifier question deserved a second real check, not an
assumption built on the first one.** The person asked directly: could
`the-architect`'s technical identifier include "the," not just its file
name and display name? The earlier entry had checked Claude Code's docs
once and concluded the identifier had to stay plain `architect` (no
spaces or capitals allowed). Re-checking the *same* rule for this new
question found what the first pass hadn't needed to notice: kebab-case
allows hyphens, and `the-architect` is entirely valid. There was no
remaining reason to keep the file name and the identifier different once
that was seen — so the split from the previous entry was undone.
Identifier, file name, and display name are now all `the-architect` (in
prose: "The Architect"). Propagated everywhere the split had just been
written down: `the-architect.md` itself, `programmer.md`, `observer.md`,
`writer.md`, `researcher.md`, `FOUNDATION.md`, `CLAUDE.md`/`AGENTS.md`,
`README.md`, `docs/project-snapshot-2026-09-06.html`. The lesson worth
keeping isn't the specific fact (kebab-case allows hyphens) — it's that
verifying a claim once doesn't cover every question that claim later
turns out to be adjacent to; the follow-up question got its own real
check rather than reasoning from the first answer.

**The Architect now owns a judgment call about ADRs, not just the fixed
Foundation Sync cascade.** The five-step cascade (`FOUNDATION.md` →
`construct` → `README.md` → `LEARNING_LOG.md` → `docs/learning-*.html`)
always runs. Whether a harness change is also significant enough — a
real architectural fork, with real alternatives weighed, not just any
addition — to deserve its own new entry in `docs/decisions/` is a
separate, judgment-based decision, now explicitly The Architect's to
make, following the existing "rare, one per genuine fork" ADR standard.

**"Divide and conquer, and let the person try first" — specific to
`programmer` and `tester` only, not a harness-wide rule.** Both now work
in small, deliberately divided steps (one function, one command, one
test at a time), split anything genuinely complex further, and actively
check the person's own understanding while working — asking what they'd
do or expect before revealing an answer, and sometimes letting them
attempt a piece themselves first, guiding from what they actually
produce rather than always producing the answer first.

**Modular & Self-Sufficient Documentation was reinforced as permanent,
not a one-time pass** — explicit in `FOUNDATION.md`, `CLAUDE.md`/
`AGENTS.md` now: every new agent, skill, or piece added to this harness
going forward is expected to carry its orientation note from the moment
it's created, and The Architect's own Foundation Sync verifier is
expected to check for it before treating a cascade as done.

All four changes propagated through the same Foundation Sync cascade
they describe — `FOUNDATION.md` first, then `CLAUDE.md`/`AGENTS.md`,
`README.md`, this entry, and `docs/learning-free-wings.html` (kept
uncommitted, as usual) — verified with a real grep for the old bare
`architect` identifier across the repository rather than trusted from
memory, the same verifier step this entry itself is describing.

## 2026-09-06 — `tester`, The Architect rename, Foundation Sync, and a documentation convention

Four real additions in one dialogue, all traced back to concrete things
the person asked for, none invented independently:

**`tester` (provisional name).** A sixth agent, testing whatever
`programmer` just built, in the same explain-first order `programmer`
itself uses: what will be tested, why, how — shown transparently while
it runs, not just reported as a pass/fail. Its closest neighbor is
`programmer`, extending what used to be a two-agent pair
(`researcher` ↔ `programmer`) into a three-agent chain:
`researcher` → `programmer` → `tester`. It inherits a real limit already
learned on this harness's own first target project: a GUI or networking
test binary on macOS gets run by the person in their own terminal, not
launched through this agent's `Bash` tool. The name is explicitly
provisional — a shorter, combined "reviewer+tester" name may replace it
later.

**"The Architect."** The `architect` agent gets a Matrix-style display
name in prose, matching the character it was already named after. Before
implementing this literally, Claude Code's own documentation was checked
directly (not assumed) for what a subagent's `name:` frontmatter field
actually allows: lowercase kebab-case only, no spaces, no capitals — so
"The Architect" could never be the literal invocation identifier. The
same documentation confirmed a subagent's filename doesn't have to match
its identifier at all, which is what made the actual fix possible: the
file is renamed `architect.md` → `the-architect.md`, the identifier
stays `architect` (unchanged, avoiding a much larger rewrite of every
existing cross-reference to it), and the file's own prose now explains
this split plainly rather than leaving it implicit.

**Foundation Sync.** A new, named responsibility for The Architect: any
dialogue that changes how this harness itself works — a new or renamed
agent, a changed behavior, a new standing rule — triggers a fixed
cascade (`FOUNDATION.md` → `construct` → `README.md` → a
`LEARNING_LOG.md` entry → `docs/learning-*.html`), verified by grepping
for the actual stale reference rather than trusted from memory, and not
considered finished until every file in the cascade has been checked.
This formalizes, as a real named rule, exactly the pattern this log
itself has been following by hand for the last several entries.

**Modular & Self-Sufficient Documentation.** A new standing convention,
credited honestly to where it actually came from: a LaTeX tutorial file
the person built for someone else's genuine first contact with LaTeX and
Overleaf, deliberately written so any section could be read on its own,
each one explaining itself rather than assuming everything above it had
already been read. Applied here: every agent/skill file now opens with a
short orientation note (what kind of file this is, its nearest
neighbors), and every agent is expected to apply the same standard when
explaining something to the person that they don't yet understand —
assume no prior context, orient before explaining.

All four propagated through `FOUNDATION.md`, `CLAUDE.md`/`AGENTS.md`,
`README.md`, `docs/reading-list.md`, and `docs/project-snapshot-2026-09-06.html`
in the same pass, verified with a real grep for stale `architect.md`
references and stale agent counts rather than trusted from memory — the
Foundation Sync verifier step, used on itself the same day it was
written down.

## 2026-09-06 — Proximity between agents: not every referral needs `architect`

A real gap in "recognize and refer," caught by the person right after
`architect` was built: the rule as written treated all five agents as
equally distant from each other, which meant every mismatch could end up
bouncing back through `architect` even when the right next agent was
already obvious to whoever hit the mismatch first — a wasted round trip.
The concrete example given: on a real team, a front-end engineer and a
back-end engineer talk constantly and share most of their tools and
vocabulary; a front-end engineer and a designer share noticeably less of
that, even though both also talk to the front-end engineer. Roles that
work adjacent to each other end up knowing enough about their neighbor's
job to hand off directly, without needing a manager to broker every
single handoff.

Mapped onto this harness's five agents, using what each agent's own file
already said it reads from or hands off to (not a new claim invented for
this) — two pairs turned out to already be exactly this kind of
neighbor: `observer` and `writer` (an observation exists specifically to
become a piece of writing), and `programmer` and `researcher` (grounding
a decision before implementing it non-trivially is already where the two
hand off). `architect` bridges both pairs rather than sitting equally
close to all four — it's the one to ask when the right next agent
genuinely isn't obvious, or a request needs more than one hop across both
pairs, not the default first stop for every mismatch.

Added a "Proximity between agents" section to `architect.md` explaining
the mapping, and updated every other agent's "Recognize and refer"
section to name its own nearest neighbor and refer straight to it when
that neighbor alone resolves the request. Propagated into
`FOUNDATION.md`, `CLAUDE.md`/`AGENTS.md`, and `README.md` too, so this
doesn't only live in one file — the same drift this harness exists to
prevent.

## 2026-09-06 — `architect`, grounded in loop engineering instead of assumed

A new agent was proposed — `architect`, an entry point that decides which
other agent a task belongs to — leaning on two named references: "loop
engineering" as its structural grounding, and "the most recent edition of
Pressman" as a comparison point. Neither got built on directly. Both were
genuinely unfamiliar enough to flag before designing anything: "loop
engineering" had only been seen in informal videos, not checked against
any real source; Pressman's current edition wasn't known with confidence
either. Both got handed to `researcher` — the agent this project already
built specifically to stop this class of mistake (see the SDD entry
further down this log) — before any design work started.

What actually came back: "loop engineering" traces to one real, checkable
source — Addy Osmani (a director on Google's Cloud AI team), in an essay
called *"Practical Loop Engineering,"* published on his own Substack on
2026-06-07 and syndicated with his permission by O'Reilly Radar shortly
after. Every source checked agrees on the same four structural pieces a
real agentic loop needs: **trigger, topology, verifier, stop rule**.
Pressman & Maxim's *Software Engineering: A Practitioner's Approach*
checked out too — 9th edition, the most recent one found — but unlike the
loop-engineering sources, which are free blog posts, it's a paid
textbook. Both facts, and the honest difference in accessibility between
them, are recorded in the new `docs/reading-list.md`, not just asserted
here.

`architect`'s own procedure was then built to mirror those four pieces
directly: its trigger is a request from a person or from another agent
that recognized a task wasn't its own job; its topology is the actual
recommendation it produces (which agent, or ordered sequence, fits);
its verifier is checking that recommendation against the target
project's real `FOUNDATION.md` and the recommended agent's own stated
scope; its stop rule is that once a recommendation — or an honest "this
fits none of them" — is given, its job for that request is done.

Alongside `architect`, a standing rule called **"recognize and refer"**
was added to every existing agent (`programmer`, `observer`, `writer`,
`researcher`): when a request falls outside an agent's own defined
scope, it says so and names which other agent fits better, instead of
attempting the work anyway or staying silent about the mismatch.
`FOUNDATION.md`, `CLAUDE.md`/`AGENTS.md`, `README.md`, and
`docs/learning-free-wings.html` were all updated to reflect five real
agents (not "programmer and observer built, three more planned") and
this new rule, rather than letting any of them drift stale the way
Shadow Glass's own `CLAUDE.md`/`AGENTS.md` pair once did before this
harness existed.

## 2026-09-06 — Skills weren't broken by session timing — they were the wrong shape

First real test of `construct`/`write-diary`/`write-article` as actual
invoked skills (not manually simulated, like every earlier "test" this
session): all three came back `Unknown skill`. The session had already
been restarted once specifically to pick up new skills, which made
"needs another restart" a reasonable-sounding next guess — and that
guess got written down (in this project's own snapshot page) as if it
were a confirmed finding, before actually being checked.

The real cause, found by comparing against an already-working skill
(`grilling`): Claude Code's real convention is a **folder** per skill,
`~/.claude/skills/<name>/SKILL.md` — not a flat `~/.claude/skills/
<name>.md` file. Every skill this project made was the wrong shape from
the very first one. Restructured all three into
`<name>/SKILL.md`, re-pointed the global symlinks at the folders instead
of the files — all three loaded correctly on the very next invocation,
same session, no restart involved at all.

Also corrected the false claim itself, in the snapshot page it was
written into, rather than leaving it standing once disproven.

Then actually ran all three for real: `construct` scaffolded a brand-new
directory's `docs/decisions/`, `docs/specs/`, `docs/observations/`, and
`docs/LEARNING_LOG.md` from `hangar/` correctly, and correctly stopped
before fabricating any `FOUNDATION.md` content. `write-diary` and
`write-article` both produced real, grounded output against Shadow
Glass, reverted/deleted immediately after, no test commits anywhere.

## 2026-09-06 — `CLAUDE.md`/`AGENTS.md` gitignored, not committed

Caught by the person, not found through any process: committing files
that are 100% regenerated from a tracked source (`FOUNDATION.md`) is the
same anti-pattern as committing a `build/` folder — and it silently
assumed every future contributor needs every supported tool's file, not
just the one they actually use. Fixed: both gitignored, removed from
tracking (`git rm --cached`, kept locally), `FOUNDATION.md`/`CLAUDE.md`/
`AGENTS.md`/`README.md` updated to describe and explain the change.
Framed as a general option any project under this harness can take, not
a rule forced onto Shadow Glass (which keeps them committed, its own
already-established practice).

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
