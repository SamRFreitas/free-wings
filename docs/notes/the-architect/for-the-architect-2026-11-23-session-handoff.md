# For the-architect — where the 2026-11-17 to 2026-11-23 session stopped

For whoever continues in Free Wings. Nothing here is decided beyond
what it says.

## Not committed yet

- `docs/research/2026-11-23-cost-tracking-per-agent.md` — new.
- `docs/notes/the-architect/for-the-architect-2026-11-17-cost-tracking-research.md`
  — corrected (worker token figures are last-request size, not cost).

The person commits when he asks, one commit per file; he always pushes.

## Done outside this repository, not yet in the diary

- **Chacal** (`~/Lab/chacal`): recompiled after the ADR 0006 sync
  (`.claude/` and the harness sections of `CLAUDE.md`), and
  `docs/specs/TEMPLATE.md` copied in. Its live cost test comes with
  the person's next ATS spec there.

Recorded in the diary entry of 2026-11-23.

## Open in Free Wings

1. **Cost tracking and the model experiment** — see
   `for-the-architect-2026-11-17-cost-tracking-research.md`.
   Agent quality evaluation researched on 2026-11-23
   (`docs/research/2026-11-23-agent-quality-evaluation.md`); choosing a
   method is on hold, by the person's decision. A before/after
   comparison of past runs is not evidence (that research, section 5).
2. **An evaluator agent?** `programmer` and `tester` were not built to
   evaluate a person's work. An open idea, no evidence yet; a new agent
   would be a Foundation Sync.
3. **Shadow Glass**: recompile, copy `docs/specs/TEMPLATE.md`, name
   its implementer in its `FOUNDATION.md`. No commits there from here.
4. **Test** whether `effort` in an agent file applies under
   `--agent`.
5. Optional: a deny rule for `Agent(programmer)`; regenerate the
   OpenCode `AGENTS.md`.
