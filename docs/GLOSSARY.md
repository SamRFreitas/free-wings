# Glossary

<!--
Lives in `docs/` of Free Wings: the project's own vocabulary, one entry
per term, alphabetical. Terms that name a concept of the harness are
defined in full in `FOUNDATION.md` ("Terminology"); this file is where a
term is looked up quickly. Any agent may add one (rule in
`hangar/blueprints/shared/agent-rules.md`, "Notes").
-->

**Closed spec** — a spec with no open questions and every requirement
carrying a "How to verify". Implementation and verification work
(`engenir`, `tester`) is delegated to a worker only with a closed spec.

**Lead agent** — the agent running the session's conversation with the
person. By default, `the-architect`.

**Orchestrator** — the pattern the lead agent follows when it breaks a
task down, delegates to workers and synthesizes what they return.
Anthropic calls it "orchestrator-workers".

**Run record** — the section at the end of a spec, filled by the lead
agent when the work is done: agents that took part, model and effort,
tokens, time, and the task's size (S, M or L).

**Subagent** — Claude Code's word for a worker. Used only in the Claude
Code adapter.

**Worker** — an agent running delegated, usually in parallel. It takes a
closed spec or a self-contained brief, works alone and returns; if it
hits a doubt, it stops and returns the question.
