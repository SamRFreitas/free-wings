# Harness vocabulary — Free Wings' names against the standard terms

> Researched 2026-11-16 by `researcher`. Goal: each Free Wings piece
> keeps its own name, and its description also states the standard
> industry/literature term for what it is — improvement only works on
> things understood under shared names. Sources are primary only.
> Verdicts: **documented** (quoted), **inference** (my reasoning from
> documented material), **not found**. Portuguese equivalents are my own
> suggestions, not sourced: **inference** throughout.
>
> Decision recorded with the person on 2026-11-16: the harness's terms
> and descriptions stay in **English**; the summary table below is the
> one agreed.

## Agreed summary

| Free Wings piece | Standard term | In one line |
| :- | :- | :- |
| `the-architect` | router / coordinator | Routes each request to the right agent |
| `programmer` | implementer (teaching) | Explains, writes the spec, builds |
| `engenir` | implementer (direct) | Builds from the spec, without teaching |
| `tester` | verifier | Checks what was built |
| `researcher` | research agent | Grounds claims in primary sources |
| `deneir` | observer *(no standard term)* | Records how the project evolves |
| `writer` | writer *(no standard term)* | Turns raw material into prose |
| `construct` | generator / compiler | Generates each tool's files |
| `FOUNDATION.md` | single-source project context | The file everything is compiled from |
| `docs/notes/` | agentic memory | What agents leave written down |
| "Show first, then write" | human-in-the-loop | You approve before anything is written |
| "Recognize and refer" | handoff | Passing the task to the right agent |

The full mapping, with its basis in the sources, is section 4.

## 1. Claim checked

"Free Wings' pieces correspond to terms the industry and literature
already use; those terms can be named, with sources." Partly confirmed:
most pieces have a documented counterpart; a few have none; two Free
Wings words collide with a standard meaning.

## 2. Sources checked (all fetched 2026-11-16)

- Anthropic, "Building effective agents", 2024-12-19 — https://www.anthropic.com/engineering/building-effective-agents
- Anthropic, "Effective context engineering for AI agents", 2025-09-29 — https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
- Anthropic, "How we built our multi-agent research system", 2025-06-13 — https://www.anthropic.com/engineering/multi-agent-research-system
- Anthropic, "Effective harnesses for long-running agents", 2025-11-26 — https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents
- Anthropic, "Raising the bar on SWE-bench Verified", 2025-01-06 — https://www.anthropic.com/engineering/swe-bench-sonnet
- Anthropic, "Demystifying evals for AI agents", 2026-01-09 — https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents
- Claude Code docs: Glossary — https://code.claude.com/docs/en/glossary ; How Claude Code works — https://code.claude.com/docs/en/how-claude-code-works ; Extend Claude Code — https://code.claude.com/docs/en/features-overview ; Subagents — https://code.claude.com/docs/en/sub-agents (first 100k chars)
- OpenAI Agents SDK docs: Multi-agent — https://openai.github.io/openai-agents-python/multi_agent/ ; Guardrails — https://openai.github.io/openai-agents-python/guardrails/
- Model Context Protocol — https://modelcontextprotocol.io/docs/getting-started/intro
- Agent Skills — https://agentskills.io/home
- AGENTS.md — https://agents.md/
- Google ADK, Workflows — https://adk.dev/workflows/
- Birgitta Böckeler (Thoughtworks), "Understanding Spec-Driven-Development: Kiro, spec-kit, and Tessl", martinfowler.com, 2025-10-15 — https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html
- Birgitta Böckeler, "Harness engineering for coding agent users", martinfowler.com, 2026-04-02 — https://martinfowler.com/articles/exploring-gen-ai/harness-engineering.html
- Addy Osmani, "Agent Harness Engineering", 2026-04-19 — https://addyosmani.com/blog/agent-harness-engineering/
- Addy Osmani, "Loop Engineering", 2026-06-07 — https://addyosmani.com/blog/loop-engineering/
- Addy Osmani, "Practical Loop Engineering", Substack, 2026-08-14 — https://addyo.substack.com/p/practical-loop-engineering
- Sumers, Yao, Narasimhan, Griffiths, "Cognitive Architectures for Language Agents" (CoALA), arXiv:2309.02427, TMLR 2024 — https://arxiv.org/abs/2309.02427

Not reachable: OpenAI, "A practical guide to building agents" (PDF
fetched as binary, not extractable here); OpenAI, "Harness engineering"
(https://openai.com/index/harness-engineering/, HTTP 403); O'Reilly
"Loop Engineering" (403). Google ADK's pages no longer define the
pattern names (only navigation labels). Claims from those sources are
**not found** here, even where secondary summaries repeat them.

Caveat on method: pages were read through a fetch tool that summarizes;
quotes are as returned. A term reported as absent may have been missed.

## 3. Current vocabulary

**Agent** — documented, contested. Anthropic: agents are "systems where
LLMs dynamically direct their own processes and tool usage", distinct
from workflows, "systems where LLMs and tools are orchestrated through
predefined code paths" (Building effective agents). Anthropic again:
"multiple agents (LLMs autonomously using tools in a loop)" (multi-agent
research). Benchmark sense: "an 'agent' refers to the combination of an
AI model and the software scaffolding around it" (SWE-bench post). The
formula "Agent = Model + Harness" (credited by Osmani to Viv Trivedy;
used by Böckeler). Contested: the workflow/agent split is Anthropic's;
others call both "agents".

**Agentic loop** — documented (Claude Code). "The cycle Claude works
through for every task: gather context, take action, verify results,
and repeat until done" (Glossary).

**Harness / agent harness** — documented, recent, two scopes.
Claude Code Glossary: "The tools, context management, and execution
environment that turn a language model into a capable coding agent.
Claude Code is the harness; Claude is the model inside it." Osmani: a
harness is "every piece of code, configuration, and execution logic
that isn't the model itself". Böckeler narrows it to the *user's outer
harness* around a coding agent, and calls engineering it "a specific
form of context engineering"; she says the term is "otherwise very
generic" and "There's plenty still to figure out." Anthropic's evals
post separates **agent harness** (lets a model act as an agent) from
**evaluation harness** ("the infrastructure that runs evals
end-to-end"). Not settled: vendor docs mean the tool; practitioners
include the user's own configuration.

**Scaffold / scaffolding** — documented as a synonym of harness.
"This scaffolding is responsible for generating the prompts that go into
the model" (SWE-bench post); the evals post writes "Agent harness
(scaffold)".

**Workflow patterns** — documented (Building effective agents):
- Prompt chaining: "decomposes a task into a sequence of steps, where
  each LLM call processes the output of the previous one".
- Routing: "classifies an input and directs it to a specialized
  followup task".
- Parallelization: "LLMs can sometimes work simultaneously on a task and
  have their outputs aggregated programmatically" (sectioning, voting).
- Orchestrator-workers: "a central LLM dynamically breaks down tasks,
  delegates them to worker LLMs, and synthesizes their results".
- Evaluator-optimizer: "one LLM call generates a response while another
  provides evaluation and feedback in a loop".
OpenAI's SDK names close equivalents: "A manager agent keeps control of
the conversation and calls specialist agents" (agents as tools);
handoffs: "A triage agent routes the conversation to a specialist, and
that specialist becomes the active agent"; chaining, parallel runs, and
an evaluator loop. Google ADK: an agent "in a dynamic coordinator role"
with sub-agents. Same ideas, vendor-specific names.

**Subagent** — documented, Claude Code's term. "A specialized AI
assistant that runs in its own context window with a custom system
prompt, specific tool access, and independent permissions. It works on
a delegated task and returns a summary" (Glossary). A subagent cannot
use `AskUserQuestion` (Subagents page). `--agent <name>` runs a session
"where the main thread itself takes on that subagent's tool
restrictions and model". Anthropic's research system calls the parent
the **lead agent**.

**Multi-agent system** — documented. "A multi-agent system consists of
multiple agents (LLMs autonomously using tools in a loop) working
together"; poor fit where agents must share context or depend heavily
on each other, and "most coding tasks involve fewer truly
parallelizable tasks than research" (multi-agent research). Claude
Code also has **agent teams** (experimental, a team lead plus peers).

**Context engineering** — documented. "the set of strategies for
curating and maintaining the optimal set of tokens", presented as the
progression of prompt engineering (Effective context engineering).

**Context window** — documented. "The working memory for a session,
holding conversation history, file contents, command outputs, CLAUDE.md,
auto memory, loaded skills, and system instructions" (Glossary).
Related: "context rot", "attention budget" (context engineering post).

**Compaction** — documented. "Automatic summarization of your
conversation when the context window approaches its limit" (Glossary);
"taking a conversation nearing the context window limit, summarizing
its contents" (context engineering post).

**Memory** — documented, several meanings. Agentic memory: "Structured
note-taking, or agentic memory, is a technique where the agent regularly
writes notes persisted to memory" (context engineering post). Claude
Code: CLAUDE.md is "persistent instructions you write"; auto memory is
"Notes Claude writes for itself" (Glossary). Academic: CoALA describes
"a language agent with modular memory components, a structured action
space". Böckeler: some tools call general context files a "memory bank".

**Tool** — documented. "An action Claude can take: read a file, edit
code, run a shell command, search the web, spawn a subagent" (Glossary).
Free Wings already separates this from "AI-Assisted Tool".

**Skill** — documented, open standard. "Agent Skills are a lightweight,
open format for extending AI agent capabilities with specialized
knowledge and workflows", loaded by "progressive disclosure"; originally
by Anthropic, released as an open standard (agentskills.io). Claude
Code: skills are "the recommended successor to custom commands".

**MCP** — documented. "an open-source standard for connecting AI
applications to external systems" (modelcontextprotocol.io).

**Hook** — documented. "A user-defined handler that executes
automatically at a specific point in Claude Code's lifecycle … Hooks
are deterministic" (Glossary). "Put guardrails in hooks. An instruction
like 'never edit .env' in CLAUDE.md or a skill is a request, not a
guarantee" (Extend Claude Code). Osmani: hooks are "the enforcement
layer".

**Spec-driven development** — documented, unsettled. "Spec-driven
development means writing a 'spec' before writing code with AI"; levels
spec-first, spec-anchored, spec-as-source ("only the spec is edited by
the human, the human never touches the code"); "the term … isn't very
well defined yet" (Böckeler, 2025-10-15). spec-kit's "constitution"
holds "high level principles that are 'immutable'".

**Plan mode** — documented, Claude Code-specific. "A permission mode
where Claude researches and proposes changes without editing your
source files … then presents a plan for approval" (Glossary).

**Evals / verifier** — documented. Eval: "a test for an AI system: give
an AI an input, then apply grading logic to its output"; grader: "logic
that scores some aspect of the agent's performance"; graders are
"code-based, model-based, and human" (evals post). Claude Code
**verification loop**: "You give Claude a check it can run … without
one, the only thing deciding the agent is finished is the agent itself"
(Glossary). Böckeler: **sensors** "observe *after* the agent acts and
help it self-correct", either computational or inferential; **guides**
steer "*before* it acts". Osmani (Loop Engineering): "split the verifier
sub-agent from the maker"; heading "keep the maker away from the
checker". Stop: "stopping conditions (such as a maximum number of
iterations)" (Building effective agents); "stop condition" (Osmani).

**Guardrails** — documented, vendor-specific. OpenAI SDK: "Guardrails
enable you to do checks and validations of user input and agent
output", plus tool guardrails and "tripwires". Anthropic uses the word
without defining it; Claude Code says to put guardrails in hooks.

**Human-in-the-loop** — documented in substance, not as a defined
term. "Agents can then pause for human feedback at checkpoints or when
encountering blockers" (Building effective agents). Claude Code: "You're
part of this loop too. You can interrupt at any point"; the Manual
permission mode: "Claude asks before file edits and shell commands".
Note: Claude Code's **checkpoint** is a different thing (a file restore
point).

**Project instruction files** — documented. CLAUDE.md: "persistent
instructions you write for Claude, loaded at the start of every
session". AGENTS.md: "a README for agents"; "One AGENTS.md works across
many agents"; stewarded by the Agentic AI Foundation (Linux Foundation).
Claude Code reads AGENTS.md when there is no CLAUDE.md.

## 4. Mapping table

"Says it?" = whether the current Free Wings description already names
the standard term (checked by grep of `FOUNDATION.md`, `CLAUDE.md` and
`hangar/blueprints/`, 2026-11-16).

| Free Wings piece | Standard term(s) | Says it? | Suggested role phrase (EN) | PT (inference) | Basis |
|---|---|---|---|---|---|
| Free Wings itself | user / outer **harness** (Böckeler); harness-engineering layer | Calls itself "harness" | "A user-side harness: the configuration layer around an agentic harness such as Claude Code" | "harness do usuário, a camada em volta da ferramenta" | documented (Böckeler, Glossary); **collision**, see §5 |
| `FOUNDATION.md` | project instructions / context file (CLAUDE.md, AGENTS.md); "constitution" (spec-kit); "memory bank" | No ("single source of truth") | "Single-source project context: the hand-written file every project instruction file is compiled from" | "fonte única de contexto do projeto" | documented terms; the mapping is inference |
| Generated `CLAUDE.md` | project instructions; always-on context; Claude Code "memory" file | Partly ("generated instruction file") | "Compiled project instructions, loaded every session" | "instruções do projeto compiladas" | documented (Glossary, Extend) |
| `construct` | **generator / compiler** (build step); scaffolder for new projects (cf. Claude Code `/init`, which "generates a starter CLAUDE.md") | Yes: compiler analogy in `FOUNDATION.md`; "bootstrapper" | "Generator: compiles FOUNDATION.md and blueprints into one tool's instruction, agent and skill files" | "gerador / compilador" | compiler analogy is Free Wings' own; `/init` documented; rest inference |
| Blueprint | agent / skill **definition** (Claude Code "custom subagents", `SKILL.md`); tool-agnostic template | No | "Tool-agnostic agent or skill definition, compiled per tool" | "definição (molde) de agente ou skill" | inference; **not found** as a standard term |
| Adapter | compiler **back-end** / target | Yes (front-end/back-end analogy) | "Per-tool compilation back-end" | "back-end por ferramenta" | Free Wings' own analogy; GoF "adapter" is a different meaning (not re-checked: **inference**) |
| `shared/agent-rules.md` | shared system-prompt section; cf. Claude Code `.claude/rules/` | No | "Shared rules appended to every agent's system prompt" | "regras comuns a todos os agentes" | inference |
| `the-architect` | **router** (routing); **coordinator** (ADK), manager / lead agent when it delegates (orchestrator-workers); "triage agent" (OpenAI handoffs) | No (says "entry point… decides which agent") | "Router and coordinator: classifies each request, routes it to the right agent or says none fits; owns harness consistency" | "roteador e coordenador" | documented patterns; fit is inference: it mostly *recommends* rather than synthesizing worker results, so "router" fits better than "orchestrator" |
| `programmer` | **implementer** / coding agent; "maker" (Osmani); generator side of evaluator-optimizer | "implements", yes; no standard term | "Implementer (coding agent) that teaches: explains, writes a spec, builds in small pieces" | "implementador que ensina" | documented patterns; fit is inference |
| `engenir` | same as `programmer`, without teaching | same | "Implementer (coding agent) that builds from a spec without teaching" | "implementador direto" | as above |
| `tester` | **verifier** / checker (Osmani); **evaluator** (evaluator-optimizer); grader; sensor (Böckeler) | "verifies", yes; not the noun | "Verifier: checks the implementer's work, claims only what it ran, hands interactive checks to the person" | "verificador" | documented; Free Wings' human-first testing = human grader (evals post), inference |
| `researcher` | research agent (Anthropic research system); **grounding** | No | "Research agent: grounds claims in cited primary sources before they're used" | "agente de pesquisa" | research system documented; "grounding" as a term **not found** in the sources checked |
| `deneir` | no standard term; nearest are observability (Osmani: "logs, traces, and cost and latency metering") and agentic memory | n/a | "Observer: records how a project and its harness evolve, as qualitative notes" | "observador / cronista" | **not found**; the observability comparison is inference and only partial (`deneir` is qualitative, not telemetry) |
| `writer` | none specific | n/a | "Writer: turns raw project material into diary entries and articles" | "redator" | **not found** |
| Skills (`/loop-status`, `/write-diary`, `/write-article`) | **skill** (Agent Skills standard) | Yes | keep, e.g. "/loop-status — progress-report skill" | "skill" | documented |
| `docs/LEARNING_LOG.md` (diary) | progress log (cf. Anthropic's `claude-progress.txt`, "a log of what agents have done") | No | "Diary: the project's progress log, wrong turns included" | "diário de bordo" | documented analogue; but Free Wings' diary is written for people, Anthropic's for agents: inference |
| `docs/notes/<agent>/` | **structured note-taking / agentic memory** | No | "Agent notes: agentic memory kept as files" | "notas dos agentes (memória)" | documented (context engineering post) |
| `docs/specs/` | **spec** in spec-driven development, at the *spec-first* level (spec-anchored if kept) | Says "Spec-Driven Development", but as spec-as-source | "Spec-first: a written spec before each non-trivial piece" | "especificação antes do código" | documented levels; **mismatch**, see §5 |
| ADR | Architecture Decision Record | Yes | keep | "registro de decisão de arquitetura" | standard term, origin not re-checked this session |
| `docs/GLOSSARY.md` | glossary | Yes | keep | "glossário" | — |
| Foundation Sync | no single standard term; parts map to verification loop / sensor (its grep verifier) and stopping condition (its stop rule) | Uses "verifier", "stop rule" | "Consistency cascade: regenerate, verify by grep, stop when every step is checked" | "cascata de consistência" | parts documented; whole **not found** |
| "Show first, then write" | **human-in-the-loop** approval; Claude Code's Manual permission mode; plan mode | No | "Human-in-the-loop approval gate on every write" | "aprovação humana antes de escrever" | documented in substance |
| "Recognize and refer" / nearest neighbor | **handoff** (OpenAI SDK); together with `the-architect` = manager plus decentralized handoffs | No | "Handoff to the right agent" | "passagem / encaminhamento" | documented (OpenAI SDK); mixing the two patterns is inference |
| `researcher → implementer → tester` | prompt chaining across sessions; implementer + tester ≈ evaluator-optimizer | No | "A chain: research, then build, then verify" | "cadeia" | inference: the human carries messages between sessions, so it is not automated chaining |
| `HARI-SELDON.md` | project template / constitution template (spec-kit) | Calls it scaffolding | "Template for a new project's FOUNDATION.md" | "molde de fundação de projeto" | inference; **collision** on "scaffold" |

## 5. Collisions, mismatches, and where Free Wings is clearer

- **"Harness"** — documented collision. In Claude Code's docs, "Claude
  Code is the harness". Free Wings is a layer *on* that harness, which
  is Böckeler's "user harness". Writing "user-side harness" avoids the
  clash.
- **"Scaffold"** — collision. Anthropic uses "scaffold" to mean the
  agent harness; Free Wings uses it for creating a project's
  `FOUNDATION.md`. Inference: "project template" or "bootstrap" is
  clearer.
- **Spec-Driven Development** — mismatch. `FOUNDATION.md` describes
  "a written spec as the primary source, with implementation as a
  regenerable output". In Böckeler's terms that is *spec-as-source*, the
  level she describes as aspirational. Free Wings' `docs/specs` practice
  (a spec written before implementing) is *spec-first*. Only
  `FOUNDATION.md` → generated files comes close to spec-as-source.
- **Enforcement** — documented gap. Every Free Wings rule (show first,
  the Foundation Sync rules, commit gate) is a prompt instruction.
  Claude Code: such an instruction "is a request, not a guarantee"; "If
  a rule must hold every time, make it a hook".
- **Where Free Wings is clearer**: "AI-Assisted Tool" vs "tool" (the
  industry overloads "tool"; Claude Code's own glossary uses it for
  model actions); `programmer` vs `engenir` (no standard term separates
  an implementer that teaches from one that doesn't); explicitly
  keeping "artifact" out (Claude Code's glossary confirms Artifact is a
  product feature).

## 6. Discrepancy in this project's own grounding (loop engineering)

`docs/reading-list.md` lists Osmani's original essay as
https://addyo.substack.com/p/practical-loop-engineering, published
2026-06-07. On 2026-11-16 that URL holds "Practical Loop Engineering:
Goals, loops, and the discipline of not delegating your judgment",
dated Aug 14, 2026. The essay dated June 7, 2026 is "Loop Engineering"
at https://addyosmani.com/blog/loop-engineering/. It defines: "Loop
engineering is replacing yourself as the person who prompts the agent",
quotes Peter Steinberger ("You should be designing loops that prompt
your agents"), and organizes a loop around automations, worktrees,
skills, plugins/connectors, and sub-agents ("keep the maker away from
the checker").

`the-architect` attributes four named parts to the essay: trigger,
topology, verifier, stop rule. As fetched, neither Osmani text uses
"topology", "stop rule" or "trigger" as a named part. Only "verifier"
and "stop condition" appear. The four-part version shows up in
secondary commentary (search results, not primary). Verdict: **not
found in the primary source**, with the fetch-tool caveat in §2. The
reading-list entry and `the-architect`'s "Grounded in real loop
engineering" section need a re-check against the full essay text.

## 7. The video

https://www.youtube.com/watch?v=paXHXZyRSsM. Via YouTube's oEmbed
endpoint: title "Meu workflow para desenvolver com IA: da ideia aos
agentes", channel **Attekita Dev** (https://www.youtube.com/@attekitadev).
Not accessible: description, chapters, upload date, transcript (the
page returned only footer content; a third-party transcript site
returned 403; a title search found nothing by this channel). Its content
and terms are **not found**, and nothing here maps them. Mapping needs
the transcript or the person's notes.

## 8. Confidence

- Confirmed (documented, several primary sources agree): agent,
  agentic loop, workflow patterns, subagent, context window,
  compaction, tool, skill, MCP, hook, plan mode, evals.
- Confirmed but vendor-specific: subagent, plan mode, hook (Claude
  Code); guardrails, handoff, manager agent (OpenAI); coordinator (ADK).
- Unsettled: harness / harness engineering (2025–2026, scope differs by
  source); spec-driven development (its author says so).
- Inference: every Free Wings → standard mapping in §4, and all
  Portuguese equivalents.
- Not found: standard terms for `deneir`, `writer`, blueprint,
  Foundation Sync; the video's content; Osmani's four named parts in
  his primary text.

## Sources

- [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents)
- [Effective context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [Multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system)
- [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)
- [SWE-bench post](https://www.anthropic.com/engineering/swe-bench-sonnet)
- [Demystifying evals](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)
- [Claude Code Glossary](https://code.claude.com/docs/en/glossary)
- [How Claude Code works](https://code.claude.com/docs/en/how-claude-code-works)
- [Extend Claude Code](https://code.claude.com/docs/en/features-overview)
- [Subagents](https://code.claude.com/docs/en/sub-agents)
- [OpenAI Agents SDK multi-agent](https://openai.github.io/openai-agents-python/multi_agent/)
- [OpenAI guardrails](https://openai.github.io/openai-agents-python/guardrails/)
- [MCP](https://modelcontextprotocol.io/docs/getting-started/intro)
- [Agent Skills](https://agentskills.io/home)
- [AGENTS.md](https://agents.md/)
- [Google ADK workflows](https://adk.dev/workflows/)
- [Böckeler SDD](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html)
- [Böckeler harness engineering](https://martinfowler.com/articles/exploring-gen-ai/harness-engineering.html)
- [Osmani harness](https://addyosmani.com/blog/agent-harness-engineering/)
- [Osmani Loop Engineering](https://addyosmani.com/blog/loop-engineering/)
- [Osmani Practical Loop Engineering](https://addyo.substack.com/p/practical-loop-engineering)
- [CoALA](https://arxiv.org/abs/2309.02427)
- [Video (oEmbed)](https://www.youtube.com/watch?v=paXHXZyRSsM)
- Secondary only, not relied on: [search results on loop engineering](https://www.sonarsource.com/blog/loop-engineering-without-verification-is-just-automation/), [harness engineering summaries](https://www.morphllm.com/harness-engineering)
