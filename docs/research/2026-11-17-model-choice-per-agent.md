# Model choice per agent — what the docs say, and a proposal

> Researched 2026-11-17 by `researcher`. Question: on a Claude **Pro**
> subscription (5-hour and weekly limits), which model and effort level
> should each Free Wings agent run on: the cheapest one that still does
> its job well? Primary sources only (Anthropic docs, help center,
> Anthropic blog). Verdicts: **documented** (quoted), **inference** (our
> reasoning), **not found** (looked for, not located).
>
> Builds on, does not repeat:
> [2026-11-11 — token efficiency](2026-11-11-token-efficiency-claude-code.md)
> (the `model`/`effort` fields, the order Claude Code uses to pick a
> subagent's model, cache TTL) and
> [2026-11-16 — anatomy](2026-11-16-anatomy-of-claude-code.md)
> (subagent vs main-thread agent, how to measure).

## 1. Current models in Claude Code

**Documented** (platform, models overview; prices per million tokens, input / output):

| | Fable 5.2 | Opus 5.5 | Sonnet 5.5 | Haiku 5.5 |
| :- | :- | :- | :- | :- |
| What Anthropic says it is for | "demanding reasoning and long-horizon agentic work" | "long-running agentic coding and knowledge work" | "The best combination of speed and intelligence" | "high-volume, latency-sensitive tasks such as classification, extraction, and routing" |
| Price | $8 / $40 | $4 / $20 | $2 / $10 | $0.10 / $0.50 (prompts ≤100K); $0.50 / $2.50 (over 100K) |
| Relative to Sonnet (inference, from price) | 4× | 2× | 1× | 1/20 (≤100K), 1/4 (over 100K) |
| Speed ("comparative latency") | Moderate | Moderate | Fast | Fastest |
| Context window / max output | 1M / 128K | 1M / 128K | 1M / 128K | 1M / 128K |
| Default effort (API) | `high` | `medium` | `high` | `medium` |

Cache reads: "0.05x the base input price" on Opus 5.5 and Sonnet 5.5,
0.1× on Haiku 5.5 (platform, pricing).

**Claude Code aliases — documented** (model-config): `opus` → Opus 5.5,
"for complex reasoning tasks"; `sonnet` → Sonnet 5.5, "for daily coding
tasks"; `haiku` → Haiku 5.5, "for simple tasks"; `fable` → Fable 5.2,
"for your hardest and longest-running tasks"; `best` = `opus`.
Default model: "Pro, Max, Ultra, Team, Enterprise, and Anthropic API …
defaults to Opus 5.5."

**Inference:** Haiku 5.5's price jumps 5× above 100K tokens of prompt.
A long Claude Code session (CLAUDE.md, files read, tool results) can
cross that line, so Haiku's advantage shrinks in exactly the long tasks.

## 2. Model choice and Pro limits

- **Documented** (help center, *How do usage and length limits work?*):
  "Your usage is affected by … which Claude model you're chatting with,
  and the effort level you've selected." "Higher effort uses more
  tokens." "Weekly limits apply across all models."
- **Documented** (help center, *Models, usage, and limits in Claude
  Code*): Opus "uses meaningfully more of your quota, so consider
  switching to Sonnet for routine work"; Opus costs "several times more
  per turn than Sonnet, and Sonnet more than Haiku." Sonnet "is the
  right choice for the large majority of coding work"; Haiku is "well
  suited to quick lookups, simple edits, or high-volume scripted runs."
- **Documented** (help center, *Claude Fable models on your plan*), Pro:
  "Fable models aren't included in your plan's usage limits"; they run
  on usage credits (paid).
- **Documented** (costs): the session and weekly limits are "shared
  across all models, so the developer can't restore access by switching
  models." Separate messages exist for "You've hit your Opus limit" and
  "Sonnet limit". After those, switching model family keeps you working.
- **Not found:** an exact Opus-to-Sonnet rate against Pro limits, or a
  separate Opus weekly cap on Pro. **Inference:** plan usage roughly
  follows token price (Opus ≈ 2× Sonnet per token), but Anthropic only
  says "several times more per turn". That wording may also cover Opus
  writing more tokens per turn.
- **Contradicted by an unverified source:** a search-result snippet said
  the help-center page names Sonnet as the default model. The page
  itself, as fetched, gives no default per plan, and model-config says
  Pro defaults to Opus 5.5. Trust model-config; check with `/model`.

## 3. Effort levels

**What they change — documented** (platform, effort): effort "affects
**all tokens** in the response", meaning text, tool calls and thinking.
"Lower effort also means fewer and terser tool calls." "Effort is a
behavioral signal, not a strict token budget." Thinking tokens are billed
as output (costs). Thinking cannot be turned off on Opus 5.5, Sonnet 5.5,
Haiku 5.5 or Fable (costs), so effort is the lever.

**Levels in Claude Code — documented** (model-config):
- `low`: "small change like a rename".
- `medium`: the default; "fits day-to-day engineering work with a clear
  scope".
- `high`: "verification matters or edge cases are likely, such as fixing
  a bug".
- `xhigh`: "deeper reasoning at higher token spend".
- `max`: "prone to overthinking".

Defaults: "Fable 5.2, Opus 5.5, Sonnet 5.5, and Haiku 5.5 default to
`medium`."

**Note — documented, two surfaces:** in Claude Code Sonnet 5.5 defaults
to `medium`; on the raw API it defaults to `high` (models overview).
Claude Code's own default is the one that applies here.

**Does a stronger model at lower effort beat a weaker model? —
documented, one measurement** (platform, *Optimizing for cost and
intelligence*):
- "On SWE-bench Pro, Claude Opus 5.5 at `low` scored 87.4% for $0.12
  per solved task, against Sonnet 5.5 at its default `high` at 87.6% for
  $0.11", described as "about the same" for about 7% more.
- Guidance: "sweep effort on your current model first … if the sweep
  shows a gap, price the stronger model alone at low effort."
- "Tuning effort is often a better lever than switching models"
  (choosing-a-model).

Blog (Lydia Hallie, Claude Code team, Jul 7 2026): "The model setting is
roughly *how capable*; the effort setting is roughly *how thorough*." A
larger model "can accomplish tasks the smaller one cannot even at the
highest effort settings."

**Not found:** any such comparison for teaching, prose, routing or web
research. The one number above is for coding.

## 4. Which tasks fit which model — documented guidance

- costs: "Sonnet handles most coding tasks well and costs less than
  Opus. Reserve Opus for complex architectural decisions or multi-step
  reasoning … For simple subagent tasks, specify `model: haiku`."
- choosing-a-model matrix:
  - Opus: "Complex agentic coding … large-scale refactoring".
  - Sonnet: "Code generation, data analysis, content creation …
    agentic tool use".
  - Haiku: "subagent tasks".
- platform, effort (Haiku 5.5): "In long agent prompts, the model is
  more likely to skip a search, stop early, or skip a check at `low`."
- Blog: "Pick a smaller model when the work is routine." "Larger models
  are also better at handling ambiguity." "If Claude has all the
  pertinent context and clearly tried and still got it wrong, that's a
  signal to pick a larger model."
- Delegation measured (optimizing page): "Claude Opus 5.5 with Claude
  Sonnet 5.5 subagents cost about a third less than one Claude Opus 5.5
  session … and scored about the same" (defect search). Haiku
  subagents under Opus `medium` "cut cost per attempt by 32% and the
  pass rate by about 4 percentage points". All figures are
  "Anthropic-internal and directional".

**Ways to mix models — documented:**
- `opusplan` (model-config): "uses `opus` during plan mode, then
  switches to `sonnet` for execution."
- Advisor tool (code.claude.com/docs/en/advisor, *experimental*): "Sonnet
  handles routine work and escalates planning, ambiguous failures, and
  completion checks to Opus." It is available on subscriptions; its
  tokens count toward plan limits. "Subagents inherit the configured
  advisor."
- **Not found:** whether every `opusplan` switch between Opus and Sonnet
  re-reads history without the cache, as a manual model switch does
  (2026-11-11, claim 10). **Inference:** likely, so use it for long plan
  and build phases, not for quick back-and-forth.

**Traps that matter here — documented:**
- With no `model` field, a subagent runs on the main conversation's
  model (sub-agents). So an Opus `the-architect` makes every delegated
  agent Opus.
- A `model` value Claude passes when it starts a subagent overrides the
  agent file's `model` (sub-agents, model order, step 1).
- Built-in Explore: "With a Claude subscription … it runs on the Opus
  model". To change that, define a project agent named `Explore` with
  `model: haiku`.
- Without an `effort` field, a subagent on a different model from the
  session gets that model's default; the session's level "doesn't apply"
  (sub-agents).
- **Not found (re-checked today):** whether `effort` in an agent file
  applies under `claude --agent`. Until confirmed, set effort for a chat
  session with `--effort` or `/effort`.

## 5. Proposal — inference, not a decision

Pro, Fable excluded (paid credits). "Subagent" = delegated by
`the-architect`; "chat" = `claude --agent <name>`.

| Agent | Subagent | Chat | Why | Quality risk / evidence |
| :- | :- | :- | :- | :- |
| `the-architect` | — (always main) | Opus, `medium` | Decides, explains trade-offs, owns Foundation Sync: "architectural decisions", "ambiguity" | Evidence is general. Haiku is listed for "routing", but this role does more than route. Test: Sonnet `medium` + `/advisor opus` |
| `programmer` | Sonnet, `medium` (rarely delegated: it needs the person) | Opus, `medium`; `low` for small pieces | Teaching = explaining with judgment in long back-and-forth; Opus `low` ≈ Sonnet `high` on coding cost | **Teaching quality per model: not found.** The person's intuition, not measured |
| `engenir` | Sonnet, `medium`; `high` for bug fixes | Sonnet, `medium` (or `opusplan`) | Closed spec = "clear scope"; "Sonnet handles most coding tasks" | Well supported. Escalate when `tester` keeps finding misses |
| `tester` | Sonnet, `high` | Sonnet, `high` | "verification matters or edge cases are likely" | Haiku could *run* a suite but *interpreting* failures needs reasoning. Medium evidence |
| `researcher` | Sonnet, `high` | Sonnet, `high` | Many fetches, judging sources; a careless miss defeats its purpose | Haiku "skip a search, stop early" risk; long context crosses Haiku's 100K price line. Thin evidence |
| `deneir` | Sonnet, `medium` | Sonnet, `medium` | Reads history, writes reflection: judgment, little code | Haiku `medium` worth a measured trial. Thin evidence |
| `writer` | Sonnet, `medium` | Sonnet, `medium` | "content creation" is listed under Sonnet | Bilingual prose quality per model: not found |

**Inference:**
- Setting `model` explicitly in every agent file is the single largest
  saving. Without it, Opus inheritance applies to all delegated agents
  (Chacal: `engenir` 66% + `tester` 24% of usage).
- `the-architect` should not pass a per-delegation `model` unless it
  means to, or it undoes the file.
- Haiku gets no agent yet. None of the seven is a "simple subagent
  task" by the docs' wording, so Haiku is for measured trials only.

**How to tell whether a cheaper model is good enough:**
1. **Baseline (one week, current setup):**
   - `/usage` plan breakdown, `w` view: share per subagent.
   - Status line `rate_limits.five_hour` / `seven_day.used_percentage`.
   - Session "Usage by model".
2. **Change one agent at a time.** Repeat on the same kind of task.
3. **Quality signals:**
   - failures `tester` catches in `engenir`'s work;
   - corrections the person has to make;
   - re-delegations;
   - for `researcher`, claims later found wrong.
   The docs' own rule: if it "clearly tried and still got it wrong," go
   up a model; if it "got it wrong by skipping a file," go up in effort.
4. **Caveats:**
   - `/usage` attribution is "approximate", from local history only.
   - Its cache line covers only the main conversation.
   - Per-agent OpenTelemetry needs `OTEL_LOG_TOOL_DETAILS=1`, or custom
     agents show as `"custom"` (2026-11-16, section 5).

## Sources checked (2026-11-17)

- https://code.claude.com/docs/en/model-config (both parts)
- https://code.claude.com/docs/en/costs
- https://code.claude.com/docs/en/sub-agents (first 100K characters)
- https://code.claude.com/docs/en/advisor
- https://platform.claude.com/docs/en/about-claude/models/overview
- https://platform.claude.com/docs/en/about-claude/pricing
- https://platform.claude.com/docs/en/build-with-claude/effort
- https://platform.claude.com/docs/en/about-claude/models/choosing-a-model
- https://platform.claude.com/docs/en/about-claude/models/optimizing-for-cost-and-intelligence (first 100K characters)
- https://claude.com/blog/claude-model-and-effort-level-in-claude-code (Jul 7, 2026)
- https://support.claude.com/en/articles/14552983-models-usage-and-limits-in-claude-code
- https://support.claude.com/en/articles/11647753-how-do-usage-and-length-limits-work
- https://support.claude.com/en/articles/8325606-what-is-the-pro-plan
- https://support.claude.com/en/articles/11145838-use-claude-code-with-your-pro-or-max-plan
- https://support.claude.com/en/articles/15424964-claude-fable-models-on-your-plan

Not read: model cards and announcement posts for the 5.5 models (the
docs above covered what each is for); the second half of the
optimizing page.

## Confidence

- **Confirmed:**
  - section 1;
  - section 2, except the exact Opus rate on Pro;
  - effort definitions and defaults;
  - the one Opus-low vs Sonnet-high coding measurement;
  - the guidance in section 4.
- **Inference:** the whole of section 5; Pro usage following token
  price; Haiku's price jump mattering in long sessions; the `opusplan`
  cache cost.
- **Not found:**
  - the exact Opus/Sonnet rate on Pro;
  - `effort` under `--agent`;
  - any quality comparison for teaching, prose or research across
    models.
- Docs change often; re-check before relying on version-gated
  behavior.
