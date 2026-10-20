# Observation — 2026-09-06

## Session identity

- **Model**: Claude Code running **Sonnet 5** (Claude-Sonnet-5). Captured
  from this session's own reliably-exposed system context — the harness
  stating its own model plainly — not by asking the model to self-report
  in conversation, per the standing rule this same session reinforced
  (see "Grounded signals" and the founding MAL rejection below).
- **Harness state**: at the close of this session, Free Wings has **6
  agents** (`programmer`, `tester`, `observer`, `writer`, `researcher`,
  `the-architect`) and **4 skills** (`construct`, `write-diary`,
  `write-article`, `loop-status`) — confirmed against both a real
  directory listing (`.claude/agents/`, `.claude/skills/`) and
  `FOUNDATION.md`'s own "Current status" section, which agree. This was
  not a quiet session for the harness itself: two agents changed name/
  identifier (`architect` → `the-architect`), one agent was added
  (`tester`), two existing agents gained new standing rules
  (`researcher`'s validation checkpoint, `observer`'s session-identity
  capture and grounded signals), and one new skill (`loop-status`) was
  built — all same-day, all propagated through Foundation Sync.

## Range observed

Commits `be68a98` through `e42df52` (8 commits, all dated 2026-09-06):

1. `be68a98` — proximity-aware referral between agents
2. `dca275e` — add `tester`, rename `architect` → The Architect, Foundation Sync
3. `7cb0bf8` — unify The Architect's identifier, add ADR judgment call
4. `ca26c2a` — `researcher` gets a validation checkpoint
5. `e2298e9` — record researcher's findings on the metrics-grounding research
6. `357865e` — `observer` gets grounded signals, MAL proposal declined
7. `3504412` — `observer` captures session identity via a reliable source
8. `e42df52` — add `loop-status`

Cross-checked against `docs/LEARNING_LOG.md`'s entries for the same date;
narrative below is built from both, not from the log alone.

## The actual throughline

Read as a list of commits, this looks like six unrelated features bolted
onto the harness in one day. Read end to end, there's one real thread
connecting most of it: **this session is mostly the harness catching
scope creep and unverified claims about itself**, using its own tools to
do it, rather than adding capability for capability's sake. Two separate
proposals arrived mid-session that would have expanded the harness in a
bad direction, and both got stopped before anything was built — not
after.

## What was tried, what turned out wrong, and why

**A "Model Adaptation Layer" (MAL) proposal was declined.** The idea:
detect which LLM is running by asking the model its own name/version,
then have The Architect adjust context size, query complexity, and
iteration count per model, with a "Validator" agent reporting on it.
Declined on real, checkable grounds, not vibes: self-reported model
identification is a known-unreliable pattern; it assumed an already-
declined "Validator" agent existed to report into; the actual underlying
problem (different models need different treatment) already has a
working, cheaper answer in this harness — `CLAUDE.md` (dense) vs.
`AGENTS.md` (explicit), compiled once by `construct`, not detected at
runtime; and no evidence surfaced that this project actually juggles
multiple wildly-different-capability models in one working session — the
real trigger MAL assumed existed. Building for a hypothetical, unobserved
problem is exactly what this harness's own "smallest correct change" bias
already pushes back on.

**A pseudo-scientific "Validator" proposal was declined, and verified as
wrong, not just distrusted.** A long PDF proposed ten formula-heavy
metrics ("Índice de Reversibilidade," "Medida de Impacto Cognitivo,"
etc.), with a full numbered bibliography. Checked before building
anything: several cited references (brain-computer interfaces, quantum
decoherence, machine unlearning) aren't about software architecture and
aren't cited anywhere in the document's own body — citation padding, not
grounding. `researcher` (built specifically to stop this class of
mistake, after an earlier SDD-definition mistake) confirmed it directly:
Cognitive Load Theory, Shannon entropy, DORA, and SPACE are all real, but
none of them derive the PDF's actual invented formulas; "reversibility"
as a formal measurable metric doesn't exist in real software-engineering
literature — only as an informal heuristic, and in mismatched sources
about literal building demolition. The context/token-efficiency thread of
the same research request came back genuinely useful, by contrast:
"lost in the middle" and "context rot" are real, current, and well-
documented, with real low-infrastructure mitigations (observation
masking preferred over LLM summarization, specifically because
summarization has a documented failure mode of smoothing over how stuck
an agent actually is) — directly relevant to a harness whose own
`LEARNING_LOG.md`/observations only grow over time.

**A new hard rule came out of that verification, not just a rejected
proposal.** When `researcher`'s job is validating or comparing
data/metrics specifically (not a single fact check), it must end with a
real comprehension check and wait for confirmation before recommending
anything — because banking a false sense of "this is scientifically
validated" is harder to walk back than a plain wrong guess. Documented
honestly as a relay-level gate (a subagent can't literally pause
mid-execution), not a literal in-process pause.

**The identifier decision got re-checked, and the first check turned out
incomplete.** Earlier the same day, `architect`'s file got renamed to
`the-architect.md` while its technical identifier stayed the bare
`architect`, on the belief that Claude Code's kebab-case rule for
subagent `name:` fields disallowed hyphens. A follow-up question — could
the identifier itself include "the"? — triggered a second real check of
the *same* rule, which found what the first pass hadn't needed to
notice: kebab-case allows hyphens, so `the-architect` is a fully valid
identifier. The earlier split (different file name vs. identifier) was
undone; propagated everywhere it had just been written down. The lesson
worth keeping isn't the specific fact — it's that verifying a claim once
doesn't automatically cover a later, adjacent question about the same
claim.

**"Recognize and refer" had a real gap, caught by the person, not found
in process.** The rule as first written treated all agents as equally
distant, meaning every scope mismatch could bounce back through
`the-architect` even when the right next agent was already obvious.
Fixed with a concrete analogy (front-end/back-end engineers share more
tooling and vocabulary with each other than either shares with a
designer): two neighbor pairs were mapped from what each agent's own
file already said it hands off to — `observer` ↔ `writer`, and a
`researcher` → `programmer` → `tester` chain — with `the-architect`
reserved for genuinely unclear or multi-hop cases, not the default first
stop.

## What was learned / resolved this session

- `the-architect` is now uniformly the file name, identifier, and
  display name — no split left standing.
- Foundation Sync's fixed five-step cascade and a separate,
  judgment-based "does this deserve its own ADR" call are both now
  explicitly `the-architect`'s to run/decide.
- Proximity between agents is now mapped and written down, not implicit.
- "Modular & Self-Sufficient Documentation" was reinforced as a
  permanent standing requirement, not a one-time cleanup pass, and
  `the-architect`'s own Foundation Sync verifier now checks for it.
- `observer` (this agent) picked up the actual, grounded answer to what
  MAL was reaching for badly: session-identity capture from a reliable
  source, plus a small set of research-backed signals — never a scored
  formula. This observation is the first real live use of both.
- `loop-status` was built to make loop engineering's own four
  structural pieces (trigger, topology, verifier, stop rule — already
  grounded via `researcher` for `the-architect`'s own design, see
  `docs/reading-list.md`) visible on demand, reusing that same anatomy
  rather than inventing a new one.

## Grounded signals

- **Did the task ship**: Yes — all 8 commits landed and are committed;
  nothing in this range was reverted or left half-done.
- **Time vs. gut-feel estimate**: not captured this session — no time
  log was kept for this range, and no gut-feel estimate was stated
  anywhere in `docs/LEARNING_LOG.md`. Saying so plainly rather than
  guessing a number.
- **Did context need a manual correction mid-task**: Yes, twice in a
  concrete, checkable way within this exact range — (1) the identifier
  re-check, which found the first pass's kebab-case conclusion was
  incomplete, and (2) the MAL proposal being caught, before any code was
  written, as quietly contradicting the self-report rule this same
  session had just committed to (in the immediately preceding Validator/
  MAL rejection). Both corrections happened before implementation, not
  after — the more expensive kind of correction (a live rewrite) never
  actually happened here.
- **Subjective note**: not directly recorded in the person's own words
  for this specific range in `docs/LEARNING_LOG.md` — noted as not
  captured rather than inferred.
- **Agents/skills touched, one line each**:
  - `the-architect` — evolved substantially (rename, Foundation Sync,
    ADR judgment call, proximity mapping); worked as intended once the
    identifier issue was actually fixed.
  - `tester` — added new (provisional name); not yet exercised against
    any real `programmer` output.
  - `researcher` — gained the validation checkpoint rule, then used for
    real the same session to check the Validator PDF; worked exactly as
    designed, catching fabricated/mismatched citations before anything
    was built on them.
  - `observer` — gained session-identity capture and grounded signals;
    this file is its first real live use of both.
  - `loop-status` — added new; not yet exercised at a real proactive
    checkpoint (no Foundation Sync run or multi-piece build has shown it
    live yet).
  - `programmer`, `writer` — untouched and unused this session.

## Still open / uncertain

- `tester` and `loop-status` are both real but unexercised — no live
  test against a `programmer` deliverable, no live proactive checkpoint
  display yet.
- No time-vs-estimate baseline exists yet for any session, this one
  included — that grounded signal has nothing to compare against.
- The `CLAUDE.md`-dense-vs-`AGENTS.md`-explicit split is still an
  untested assumption; `construct`'s passive model-identity logging
  exists specifically to make this checkable later, but no data has
  accumulated yet.
- Peter Steinberger's original loop-engineering-adjacent quote is
  flagged in `docs/reading-list.md` as not independently confirmed by
  URL — a small, still-open verification thread.
- No spec was written for any of this session's own agent/skill work,
  despite the standing rule (from `FOUNDATION.md`'s "Specs" section)
  requiring one for non-trivial pieces — an inconsistency already named
  in the founding log entry and still unresolved as of this range.

## Flag for `writer` / possible Field Notes material

- **"A harness catching bad ideas about itself, using its own agents"**
  — two declined proposals (MAL, the pseudo-scientific Validator) in one
  session, both stopped by verification rather than by taste, reads like
  strong diary material: it's a concrete demonstration of the harness's
  stated philosophy (verify before building) actually holding under
  pressure from real, plausible-sounding proposals, not just stated as
  principle.
- **The loop-engineering four-piece anatomy (trigger, topology, verifier,
  stop rule)** has now grounded two separate pieces of this harness
  (`the-architect`'s own design, and `loop-status`'s block format) —
  reused twice, not incidental. That's enough depth/reuse to be worth a
  standalone Field Notes visual explainer, rather than staying implicit
  across two separate agent/skill files.
