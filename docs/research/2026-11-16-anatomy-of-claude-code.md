# Anatomy of Claude Code — how one session actually works

> Researched 2026-11-16 by `researcher`. Purpose: understand how Claude
> Code itself works before changing the harness, because its mechanics
> decide what every agent and skill costs on a Pro subscription with
> 5-hour and weekly limits. Written for a learner: each piece is
> explained plainly, then cited. Primary sources only. Verdicts:
> **documented** (quoted from the source), **inference** (our
> reasoning, not stated by a source), **not found** (looked for, not
> located).
>
> Builds on, does not repeat:
> [2026-11-11 — token efficiency](2026-11-11-token-efficiency-claude-code.md)
> (model/effort fields, cache TTL, `omitClaudeMd`, hooks, `/clear` vs
> `/compact`, delegation guidance).

## Correction to the 2026-11-11 file

That file's assessment said, as **inference**, that `engenir` and
`tester` "are opened with `claude --agent`, so they are main-thread
sessions." That was wrong for Chacal: there, `the-architect` ran as the
main session and **delegated** `engenir` and `tester` as subagents.
Consequences (**inference**, from the documented rules in that file and
in section 3 below):

- The subagent-only levers *do* apply to them: `omitClaudeMd`,
  `autoCompactWindow`, `effort` in frontmatter, and the subagent cache
  bucket ("five minutes even on a subscription").
- The open question "how `/usage` attributes a session opened with
  `--agent`" does not bear on the Chacal 66% / 24% figures: `/usage`
  attributes "recent usage … to skills, subagents, plugins, and
  individual MCP servers" (costs), so those were subagent shares.

The 2026-11-11 file is left unchanged; this note is the correction.

## Anatomy at a glance

    ┌─────────────────────────── ONE API REQUEST (re-sent every step) ───────────────────────────┐
    │ 1. System prompt layer  — core instructions + tool definitions                             │
    │      (an `--agent` body REPLACES the default instructions here)                            │
    │      MCP tools: names only, full schemas deferred (tool search)                            │
    │ 2. Project context      — CLAUDE.md, auto memory (first 200 lines / 25KB), unscoped rules  │
    │ 3. Conversation         — your messages, Claude's replies, every tool result,              │
    │                           invoked skill bodies, subagent results, system reminders         │
    └────────────────────────────────────────────────────────────────────────────────────────────┘
          │ model replies with tool_use ──► Claude Code runs the tool ──► tool_result appended ─┐
          └──────────────────────────────── next request (whole thing again) ◄──────────────────┘

| Piece | Enters context | Paid on | Survives compaction |
| :- | :- | :- | :- |
| System prompt + built-in tool defs | Session start | Every request (cached) | Yes |
| CLAUDE.md, auto memory | Session start | Every request (cached) | Re-injected from disk |
| Skill descriptions | Session start | Every request | Rebuilt and re-injected |
| Skill body | When invoked | Every later request | Re-injected, max 5k tokens per skill / 25k total |
| Subagent descriptions | Session start | Every request (**inference**: as part of the context) | — |
| MCP tool names + server instructions | Session start | Every request | — |
| MCP full tool schema | When Claude searches for it | Every later request | — |
| Hook | Never, unless it returns output | Only its output | Summarized |
| File read / command output | When the tool runs | Every later request | Up to 5 recent files re-read |
| Subagent work | Its own window | Its own requests | Only its final result is in yours |

## 1. The agent loop

**What it is, plainly.** The model cannot do anything by itself; it
can only write text. When it wants to read a file or run a command, it
writes a structured request ("call tool X with these arguments") and
stops. Claude Code runs the tool, appends the result to the
conversation, and calls the model *again* with everything so far. That
cycle repeats until the model answers without asking for a tool. One
"turn" from your point of view is therefore many API requests.

- **Documented** (platform, how-tool-use-works): "The model never runs
  anything on its own. It emits a structured request, your code (or
  Anthropic's servers) runs the operation, and the result flows back
  into the conversation." The loop: "Send a request with your `tools`
  array and the user message. Claude responds with `stop_reason:
  "tool_use"` … Run each tool … Send a new request containing the
  original messages, the assistant's response, and a user message with
  the `tool_result` blocks. Repeat … while `stop_reason` is
  `"tool_use"`."
- **Documented** (how-claude-code-works): "Claude Code is the layer
  around the model that provides the tools and manages the context the
  model sees. This surrounding layer is what the term agentic harness
  refers to." "Each tool use gives Claude new information that informs
  the next step."
- **What is re-sent — documented** (prompt-caching): "The model doesn't
  remember anything between requests, so Claude Code re-sends the full
  context: the system prompt, your project context, every prior message
  and tool result, and your new message." Caching does not shrink this;
  it makes the unchanged prefix cheaper: "Cached prompt prefixes still
  occupy the context window: prompt caching changes what you pay for
  those tokens, not whether they count" (platform, context-windows).
- **Documented** (costs): "each time Claude uses tools it sends another
  request carrying that batch of tool results."
- **Documented** (platform, context-windows): thinking tokens "are
  billed as output tokens," and on Opus 4.5+ / Sonnet 4.6+ / Haiku 5.5
  "The API keeps previous thinking blocks by default … those blocks
  count toward the context window like any other input tokens."
- **Inference:** a 10-tool-call turn costs roughly 10 requests, each
  carrying the whole conversation. A tool that returns a lot of text
  makes *every later request* bigger, not just one.

Sources: https://platform.claude.com/docs/en/agents-and-tools/tool-use/how-tool-use-works ·
https://code.claude.com/docs/en/how-claude-code-works ·
https://code.claude.com/docs/en/prompt-caching

## 2. The context window

**What it is, plainly.** The context window is the model's working
memory for one request: everything it can see at once. It is not
training data, and it is not memory across sessions. Everything in the
request counts toward it.

- **Documented** (platform, context-windows): "Everything in the
  request counts toward the context window: the system prompt, every
  message in `messages` (including tool results, images, and
  documents), and your tool definitions. The output Claude generates
  for the turn, including its extended thinking, counts too."

**What fills it, in order — documented** (prompt-caching, layer table):

| Layer | Content | Changes when |
| - | - | - |
| System prompt | Core instructions, tool definitions | The set of loaded tool definitions changes |
| Project context | CLAUDE.md, auto memory, unscoped rules | Session starts, or after `/clear` or `/compact` |
| Conversation | Your messages, Claude's responses, tool results | Every turn |

Further detail:

- **Before you type** (context-window): "CLAUDE.md, auto memory, MCP
  tool names, and skill descriptions all load into context." Plus,
  optionally, an output style or `--append-system-prompt` text.
- **Auto memory** (how-claude-code-works): "The first 200 lines or
  25KB of MEMORY.md, whichever comes first, load at the start of each
  session."
- **CLAUDE.md arrives as a message, not in the system prompt**
  (how-claude-code-works): "Claude Code adds its own context to the
  conversation alongside your messages: Your CLAUDE.md files …"; and
  (sub-agents) "CLAUDE.md files and project memory still load through
  the normal message flow."
- **Working directory, platform, git** (prompt-caching): "the
  conversation opens with an announcement of the working directory,
  platform, shell, and OS version"; each conversation "also carries
  the branch and recent commits from that snapshot."
- **Skills** append "as user messages at the point of invocation"
  (prompt-caching). The skill listing truncates each skill's
  description at 1,536 characters (skills).
- **Subagent descriptions** (sub-agents): "Those descriptions take up
  context, so keep them short."
- **Not found:** the exact position of the skill listing and of the
  subagent descriptions inside the three layers. **Inference:** the
  subagent descriptions travel with the `Agent` tool's definition in
  the system-prompt layer. Check with `/context`.

**Size — documented** (platform, context-windows): "Claude … Opus 5.5
… Sonnet 5.5 … Haiku 5.5 … have a 1M-token context window … For every
model with a 1M-token context window, 1M is the default." In Claude
Code (model-config): "Fable models, Sonnet 5 and later, Haiku 5.5, and
Opus 4.7 and later run with the 1M window by default, with no `[1m]`
suffix needed." The plan table that restricts 1M on Pro covers only
Opus 4.6 and Sonnet 4.6 ("Pro: Requires usage credits"). **Inference:**
on Pro with Opus 5.5 or Sonnet 5.5, the window is 1M.

**Bigger is not better — documented** (platform, context-windows): "As
token count grows, accuracy and recall degrade, a phenomenon known as
*context rot*." Engineering blog (Sep 29, 2025): the goal is "finding
the smallest possible set of high-signal tokens that maximize the
likelihood of some desired outcome."

**When it fills — documented** (model-config): "Models running with a
native 1M window compact before the window fills, at about 967K tokens
by default." You can compact earlier with `/autocompact 200k`,
`autoCompactWindow`, or `CLAUDE_CODE_AUTO_COMPACT_WINDOW`; to turn off
1M, use `CLAUDE_CODE_DISABLE_1M_CONTEXT=1`. Compaction "summarizes the
conversation history … The summary keeps your requests and key code
snippets, but detailed instructions from early in the conversation may
be lost" (how-claude-code-works). What comes back after compaction
(context-window):

- CLAUDE.md and auto memory: re-injected from disk.
- Skill descriptions: rebuilt and re-injected.
- Files: "re-reads up to five, most recently modified first"; a file
  over 5,000 tokens comes back as a path only.
- Invoked skill bodies: "capped at 5,000 tokens per skill and 25,000
  tokens total."
- "Path-scoped rules and nested CLAUDE.md files … compaction
  summarizes them away."

The cost of `/compact` itself is in the 2026-11-11 file, claim 7.

Sources: https://platform.claude.com/docs/en/build-with-claude/context-windows ·
https://code.claude.com/docs/en/context-window ·
https://code.claude.com/docs/en/model-config ·
https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents

## 3. How an agent changes a conversation

**Plainly.** An agent file can be used in four ways. They differ in
what the agent starts with, what comes back to you, whose cache it
uses, and whether you can talk to it.

| | Main-thread agent (`--agent` / `"agent"`) | Subagent | Fork | Agent-team teammate |
| :- | :- | :- | :- | :- |
| Starts with | Agent body *replacing* the default system prompt, plus CLAUDE.md and memory | Fresh window: own prompt, CLAUDE.md (unless `omitClaudeMd`), git status, delegation message | The parent's whole conversation, system prompt, tools, model | CLAUDE.md, MCP, skills, plus the spawn prompt; no lead history |
| Sends back | — (it *is* the conversation) | Final result only | Final result only | Messages and an idle notification with its final answer |
| Cache | Main bucket (1h on subscription) | Own cache, 5-min bucket | Reads the parent's cache; 5-min bucket | Own cache; 5-min bucket (in-process) |
| Can ask you? | **Inference:** yes | No (`AskUserQuestion` removed) | **Not found** | **Not found** whether it can use `AskUserQuestion`; you can message it directly |
| Status | Stable | Stable | On by default (v2.1.302+) | Experimental, off by default |

**Main-thread agent — documented** (sub-agents): "Pass `--agent <name>`
to start a session where the main thread itself takes on that
subagent's tool restrictions and model." "Unless the agent's prompt is
empty, a custom subagent's system prompt replaces the default Claude
Code system prompt entirely … `CLAUDE.md` files and project memory
still load." Fields that do nothing in this mode: `omitClaudeMd`
("Ignored when the agent runs as the main session agent") and
`autoCompactWindow` ("the field has no effect there").

**Inference:** it can ask you questions, because it is the main
conversation. The docs remove `AskUserQuestion` only "from every
subagent."

**Subagent — documented** (sub-agents): "Each subagent starts with a
fresh, isolated context window. It doesn't see your conversation
history, the skills you've already invoked, or the files Claude has
already read. Claude composes a delegation message that summarizes the
task, and the subagent works from there." From tools-reference: "The
parent doesn't see the subagent's intermediate tool calls or outputs,
only that final result." And (context-window): "Only the summary and a
small metadata trailer come back."

- **Tools removed from every subagent:** `AskUserQuestion`,
  `EnterPlanMode`, `ExitPlanMode` (unless `permissionMode: plan`),
  `EndConversation`, `ScheduleWakeup`, `WaitForMcpServers`, `Workflow`,
  and `Agent` at the depth limit.
- **Permissions:** foreground "Permission prompts are passed through
  to you"; background "surfaces the prompt in your main session and
  names the subagent."
- **You can still steer it — documented:** "With a fork's or
  subagent's transcript open, follow-up messages and skills go to that
  agent."
- **Default mode:** "With nothing configured, a subagent that Claude
  spawns from your main conversation starts in the background."
- **Nesting:** "a subagent can spawn subagents of its own, up to three
  layers below the main conversation."
- **Cost:** "every subagent … sends its own requests on top of the
  main conversation's" (costs). Its first request "doesn't read the
  parent's cache" (prompt-caching).

**Fork — documented** (sub-agents): "A fork is a subagent that
inherits the entire conversation so far instead of starting fresh …
Input: a fork sees the same system prompt, tools, model, and message
history as its parent conversation. Output: the fork's own tool calls
stay out of its parent conversation and only its final result comes
back."

- Started by `/subtask <task>`, or by Claude while fork mode is on (the
  default from v2.1.302).
- "its first request reuses the parent's prompt cache … a fork costs
  less than a fresh subagent for tasks that need the same context."
- "A fork can't spawn further forks."
- `/fork` with agent view on is a different thing: it "copies the
  whole session into a new background session."
- **Inference:** each fork request still carries the whole parent
  history as input, read from cache. A fork is cheap only when you
  would otherwise have to re-explain that history.

**Agent teams — documented** (agent-teams): "experimental and disabled
by default" (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`).

- "Teammates work independently, each in its own context window, and
  communicate directly with each other. You can also talk to any
  teammate directly without going through the lead."
- "Each teammate is a full, independent Claude Code session."
- "The lead's conversation history does not carry over."
- Permission prompts "appear in the lead session."
- Cost (costs): "approximately 7x more tokens than standard sessions
  when teammates run in plan mode."
- Warning: once teams are enabled, "a subagent that Claude names
  launches as a teammate, so teams can form even when you didn't ask
  for one."

**Documented** (sub-agents), when to use which. Main conversation when
"the task needs frequent back-and-forth … multiple phases share
significant context … quick, targeted change … latency matters."
Subagents when "verbose output you don't need in your main context …
enforce specific tool restrictions … self-contained and can return a
summary."

Sources: https://code.claude.com/docs/en/sub-agents ·
https://code.claude.com/docs/en/tools-reference ·
https://code.claude.com/docs/en/agent-teams

## 4. Tools, skills, hooks, MCP

**Context cost by feature — documented** (features-overview):

| Feature | When it loads | What loads | Context cost |
| - | - | - | - |
| CLAUDE.md | Session start | Full content | Every request |
| Skills | Session start + when used | Descriptions at start, full content when used | Low (descriptions every request) |
| MCP servers | Session start | Tool names; full schemas on demand | Low until a tool is used |
| Subagents | When spawned | Fresh context … or the parent conversation for a fork | Isolated from main session |
| Hooks | On trigger | Nothing (runs externally) | Zero, unless hook returns additional context |

**Tools.** Built-in actions: read, edit, search, Bash, web, and also
`Agent`, `AskUserQuestion`, `Skill` and `ToolSearch`. **Documented**
(prompt-caching): tool definitions sit in the system-prompt layer, and
a change to the set of definitions invalidates the cache. **Not
found:** the token size of the built-in definitions; `/context` shows
it.

**Skills.** A markdown file of instructions, loaded on demand.

- **Documented** (skills): "skill descriptions are loaded into context
  so Claude knows what's available, but full skill content only loads
  when invoked."
- Once loaded, "the rendered `SKILL.md` content enters the
  conversation as a single message and stays there across later
  turns."
- `disable-model-invocation: true` keeps even the description out
  (features-overview: "User-only skills have zero cost until
  invoked").
- `context: fork` "starts a new subagent of the type set in the
  `agent` field and gives it the skill content as its prompt. The
  subagent doesn't see your conversation history" (skills). Despite the
  name, it is not a conversation fork.

**Hooks.** Scripts that Claude Code (not the model) runs on lifecycle
events.

- **Documented** (features-overview): a hook runs "a shell command,
  HTTP request, MCP tool call, LLM prompt, or subagent"; it "Fires on
  its event without Claude deciding"; its context cost is "Zero unless
  the hook returns output." Also: "Put guardrails in hooks … a request,
  not a guarantee" applies to instructions, while a hook is
  enforcement.
- **Inference:** a hook of type prompt or subagent makes its own model
  call, so it spends tokens even when it adds nothing to your context.

**MCP — what the protocol is.** **Documented** (modelcontextprotocol.io,
architecture):

- **Participants.** "an MCP host — an AI application like Claude Code
  … establishes connections to one or more MCP servers. The MCP host
  accomplishes this by creating one MCP client for each MCP server."
- **Two layers:**
  - a data layer: "JSON-RPC based protocol … including capability and
    version discovery, and core primitives, such as tools, resources,
    prompts and notifications";
  - a transport layer: "Stdio transport" for local processes and
    "Streamable HTTP transport" for remote servers.
- **Server primitives:**
  - "**Tools**: Executable functions that AI applications can invoke
    to perform actions."
  - "**Resources**: Data sources that provide contextual information."
  - "**Prompts**: Reusable templates that help structure interactions."
  - Discovery and use go through `tools/list`, `tools/call`,
    `resources/read` and `prompts/get`.
- **Client primitive:** Elicitation ("Allows servers to request
  additional information from users").
- **Current version:** "Roots and sampling … are deprecated as of
  protocol version `2026-07-28`." The protocol is now stateless: every
  request carries version and capabilities in `_meta`.
- "MCP focuses solely on the protocol for context exchange—it does not
  dictate how AI applications use LLMs or manage the provided context."

**MCP in Claude Code — documented** (mcp):

- **Tools.** Tool search defers them: "Only tool names and server
  instructions load at session start, so adding more MCP servers has
  minimal impact on your context window." It is on by default on
  supported models. `ENABLE_TOOL_SEARCH=auto` loads tools upfront while
  they total "under 10% of the context window"; `false` disables tool
  search. `alwaysLoad: true` exempts a server.
- **Truncation limits.** Server instructions and upfront tool
  descriptions are cut at 4,096 characters; descriptions loaded
  through search at 16,384.
- **Output limits.** "a warning when MCP tool output exceeds 10,000
  tokens and limits output to 25,000 tokens by default
  (`MAX_MCP_OUTPUT_TOKENS`)."
- **Resources** are referenced as `@server:protocol://resource/path`
  and "automatically fetched and included as attachments."
- **Prompts** become commands: `/mcp__servername__promptname`.
- **Scopes:** local (default, `~/.claude.json`), project (`.mcp.json`),
  user.

**Why deferral matters — documented** (platform, tool-search-tool):

- "A typical multiserver setup (GitHub, Slack, Sentry, Grafana, and
  Splunk) can consume ~55k tokens in definitions before Claude does
  any work. Tool search typically reduces this by over 85 percent."
- "Claude's ability to pick the right tool degrades once you exceed
  30–50 available tools."
- Once loaded, "the tool definitions that search loads into context
  count as input tokens like any other tool definition."
- Caching (prompt-caching): with tools deferred, "a server connecting
  or disconnecting mid-session doesn't disturb anything already
  cached."

**Documented** (costs): "Prefer CLI tools when available: Tools like
`gh`, `aws`, `gcloud`, and `sentry-cli` are still more context-efficient
than MCP servers because they don't add any per-tool listing."

Sources: https://code.claude.com/docs/en/features-overview ·
https://code.claude.com/docs/en/skills ·
https://code.claude.com/docs/en/mcp ·
https://modelcontextprotocol.io/docs/learn/architecture ·
https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool

## 5. Measuring context per task

**What Claude Code exposes — documented:**

| Instrument | What it shows | Limits |
| :- | :- | :- |
| `/context [all]` | "Visualize current context usage as a colored grid," with optimization suggestions; `all` expands per item and shows tokens per MCP tool | Current session only |
| `/usage` (`/cost`, `/stats` are aliases) | Session tokens by model; `Prompt cache (main)` hit ratio and misses; on Pro, plan bars and **attribution** "to skills, subagents, plugins, and individual MCP servers," plus recommendations ≥10%; `d`/`w` switch 24 hours / 7 days | Cache line "covers the main conversation only, not subagents." Attribution is "approximate and computed from local session history on this machine" |
| Spinner count | "live, approximate count of the output the current turn has produced," including foreground subagents' output | Output only; not your context size |
| Status line JSON | `context_window.used_percentage`, `total_input_tokens`, `current_usage` (cache read/write split); `rate_limits.five_hour` / `seven_day.used_percentage` | `rate_limits` only for Pro/Max/Ultra, after the first response. `used_percentage` is input-only |
| OpenTelemetry | `claude_code.token.usage` by `type` (input/output/cacheRead/cacheCreation), `model`, `query_source` (`main`/`subagent`/…), `agent.name`, `skill.name`; `api_request` events per request | Custom agent names become `"custom"` unless `OTEL_LOG_TOOL_DETAILS=1`. Needs `CLAUDE_CODE_ENABLE_TELEMETRY=1` and a collector |
| Subagent transcript | `compact_boundary` entries with `preTokens` | Only when that subagent compacted |
| `claude -p … --output-format json` | `usage.cache_creation` (1h vs 5m writes) | Non-interactive run |

**Not found:** what the subagent's "small metadata trailer" contains
(token count, tool count, duration). Read it in a real task result
before relying on it.

**A practical scale of context needs.** The four levels are ours
(**inference**). The criteria under each quote the docs where they
exist.

**(a) No context — a plain chat suffices.**
- **Documented** (platform, how-tool-use-works): tool use "doesn't fit
  when … The model can answer from training alone … one-shot Q&A with
  no side effects."
- **Documented** (commands): inside a session, `/btw` asks "a side
  question about the current session without adding to the
  conversation."
- **Inference:** a question that needs no files, no project rules and
  no tools does not need Claude Code at all. Inside a busy session it
  still pays for the session's prefix, and `/btw` likely reads it once.

**(b) Message-only — a subagent that takes everything from the
delegation prompt.**
- **Documented** (sub-agents): `omitClaudeMd` is for "subagents that
  take everything they need from the delegation prompt." Explore and
  Plan "start without your CLAUDE.md files and the git status snapshot
  to keep research fast and inexpensive." "The main conversation still
  has your full CLAUDE.md when it reads these subagents' results, so
  most rules don't need to reach the subagent itself."
- Criteria (sub-agents): "self-contained and can return a summary";
  "verbose output you don't need in your main context."
- **Inference:** fits `researcher`-style or test-run tasks whose input
  fits in one message and whose rules can travel in that message.

**(c) Project context — CLAUDE.md plus the files it reads.**
- **Documented:** the default for every non-Explore/Plan subagent
  ("load both") and for every session.
- Criterion (features-overview): CLAUDE.md is for "'Always do X'
  rules"; "Keep CLAUDE.md under 200 lines."
- **Inference:** fits an implementer that must follow the project's
  conventions but does not need the conversation that produced the
  task.

**(d) Full session continuity — the main conversation, or a fork.**
- **Documented** (sub-agents): use the main conversation when "the
  task needs frequent back-and-forth or iterative refinement" or
  "multiple phases share significant context, such as planning,
  implementation, and testing." Use a fork "when any other subagent
  would need too much background to be useful."
- Counter-pressure (costs): "Use `/clear` to start fresh when switching
  to unrelated work. Stale context wastes tokens on every subsequent
  message."
- Across sessions (how-claude-code-works): "Sessions are independent.
  Each new session starts with a fresh context window."
- **Inference:** this is the most expensive level per request. Earn it
  only when the shared history is really used.

Sources: https://code.claude.com/docs/en/commands ·
https://code.claude.com/docs/en/costs ·
https://code.claude.com/docs/en/statusline ·
https://code.claude.com/docs/en/monitoring-usage

## Sources checked (2026-11-16)

- https://code.claude.com/docs/en/how-claude-code-works
- https://code.claude.com/docs/en/context-window
- https://code.claude.com/docs/en/features-overview
- https://code.claude.com/docs/en/model-config (both halves)
- https://code.claude.com/docs/en/sub-agents (both halves)
- https://code.claude.com/docs/en/agent-teams
- https://code.claude.com/docs/en/mcp (tool search, resources, prompts sections)
- https://code.claude.com/docs/en/costs
- https://code.claude.com/docs/en/commands
- https://code.claude.com/docs/en/statusline
- https://code.claude.com/docs/en/monitoring-usage (first 100k characters; later events not read)
- https://code.claude.com/docs/en/prompt-caching
- https://code.claude.com/docs/en/skills (first 100k characters)
- https://code.claude.com/docs/en/tools-reference (Agent tool section)
- https://modelcontextprotocol.io/docs/learn/architecture
- https://platform.claude.com/docs/en/agents-and-tools/tool-use/how-tool-use-works
- https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool
- https://platform.claude.com/docs/en/build-with-claude/context-windows
- https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents (Sep 29, 2025)

Not read: the hooks reference (the hooks-guide summary in
features-overview was enough), and the remaining OpenTelemetry events
(`subagent_completed` and others).

## Confidence

- **Confirmed:** sections 1, 2 (the three layers, size, compaction), 3
  (all four modes), 4, and the instruments in section 5.
- **Inference:** whether a main-thread agent can ask questions;
  per-request cost of forks; the four-level scale.
- **Not found:**
  - the position of the skill listing and agent descriptions inside the
    layers;
  - whether a fork or a teammate can use `AskUserQuestion`;
  - what the subagent metadata trailer contains;
  - the built-in tool definitions' token size.
- Version-gated behavior is up to v2.1.324 at time of reading;
  re-check before relying on it.
