# Cost tracking per agent per task in Claude Code — what each mechanism measures

> Researched 2026-11-23 by `researcher`. Question: how can a person using
> Claude Code measure the cost (tokens, ideally cache reads/writes and
> model) of each agent per task: the lead agent (main thread, opened with
> `claude --agent <name>`) and each delegated subagent (custom agents in
> `.claude/agents/*.md`, each with its own `model` and `effort`)? On a
> Pro/Max subscription, not only API billing. Primary sources first
> (code.claude.com/docs); secondary sources (GitHub issues, community
> tools) are marked as such. Verdicts: **documented** (quoted),
> **observed** (seen in this machine's own files, not in the docs),
> **inference** (our reasoning), **not found** (looked for, not located).
>
> Builds on, does not repeat:
> [2026-11-17 — model choice per agent](2026-11-17-model-choice-per-agent.md)
> (section 5, measurement method),
> [2026-11-16 — anatomy](2026-11-16-anatomy-of-claude-code.md)
> (section 5, instruments) and
> [2026-11-11 — token efficiency](2026-11-11-token-efficiency-claude-code.md).
> Open questions came from
> `docs/notes/the-architect/for-the-architect-2026-11-17-cost-tracking-research.md`.
> This file answers its research items 1, 2 and 4. Item 3 (S/M/L cut-offs)
> was not asked here and is untouched.

## 0. The finding that changes the baseline

**The "tokens" figure in a worker's report is not the worker's total
consumption. It is the size of its last request.**

- **Documented** (hooks, `PostToolUse` on the Agent tool, `tool_response`):
  `totalTokens` is the "Token count from the subagent's final API
  request: input, output, and cache tokens combined. This isn't a total
  across the whole run." `usage` is the "Per-type token breakdown of the
  final API request". The page adds: "for token and cost rollups across
  subagents, use the token and cost counters filtered to `query_source`
  `"subagent"`, since `totalTokens` and `usage` cover the final request
  only."
- **Documented** (monitoring-usage, `subagent_completed` event):
  `total_tokens` is "the token footprint of the subagent's final API
  request … roughly the subagent's context size at completion. Not a sum
  across the run."
- **Observed**, one run checked. The lead agent's transcript holds the
  notification the worker's report is built from:
  `<usage><subagent_tokens>28384</subagent_tokens><tool_uses>4</tool_uses><duration_ms>32760</duration_ms></usage>`.
  That is the `deneir` run in the 2026-11-17 baseline table ("~28k, 4
  tools, 0.5 min"). The same subagent's own transcript
  (`<session>/subagents/agent-<id>.jsonl`) has four distinct API
  responses (four distinct `message.id`; one appears on two lines with
  identical usage):

  | Response | input | cache write | cache read | output |
  | :- | -: | -: | -: | -: |
  | 1 | 2 | 15,593 | 0 | 242 |
  | 2 | 2 | 8,311 | 15,593 | 312 |
  | 3 | 2 | 746 | 23,904 | 3,056 |
  | 4 (last) | 2 | 3,135 | 24,650 | 597 |

  The last row sums to 2 + 3,135 + 24,650 + 597 = **28,384**, exactly
  the reported figure. The four rows summed are about **96,100 tokens**
  (8 input, 27,785 cache write, 64,147 cache read, 4,207 output), about
  3.4 times the reported number, and mostly cache reads. Limits of this
  check: one run; I counted by pattern search, not with a parser;
  whether the per-line `output_tokens` is final or a placeholder in the
  transcript is not documented (the SDK page says per-step
  `output_tokens` in the SDK message stream is a placeholder, see
  section 1.4).
- **Inference:** the baseline table in the 2026-11-17 note
  (~100k, ~190k, …) measures how large each worker's context was at its
  last request, and ranks workers by that. It is not a cost and not a
  per-worker consumption total. It is still useful as a "how big did the
  context get" signal. It must not be added up across tasks as spend, or
  used for a before/after comparison of model choice as if it were spend.

## 1. Mechanisms — what each one measures

### 1.1 Summary table

| Mechanism | Measures | Per agent? | Per model? | Cache split? | Reliability (per docs) | Setup | Pro/Max |
| :- | :- | :- | :- | :- | :- | :- | :- |
| **Subagent report** (the notification the lead sees) | Last request's input+cache+output, tool count, duration | One subagent run | No | No | Documented as final request only, not a run total | None | Yes (not billing-dependent; inference) |
| **`PostToolUse` hook on `Agent`** | Same final-request figures, as `totalTokens` and `usage`, plus `resolvedModel`, `modelsUsed` | One subagent run | Yes (`resolvedModel`) | Yes, final request only | Same as above. **No `tool_response` usage for background subagents** | Hook script | Yes (inference) |
| **`SubagentStop` / `Stop` hook** | **No token or cost field in the payload** | Yes (`agent_type`, `agent_id`) | `effort` yes; model not listed | No | n/a | Hook script | Yes (inference) |
| **Transcripts** (`~/.claude/projects/...`) | Every API request: usage with input, cache write, cache read, output, plus model (**observed**) | Yes: one file per subagent run; main thread in the session file | Yes (**observed**) | Yes | Format "is internal to Claude Code and changes between versions" (docs) | A script | Yes (inference) |
| **OpenTelemetry** | Counters per API request: tokens by `type`, cost, `model`, `query_source`, `effort`, `agent.name` | Yes, with caveats (see 1.3) | Yes | Yes | Documented schema; cost is an "approximation" | Env vars plus a collector/backend | **Not stated** |
| **Status line JSON** | Session cost estimate (includes subagents per docs wording), main-conversation tokens, cache split of the last call, `rate_limits` % | Only `agent.name` of the main thread; no subagent split | Current model only | Last call only | Cost is a client-side list-price estimate | Script in settings | Yes; `rate_limits` only on Pro/Max/Ultra |
| **`/usage` (`/cost`, `/stats`)** | Session tokens by model and estimated cost; on a plan, approximate attribution to skills, subagents, plugins, MCP servers (24 h or 7 d) | Approximate share per subagent | Session block: yes | `Prompt cache (main)` line: main only | "approximate and computed from local session history on this machine" | None | Yes; the dollar figure is "not relevant for billing" for subscribers |
| **`claude -p --output-format json`** | `total_cost_usd`, `modelUsage` (per model, incl. subagents) | No | Yes | Yes | "client-side estimates" | Only for non-interactive runs | Yes |

### 1.2 Hooks

**What the payloads contain — documented** (hooks page; raw text read for
SubagentStop, Stop and SessionEnd):
- Common fields: `session_id`, `transcript_path`, `cwd`, `prompt_id`,
  `scratchpad_dir`, `permission_mode`, `effort`, `hook_event_name`,
  `agent_id`, `agent_type` (the last two and the full list as given by
  the page's field table, via the fetch summary). `agent_type`: "Present
  when the session uses `--agent` or the hook fires inside a subagent.
  For subagents, the subagent's type takes precedence over the session's
  `--agent` value."
- `SubagentStop` adds `stop_hook_active`, `agent_id`, `agent_type`,
  `agent_transcript_path`, `last_assistant_message`, `background_tasks`,
  `session_crons`. "The `transcript_path` is the main session's
  transcript, while `agent_transcript_path` is the subagent's own
  transcript stored in a nested `subagents/` folder."
- `Stop` adds `stop_hook_active`, `last_assistant_message`,
  `background_tasks`, `session_crons`. It fires per finished turn, so it
  is not a "session total" event.
- `SessionEnd` adds `reason` (`clear`, `resume`, `logout`,
  `prompt_input_exit`, `other`). Default timeout **1.5 seconds**; raise
  with a per-hook `timeout` (budget up to 60 s) or
  `CLAUDE_CODE_SESSIONEND_HOOKS_TIMEOUT_MS`.
- **No token or cost field** appears in `Stop`, `SubagentStop` or
  `SessionEnd` input. The only usage-bearing hook payload found is the
  Agent tool's `PostToolUse` `tool_response` (section 0), and it covers
  the final request only. The only cost-like field found anywhere in hook
  input is `estimated_cache_write_usd`, in a model-switch payload
  (`context_tokens`, `cache_ttl`), which is not about agent cost.
- **Documented caveat on `transcript_path`:** "The transcript file is
  written asynchronously and may lag the in-memory conversation, so it
  may not yet include the current turn's most recent messages when a
  hook fires." **Inference:** a `Stop` hook that sums the main
  transcript can miss the last response of the turn; `SubagentStop`
  reading `agent_transcript_path` may have the same lag (the page states
  the lag for `transcript_path`, not for `agent_transcript_path`).
- Hooks run in subagents too ("Hooks from settings files, managed policy
  settings, and plugins also run inside subagents", 2026-11-11 claim 5),
  and a subagent's frontmatter `Stop` hook "is converted to
  `SubagentStop` at runtime" (sub-agents page). Agent-file `hooks`
  "fire … when the agent runs as the main session via `--agent`"
  (2026-11-11 claim 5).

**Setup cost:** a command hook plus a script (`jq` or Python). Hooks in
project `.claude/settings.json` run (the page excludes only the OTel
exporter variables from project settings, section 1.3).

**Pro/Max:** nothing in the hooks page ties them to billing mode.
**Inference:** they work the same on a subscription.

### 1.3 OpenTelemetry

**What it emits — documented** (monitoring-usage; the first 100,000
characters read as text, the second half searched, not read in full):
- Metric `claude_code.token.usage`, "Incremented after each API request",
  attribute `type`: `input`, `output`, `cacheRead`, `cacheCreation`.
  `"input"` "excludes tokens read from or written to the prompt cache".
  `cacheCreation` counts 5-minute and 1-hour writes together.
- Metric `claude_code.cost.usage` (USD), same attributes.
- Attributes on both: `model`, `query_source` (`main`, `subagent`,
  `auxiliary`, `plugin_model_complete`, `plugin_model_classify`),
  `effort`, `speed`, `agent.name`, `skill.name`, and others.
- Event `claude_code.api_request`, one per request: `model`, `cost_usd`,
  `cost_usd_micros`, `duration_ms`, `input_tokens`, `output_tokens`,
  `cache_read_tokens`, `cache_creation_tokens`,
  `cache_creation_5m_tokens` / `cache_creation_1h_tokens` (v2.1.300+),
  `request_id`, `query_source`, `effort`, `agent.name`. Events (never
  metrics) also carry `prompt.id`, `agent_id`, and `parent_agent_id`
  (v2.1.310+), "because they would cause unbounded cardinality".
- Event `claude_code.subagent_completed`: `agent_type`, `agent.source`
  (`built-in`, `plugin`, or the settings source of a custom agent),
  `total_tokens` (final request only), `total_tool_uses`, `duration_ms`,
  `model`, `final_model`, `model_swapped`. The page itself says: "for
  token or cost rollups, use the token counter and cost counter filtered
  to `query_source` `"subagent"`".

**How custom agents are labelled — documented:**
- `agent.name`: "Built-in agent names and agents from
  official-marketplace plugins appear verbatim. Other user-defined agent
  names are replaced with `"custom"`. Absent when the request was not
  issued by a named subagent type."
- `OTEL_LOG_TOOL_DETAILS=1` "also enables … real agent, skill, plugin,
  and MCP server and tool names on the cost and token counters". Before
  v2.1.273 the counters stayed redacted even with it set.
- The same variable "enable[s] logging of tool parameters and input
  arguments in tool events and trace span attributes". **Inference:**
  turning it on for names also sends tool arguments to your backend, so
  it matters where the backend is.
- Without it, the 2026-11-16 statement stands: every Free Wings agent
  would read `"custom"`, and the seven could not be told apart in the
  counters (`agent_id` in events would still separate runs, not names).

**What is not stated (found no text):**
- Whether a main thread started with `claude --agent <name>` carries
  `agent.name`, or which `query_source` it gets. The page never mentions
  `--agent`, and defines `agent.name` as "Subagent type that issued the
  request". **Inference:** the lead agent's own cost may land under
  `query_source: main` with no agent name, so the main thread of
  `the-architect` could be indistinguishable from a plain session except
  by `session.id`. Needs a test.
- Whether telemetry works on a Pro/Max login. The page "never says
  whether telemetry differs by login method"; it only mentions identity
  attributes "when authenticated". **Not found.** Needs a test.
- Where `console` exporter output goes in an interactive session. "Not
  stated". **Inference:** the TUI is likely to be disturbed; do not
  assume a zero-infrastructure console path works.
- A `task` attribute. None exists. Correlation attributes are `session.id`
  (replaced by `/clear`), `prompt.id` (events only, one user prompt),
  `agent_id`, `parent_agent_id`, `workflow.run_id`.

**Reliability:** "Cost metrics are approximations. For official billing
data, see Manage costs for your organization." Costs are list price
unless a managed `modelPricing` table is set (costs page).

**Setup cost:** highest of the group. Needs
`CLAUDE_CODE_ENABLE_TELEMETRY=1` plus exporter variables and a receiver
(a collector, Prometheus, or a backend). The page says to set them "in
managed settings, or have each developer set them in their shell or
`~/.claude/settings.json`"; they are ignored in a repository's
`.claude/settings.json` and `.claude/settings.local.json`, so the setup
is **not** something a project's committed files can switch on.

### 1.4 Transcripts

**What the docs say — documented** (sessions, claude-directory):
- "By default, Claude Code stores transcripts as JSONL at
  `~/.claude/projects/<project>/<session-id>.jsonl`".
- Subagent transcripts: `~/.claude/projects/{project}/{sessionId}/subagents/`,
  each stored as `agent-{agentId}.jsonl`, "in the same format as the
  session transcript", with `agent-{agentId}.meta.json` beside it
  (sub-agents page). They survive compaction of the main conversation.
- The transcript "can also hold what Claude Code sends the model": system
  prompt, tool definitions, request parameters, instruction files
  (claude-directory).
- **Stability warning, verbatim:** "The entry format is internal to Claude
  Code and changes between versions, so scripts that parse these files
  directly can break on any release. To build on session data, use
  `/export` or the script interfaces instead." The "script interfaces"
  named there are `claude -p --output-format json`, the `transcript_path`
  that hooks and status lines receive, and the Agent SDK.
- Retention: deleted after `cleanupPeriodDays`, **30 days by default**
  (minimum 1). Subagent transcripts go with their parent session.
- `~/.claude/stats-cache.json`: "Aggregated token and cost counts shown
  by `/usage`" (claude-directory). Its format is not described.

**The usage fields — not documented; observed on this machine:**
- Assistant lines in the main and subagent files carry
  `"usage":{"input_tokens":…,"cache_creation_input_tokens":…,"cache_read_input_tokens":…,"output_tokens":…}`,
  and the model name (`claude-sonnet-5-5` on the subagent lines of the
  run in section 0).
- One API response is written on several lines (one per content block);
  the usage is identical on those lines and the `message.id` is the same.
  Summing every line overcounts. The Agent SDK page gives the rule:
  "When Claude uses multiple tools in one turn, all messages in that turn
  share the same ID, so deduplicate by ID to avoid double-counting."
- **Placeholder warning, documented for the SDK stream, not verified for
  the transcript file:** "the message's `output_tokens` is only the count
  the API had reported at `message_start`", and "per-step `output_tokens`
  is a placeholder", with the real output count read from the result
  message. If the transcript has the same property, summing
  transcript `output_tokens` undercounts output; input and cache fields
  are "accurate". Output tokens are the expensive and, for thinking,
  effort-dependent part, so this matters for comparing effort levels.
  **Not found:** whether transcript lines are patched with the final
  count. In the run checked, the per-response outputs (242, 312, 3,056,
  597) are not obviously placeholders, but that is not proof.
- `meta.json`, **observed** (undocumented): contains `agentType` with the
  real custom name (`"deneir"`, not `"custom"`), `description`,
  `toolUseId`, `spawnDepth`, `finished`, and a field `pickedEffortModel`.
  In the run checked, `pickedEffortModel` read `claude-opus-5-5` while
  the transcript's assistant lines read `claude-sonnet-5-5`. Not
  explained; the transcript's per-line model is the better evidence of
  what served the request, and the discrepancy should be understood
  before any script relies on `meta.json` for the model.
- The main transcript starts with an entry of type `agent-setting`
  naming the `--agent` agent (`"agentSetting":"the-architect"`),
  **observed**, undocumented. It lets a parser attribute a main-thread
  session to its agent without OTel.
- **Secondary source, a GitHub issue** (anthropics/claude-code #23254,
  Feb 2026, closed "not planned" for inactivity, no fix shown): reported
  that subagent usage was not saved in session JSONL files and that
  session files held "~50-70% of actual usage when subagents are used".
  Both the current docs (separate `subagents/` transcripts) and the files
  on this machine show subagent usage now exists on disk, so the issue
  describes an older state. Treat it as a reason to check, not as
  current behavior.
- **Secondary, community tools** (not checked beyond search snippets):
  `ccusage` (a pull request #735 proposing a `--breakdown-subagents`
  flag; merge not confirmed), `ccost` (per-subagent breakdown), and
  others parse these files. They show the approach is common; they are
  not evidence of correctness and they inherit the format-stability risk.

**Setup cost:** a script. No collector. Works after the fact, as long as
the 30-day window has not passed.

**Pro/Max:** the files are written regardless of login (inference; the
docs tie transcripts to nothing billing-related).

### 1.5 `/usage`, `/cost`, `/stats`

**Documented** (costs):
- `/cost` and `/stats` are aliases of `/usage` (2026-11-16, section 5).
- Session block: `Estimated cost`, durations, `Usage by model` (input,
  output, cache read, cache write, per model). "Claude Code computes the
  dollar figure locally from token counts at list price". The note:
  "The Session block in `/usage` shows API token usage and is intended
  for API users. Claude Pro, Max, and Ultra subscribers have usage
  included in their subscription, so the session cost figure isn't
  relevant for billing purposes." **Inference:** on Pro the dollars are a
  list-price equivalent, useful as a common unit; the plan limits are
  what is actually spent.
- Totals "reset when `/clear` starts a new session" (since v2.1.211).
- `Prompt cache (main)` line: "covers the main conversation only, not
  subagents."
- Plan breakdown (Pro/Max/Ultra/Team/Enterprise): attribution "to skills,
  subagents, plugins, and individual MCP servers", as percentages, for
  the last 24 hours or 7 days. "The figures are approximate and computed
  from local session history on this machine, so usage from other
  devices or claude.ai is not included."
- Whether the Session block's `Usage by model` includes subagent
  requests: **not stated** in the costs page. The SDK page says
  `total_cost_usd` and `modelUsage` include subagents (section 1.8).
- Not scriptable as far as the docs show (no machine-readable output
  described): **not found.**
- How `/usage` attributes a session opened with `--agent`: still **not
  found** (open since 2026-11-11).

### 1.6 Status line

**Documented** (statusline; raw text read):
- Input JSON includes `cost.total_cost_usd` ("Estimated session cost in
  USD, computed client-side at list price … May differ from your actual
  bill. Resets to $0 when `/clear` starts a new session"),
  `cost.total_duration_ms`, `cost.total_api_duration_ms`,
  `context_window.total_input_tokens` / `total_output_tokens` (what is in
  the context window now, "from the most recent API response"),
  `context_window.current_usage` (input, output, cache creation, cache
  read of the last API call), `model.id`, `effort.level` (the session's,
  "not one that a skill, a subagent, or `ultrathink` set"),
  `session_id`, `prompt_id`, `transcript_path`, `agent.name` ("appears
  only when running with the `--agent` flag or agent settings
  configured"), and `prompt_cache` (main conversation only).
- `rate_limits.five_hour.used_percentage` and `seven_day.used_percentage`
  (0 to 100) with `resets_at`: "appears only for claude.ai Pro, Max, and
  Ultra subscribers", and only after the first API response.
- "The status line runs locally and does not consume API tokens."
  Updates are event-driven, debounced at 300 ms; `refreshInterval`
  (minimum 1 second) re-runs it on a timer.
- Does `cost.total_cost_usd` include subagents? The costs page says the
  field "accumulates the estimated cost of all API calls in the current
  session" (statusline page, cost section). **Secondary source**
  (anthropics/claude-code #32617, v2.1.71, Mar 2026, closed as duplicate):
  the reporter says `cost.total_cost_usd` "correctly aggregates across
  subagents" while `context_window` token counts do not. Consistent with
  the docs wording; not independently tested.
- It gives per-session totals for the whole tree, **not** per agent, and
  no per-subagent breakdown.
- Plan impact: `rate_limits` percentages are the only plan-native
  measure in the group. **Inference:** they cover the whole account
  (other sessions, claude.ai), so a delta over a task is noisy if
  anything else ran; the docs do not state the percentage's granularity.

### 1.7 The subagent's returned report

Covered in section 0. In short: free, always present, documented (as hook
and OTel fields) to be the final request only. **Observed** (undocumented
shape): the lead agent's transcript records
`<usage><subagent_tokens>…</subagent_tokens><tool_uses>…</tool_uses><duration_ms>…</duration_ms></usage>`
in a `task-notification`, with the same three numbers also stored as
`usage:{totalTokens,toolUses,durationMs}`. That the notification's
`subagent_tokens` equals the documented `totalTokens` is **observed on
one run** (28,384), not documented. This closes the 2026-11-16 "not
found" about the trailer's contents, for this version.
**Tool count and duration are whole-run figures** (documented:
`totalToolUseCount` "Count of tool calls the subagent made"; `duration`
"Wall-clock duration of the subagent run"), so they remain reliable.

### 1.8 Other documented mechanisms

- **`claude -p … --output-format json`** (headless; Agent SDK cost
  tracking): `total_cost_usd` and `modelUsage` (per model: input, output,
  cache read, cache creation, `costUSD`) include subagent requests; the
  `usage` field "excludes" them ("Use `modelUsage` … for whole-tree token
  accounting; the `usage` field undercounts as soon as nesting occurs").
  "Both figures are client-side estimates." Relevant only if a task runs
  non-interactively. Whether `--agent` combines with `-p` for this
  purpose: not checked.
- **Claude Code Analytics API / Console dashboard / Enterprise Analytics
  API / Teams spend report CSV** (costs): API-billing and Team/Enterprise
  channels; per user and per model, not per agent. Not applicable to an
  individual Pro plan. **Inference:** out of scope here.
- **`/insights`**: analyses recent sessions and writes an HTML report. "The
  tokens count against your plan or API usage." Not a measurement tool for
  this purpose, and it costs tokens (section 3).
- **`--max-budget-usd`** (headless, cli-reference): a cap, not a record.
- **Agent SDK** result and per-step usage: the programmatic form of the
  transcript data (section 1.4); deduplicate by message ID; read output
  from the result message.

## 2. Where a record can live — options, no decision

"Task" has no native meaning in Claude Code. Correlation handles that
exist: `session_id`, `prompt_id` (one user prompt), `agent_id` (one
subagent run), the time window, and the spec file the person is
working from. **Inference:** a per-task record needs one explicit link,
for example a note of the `session_id` and time range in the spec, or a
convention that a spec corresponds to one session (`/clear` between
tasks, which also resets the `/usage` and status line totals).

| Option | How it works | Gives | Cost and trade-offs |
| :- | :- | :- | :- |
| **A. Spec run record only** | Lead agent copies numbers (report, `/usage`) into the template's run record | Per task, readable | Not summable; today's "tokens" is the final-request footprint (section 0), so it needs a different source of numbers |
| **B. Hook appends to a file** | `SubagentStop` (and for the main thread `SessionEnd` or `Stop`) runs a script that reads the transcript path from the payload, sums deduplicated usage by model, appends one JSON line (agent, agent_id, session_id, prompt_id, model, effort, four token fields, time) to a log | Per agent run, per model, cache split, summable with `jq` | Depends on the internal transcript format; `SessionEnd` has a 1.5 s default timeout; transcript "may lag", so a `Stop` read can miss the last response; a `SubagentStop` read may have the same lag (not stated); `async: true` avoids blocking; the log can sit in the repo (gitignored or not) or outside it. Hooks can be defined once in settings or per agent in frontmatter |
| **C. Parse transcripts after the fact** | A script walks `~/.claude/projects/<project>/<session>/` (main file plus `subagents/`) when a spec closes, or on demand | Same data as B, no hook running during work, complete (no lag problem) | Same format dependency; must run within `cleanupPeriodDays` (30 days default) or the data is gone; mapping files to tasks is manual unless the spec records the `session_id`; `meta.json` and `agent-setting` lines are undocumented |
| **D. OpenTelemetry to a collector/backend** | Documented counters and events, stored by your own stack | Documented schema; per request; per model; cache split; stable across versions in principle | Heaviest setup; cannot be switched on from the committed project files; `OTEL_LOG_TOOL_DETAILS=1` needed to name custom agents and it also logs tool arguments; main-thread labelling under `--agent` not stated; subscription behavior not stated; costs are list-price approximations |
| **E. Status-line logger** | The status line script appends `cost.total_cost_usd`, `rate_limits`, `session_id`, `agent.name`, model, effort at intervals | Whole-session cost including subagents (per docs wording); plan-limit percentage over time | No per-subagent split; totals reset on `/clear`; runs only on events or `refreshInterval`; the plan percentage is account-wide |
| **F. Community tools** (`ccusage` and similar) | Parse the same files | Ready-made reports, some with a subagent breakdown | Secondary and unaudited here; same format dependency; maintenance is someone else's |

Mixed designs are possible (for example E for plan-limit deltas plus B
or C for per-agent tokens). Choosing is the person's decision.

## 3. Is it cheap enough

**Mechanisms that add nothing to the session's tokens (out-of-band):**
- **Status line:** "runs locally and does not consume API tokens."
  (documented).
- **Command hooks that exit 0 with no stdout:** "Exit 0 with no output is
  a success that adds nothing to context." Plain stdout is added as
  context only for `UserPromptSubmit`, `UserPromptExpansion`,
  `SessionStart` and `PostModelSwitch`; "Claude never sees stderr from a
  hook that exits 0". `SubagentStop`, `Stop` and `SessionEnd` are not on
  the stdout-injection list (documented, hooks page).
- **OpenTelemetry export:** out-of-band by design (it exports from the
  Claude Code process). The documents do not discuss its own token cost;
  there is no model call involved. **Inference.**
- **Transcript parsing and `/export`:** local file reads.
- **The subagent's report:** already in the lead's context, free.

**Ways a measurement can cost tokens:**
- `additionalContext` (any hook): "Claude Code wraps the string in a
  system reminder and inserts it into the conversation". Keep it out of
  the measurement hook. Stop and SubagentStop hooks that block or add
  context also keep the agent running ("8-consecutive-continuation cap").
- Plain stdout from a `SessionStart` or `UserPromptSubmit` measurement
  hook.
- `/insights`: "the tokens count against your plan or API usage."
- Asking an agent to read a transcript or copy numbers by hand: ordinary
  tokens, plus the transcript can be large. **Inference.**
- Whether `/usage` itself triggers a model request: not stated. It
  fetches plan limits from an endpoint; the costs page does not describe
  it as using tokens. **Not found** either way.

**Cheapest reliable option, as a proposal (inference, not a decision):**
out-of-band transcript summing, either triggered by a hook (B) or run
after the fact (C). It is the only mechanism found that gives, with no
extra infrastructure, per-request input, cache write, cache read and
output plus model, for each subagent run and for the main thread.
Reliability caveats to carry with it:
1. The transcript format is documented as internal and unstable; a
   parser should be checked after each Claude Code upgrade (a one-run
   reconciliation like the one in section 0 is the test).
2. Deduplicate by `message.id`.
3. Output tokens may be placeholders in the file (not verified); until
   checked, treat output as a lower bound, and treat input and cache
   fields as reliable (SDK page: "accurate for input and cache tokens").
4. Record the dollar figure as a "list-price equivalent", not spend, on
   Pro.
5. Pair it with the status line's `rate_limits` if the question is about
   the Pro limits themselves.

If the person prefers a documented, versioned schema over zero
infrastructure, D is the alternative. Two tests would be needed before
trusting it here: whether `--agent` main threads get an `agent.name`, and
whether telemetry runs on a Pro login. Neither is answered in the docs.

## 4. Corrections to what the earlier files say

- **2026-11-17 handoff note, "What is known":** "A worker's report gives
  tokens, tools used and time: reliable, per worker." Corrected: tool
  count and time are whole-run; **tokens are the final request's size**
  (section 0). The baseline table measures context size at completion.
- **2026-11-16 anatomy, section 5, "Not found: what the subagent's
  'small metadata trailer' contains":** now found for this version
  (section 1.7), with the shape observed, not documented.
- **2026-11-16 anatomy, OTel row:** still correct. Added now:
  `OTEL_LOG_TOOL_DETAILS` also exposes tool arguments; the counters
  ignored it before v2.1.273; `--agent` main-thread labelling is not
  stated.
- **2026-11-17 model-choice, section 5, baseline step "Status line
  `rate_limits`":** still valid as an account-wide, coarse measure
  (section 1.6). "Session 'Usage by model'" does not split by agent.
- **Still open from earlier notes:** how `/usage` attributes a
  `--agent` session; `effort` in an agent file under `--agent`.

## Sources checked (2026-11-23)

Primary, code.claude.com:
- https://code.claude.com/docs/en/monitoring-usage (first 100,000
  characters read through the fetch summary; characters 100,000 onward
  searched and read around the `subagent_completed` event, cost
  monitoring and security sections, not read in full)
- https://code.claude.com/docs/en/hooks (characters 0 to about 290,000
  through the fetch summary; SubagentStop, SessionEnd and Notification
  sections read as raw text; the `PostToolUse` Agent `tool_response`
  table and the common-fields table come from the fetch summary, not raw
  text)
- https://code.claude.com/docs/en/statusline (raw text read for the field
  table, JSON example and prompt-cache section)
- https://code.claude.com/docs/en/costs (read in full)
- https://code.claude.com/docs/en/sessions (read in full)
- https://code.claude.com/docs/en/claude-directory (read in full)
- https://code.claude.com/docs/en/headless (read in full)
- https://code.claude.com/docs/en/agent-sdk/cost-tracking (read in full)
- https://code.claude.com/docs/en/sub-agents (first 100,000 characters,
  through the fetch summary; transcripts, `--agent`, SubagentStop)

Secondary (marked as such in the text):
- https://github.com/anthropics/claude-code/issues/32617 (statusline
  token counts vs cost with subagents; closed as duplicate)
- https://github.com/anthropics/claude-code/issues/23254 (subagent usage
  not in session JSONL; closed not planned)
- Web-search snippets only, pages not opened: ccusage pull request #735,
  `ccost`, community transcript-format notes
  (https://evilpiepirate.org/forge/kent/consciousness/raw/branch/master/doc/claude-code-transcript-format.md,
  https://cdn.jsdelivr.net/npm/oh-my-opencode@4.18.2/dist/skills/coding-agent-sessions/references/claude.md,
  https://frederick-douglas-pearce.github.io/blog/2026/inside-the-subagent-trace-file/).
  Nothing in this report depends on them.

Observed locally (not a published source): the lead session transcript,
one subagent transcript and its `meta.json`, in
`~/.claude/projects/-Users-samrfreitas-Lab-free-wings/`. Field names and
numbers were read with pattern search; no prompt or file content was
copied into this report.

Not read: the hooks page `PermissionRequest` and other event sections
beyond Stop, SubagentStop, SessionEnd and PostToolUse; the monitoring page
traces (beta) section; `settings-reference` (`modelPricing`, `env`);
`env-vars`; the Agent SDK TypeScript `SDKResultMessage` reference;
`/usage` source behavior (no official text on whether it spends tokens).

## Confidence

- **Confirmed (documented, quoted):**
  - the `totalTokens`, `usage` and `total_tokens` fields cover the final
    request only (section 0);
  - hook payload fields for `Stop`, `SubagentStop`, `SessionEnd`, and the
    absence of token or cost fields in them;
  - the OTel counters, attributes, `agent.name` redaction rule and
    `OTEL_LOG_TOOL_DETAILS` behavior;
  - transcript and subagent-transcript locations, 30-day retention, and
    the "internal format" warning;
  - the status line fields, and that it consumes no API tokens;
  - the `/usage` approximations;
  - the `rate_limits` condition (Pro/Max/Ultra only);
  - the dedupe-by-message-ID rule and the `output_tokens` placeholder
    behavior in the SDK stream.
- **Observed once, not documented:** the notification's
  `subagent_tokens` equals the last request's size (28,384 on one run);
  the run's real total was about 96,100; usage and model on transcript
  lines; `meta.json` fields; the `agent-setting` entry.
- **Inference:** that the baseline table is a context-size ranking, not a
  spend record; that hooks, transcripts and the status line behave the
  same on a subscription; that a `Stop` hook may miss the last response;
  that `OTEL_LOG_TOOL_DETAILS=1` sends tool arguments to the backend; the
  cheapest-option proposal; the "one spec = one session" convention.
- **Not found:**
  - OTel behavior on Pro/Max;
  - OTel labelling of a `--agent` main thread;
  - where the OTel console exporter writes in interactive mode;
  - whether transcript `output_tokens` is final or a placeholder;
  - whether `agent_transcript_path` lags like `transcript_path`;
  - how `/usage` attributes a `--agent` session;
  - whether `/usage` spends tokens;
  - a documented per-subagent total anywhere except the transcripts and
    the OTel counters filtered by `query_source`;
  - the cause of the `pickedEffortModel` vs transcript model difference.
- Docs change often (version notes in the pages go up to about v2.1.327);
  the transcript format changes between versions by the docs' own
  statement. Re-check before building a parser.
