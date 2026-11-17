# Token efficiency in Claude Code — what the docs actually say

> Researched 2026-11-11 by `researcher`. Trigger: in Chacal, `/usage`
> (24 h) attributed 66% of usage to `engenir` and 24% to `tester`. Goal:
> ground every claim before proposing defaults for every project under
> Free Wings. Sources are primary only (Anthropic docs). Verdicts:
> **documented** (quoted), **contradicted**, **inference**, **not found**.

## Sources checked (2026-11-11)

- https://code.claude.com/docs/en/costs
- https://code.claude.com/docs/en/sub-agents
- https://code.claude.com/docs/en/prompt-caching
- https://code.claude.com/docs/en/model-config
- https://code.claude.com/docs/en/memory
- https://code.claude.com/docs/en/hooks (partially; the page is longer than the fetch window)
- https://platform.claude.com/docs/en/build-with-claude/prompt-caching (docs.claude.com redirects here)

Not read: the Claude Code changelog (not needed), the `errors` page
section on the 15k warning (truncated in fetch; the sub-agents page
covers it).

## Claims

### 1. Subagent `effort` field — documented

sub-agents, frontmatter table: "Effort level when this subagent is
active. Overrides the session effort level, but not the
`CLAUDE_CODE_EFFORT_LEVEL` environment variable. Options: `low`,
`medium`, `high`, `xhigh`, `max`; available levels depend on the model."

Without the field: "On the main conversation's model, the subagent runs
at the level you pick in the session, such as with `/effort`, or set
with `--effort`. On another model, that level doesn't apply."

model-config: "Fable 5.2, Opus 5.5, Sonnet 5.5, and Haiku 5.5 default
to `medium`, and Opus 4.7 defaults to `xhigh`. Other models … default
to `high`." An unsupported level falls back "to the highest supported
level at or below the one you set."

**Not found:** whether `effort` in an agent file applies when that file
runs as the main thread (`--agent` / `"agent"`). The docs state this for
`hooks`, `mcpServers`, `tools`, `model`, `initialPrompt` and
`omitClaudeMd`, but not for `effort`.

### 2. Subagent `model` field — documented

sub-agents, frontmatter table: "`sonnet`, `opus`, `haiku`, `fable`, a
full model ID such as `claude-opus-5-5`, or `inherit`. When you omit
it, Claude Code picks the model in the subagent model order."

The order: "1. The per-invocation `model` parameter 2. The subagent
definition's `model` frontmatter, where `inherit` selects the main
conversation's model 3. The `CLAUDE_CODE_SUBAGENT_MODEL` environment
variable … 4. The main conversation's model."

So when the field is left out and the variable is unset, the subagent
effectively inherits. model-config adds: "Switch to Opus before Claude
delegates research or test runs to one of them, and that work runs on
Opus too. To keep a custom subagent on a smaller model, set `model` in
its definition."

Main thread: "Pass `--agent <name>` to start a session where the main
thread itself takes on that subagent's tool restrictions and model."
`"agent"` in `.claude/settings.json` does the same as a project default;
"The CLI flag overrides the setting if both are present."

### 3. Prompt-cache TTL — documented; depends on billing, differs by bucket

prompt-caching, "Which TTL each request gets": two buckets. "Main
conversation" covers interactive turns, `-p` and SDK turns. "Everything
else" covers "subagents, workflows, in-process teammates, forks,
compaction, and session titles".

| Request bucket | Claude subscription, within plan usage | Usage credits, API key, or cloud provider |
| - | - | - |
| Main conversation | One hour | Five minutes |
| Everything else | Five minutes, except the server-controlled helper requests, which get one hour | Five minutes |

"Subagents fall outside the main-conversation TTL bucket, so they get
five minutes even on a subscription until you choose a longer one."

The 1-hour main-session TTL observed earlier is therefore the documented
subscription default. Controls: `promptCacheTtl` /
`CLAUDE_CODE_PROMPT_CACHE_TTL` for the main conversation;
`subagentPromptCacheTtl` / `CLAUDE_CODE_SUBAGENT_PROMPT_CACHE_TTL` for
the rest; per subagent, `experimental: { cacheTtl: 5m|1h }`, which is
ignored at `1h` while on usage credits. Each takes `5m` or `1h`.

Price (platform prompt-caching): "Cache write tokens with a 5-minute TTL
are 1.25 times the base input tokens price." "Cache write tokens with a
1-hour TTL are 2 times the base input tokens price." Reads are 0.1x by
default; "Cache hits and refreshes on Claude Opus 5.5 and Claude Sonnet
5.5 are priced at 0.05x the base input price." "The cache is refreshed
for no additional cost each time the cached content is used."

Trade-off (prompt-caching): the 1-hour TTL "helps when you leave a
session idle and come back to it … It costs more on short bursts of
work that never idle past five minutes."

### 4. `omitClaudeMd` — documented; subagents do load CLAUDE.md by default

sub-agents: "Explore and Plan start without your CLAUDE.md files and the
git status snapshot to keep research fast and inexpensive. Other
built-in and custom subagents load both, unless you set `omitClaudeMd`."

Field: "Set to `true` to start this subagent without the user, project,
and local CLAUDE.md files in its initial context; managed policy files
still load … Use it for subagents that take everything they need from
the delegation prompt. Ignored when the agent runs as the main session
agent via `--agent` or the `agent` setting. Requires Claude Code
v2.1.271 or later."

Caveat: "For Explore, Plan, and subagents with `omitClaudeMd`, a
subdirectory's `CLAUDE.md` and matching path-scoped rules still load
when the subagent works with files there." And: "The main conversation
still has your full CLAUDE.md when it reads these subagents' results,
so most rules don't need to reach the subagent itself."

The main conversation's auto memory never reaches a non-fork subagent
(memory, sub-agents).

### 5. `PreToolUse` hook filtering test output — documented

costs, "Offload processing to hooks and skills": "This PreToolUse hook
trims test output to its last 100 lines". It uses a `Bash` matcher in
`settings.json` and runs `~/.claude/hooks/filter-test-output.sh`. When
the command matches `^(npm test|pytest|go test)…$`, the script rewrites
it to `"$cmd 2>&1 | tail -n 100"` through
`hookSpecificOutput.updatedInput`.

Also: "Instead of Claude reading a 10,000-line log file to find errors,
a hook can grep for `ERROR` and return only matching lines, reducing
context from tens of thousands of tokens to hundreds."

hooks: "Hooks from settings files, managed policy settings, and plugins
also run inside subagents." Frontmatter `hooks` "fire … when the agent
runs as the main session via `--agent` or the `agent` setting."

### 6. 15,000-token description warning — documented

sub-agents: "When the combined descriptions of your subagents, except
the built-in ones, exceed 15,000 tokens, Claude Code shows a warning at
startup with the total token count. Trim the `description` fields of
your subagents, and move detail into each subagent's system prompt,
which only loads when that subagent runs." Also: it "still loads every
subagent."

### 7. `/clear` free, `/compact` a model call — documented, with nuance

costs: "`/compact` reads the conversation it summarizes, so compacting a
large context is itself a large request. When you want a fresh start
instead of continuity, `/clear` costs nothing."

prompt-caching qualifies `/compact`: "While the cache is warm, that
request reads your prefix from the cache. A mid-session `/compact` then
costs a fraction of what the context size suggests … After a break
longer than the cache lifetime … the summarization request reprocesses
that history as uncached input. This is why `/compact` costs the most
when you resume an old session."

Also: "If you've gone down a path you want to abandon entirely,
`/rewind` to an earlier turn instead. Rewinding truncates back to a
prefix that is already cached." Idle compaction on the 1-hour TTL can
run while you're away; turn it off with `idleCompaction: false`
(v2.1.290+).

### 8. "7× more tokens" — documented for agent teams only

costs: "Agent teams use approximately 7x more tokens than standard
sessions when teammates run in plan mode, because each teammate
maintains its own context window and runs as a separate Claude
instance."

**Inference:** it does not describe ordinary subagents. The docs give no
multiplier for them, only: "each subagent spends tokens of its own
while it runs" and "every subagent … sends its own requests on top of
the main conversation's."

### 9. Main-thread agent body replaces the default system prompt — documented

sub-agents: "Unless the agent's prompt is empty, a custom subagent's
system prompt replaces the default Claude Code system prompt entirely,
the same way `--system-prompt` does. `CLAUDE.md` files and project
memory still load through the normal message flow, even when the
agent's definition sets `omitClaudeMd`."

Resent every turn, and cached (prompt-caching): "Claude Code re-sends
the full context: the system prompt, your project context, every prior
message and tool result, and your new message." The system prompt layer
comes first and is invalidated only when "the set of loaded tool
definitions changes." So the agent body is billed every request, at the
cached-read rate while the cache is warm and at full write rate after a
cold gap.

### 10. Official guidance

**When delegation saves vs. costs** (sub-agents):
"Use the main conversation when: the task needs frequent back-and-forth
or iterative refinement; multiple phases share significant context,
such as planning, implementation, and testing; you're making a quick,
targeted change; latency matters." "Use subagents when: the task
produces verbose output you don't need in your main context; you want
to enforce specific tool restrictions or permissions; the work is
self-contained and can return a summary."

costs: "The subagent's own requests still draw on your usage. To spend
less on them, choose a smaller model for a subagent or run every
subagent on one model."

A fork "costs less than a fresh subagent for tasks that need the same
context," because it shares the parent's cache.

**Cheaper model per subagent** (costs): "Sonnet handles most coding
tasks well and costs less than Opus. Reserve Opus for complex
architectural decisions or multi-step reasoning … For simple subagent
tasks, specify `model: haiku`." To force one model on every subagent,
set `CLAUDE_CODE_SUBAGENT_MODEL` together with
`CLAUDE_CODE_SUBAGENT_MODEL_FORCE=1` (v2.1.257+).

**Effort / thinking** (costs): "Thinking tokens are billed as output
tokens … For simpler tasks … you can reduce costs by lowering the effort
level." Changing effort mid-session keeps the cache on Opus 5.5, Sonnet
5.5, Haiku 5.5 and Fable 5.1+ with an API key or subscription; on other
models it invalidates the cache (prompt-caching).

**Short command output** (costs, sub-agents): the hook above, plus the
sub-agents example prompt "Use a subagent to run the test suite and
report only the failing tests with their error messages."

**Session hygiene** (costs):
- "Use `/clear` to start fresh when switching to unrelated work. Stale
  context wastes tokens on every subsequent message."
- "Claude Code sends your full conversation with every request … a
  one-line question in a session that has been open all day still draws
  usage for the whole conversation."
- "your first message after a break longer than the cache lifetime
  misses the cache and reprocesses your full context."
- "Pick your model and effort level at the top of a session"
  (prompt-caching tip); switching models mid-session reads the whole
  history uncached.

**CLAUDE.md size** (costs, memory): "Aim to keep CLAUDE.md under 200
lines." "Imports … don't reduce its context cost." "Block-level HTML
comments … are stripped before the content is injected." Workflow-
specific instructions belong in skills: "Skills load on-demand only
when invoked."

**Background usage** (costs): session recap, idle compaction, prompt
suggestions, scheduled `/loop` tasks, cross-session messages and goal
check-ins can all send the full context while the session is idle.

**Per-subagent compaction** (sub-agents): `autoCompactWindow`
(100000–1000000, plain integer, v2.1.296+). "You run the whole session
as this subagent with `--agent` … the field has no effect there."

## What this means for Free Wings (assessment, not decisions)

- **Inference:** `engenir` and `tester` are opened with `claude --agent`,
  so they are main-thread sessions. The subagent-only levers do not
  apply to them: `omitClaudeMd`, `autoCompactWindow`, the
  subagent-bucket TTL, and possibly `experimental.cacheTtl`. The levers
  that do apply: `model` (documented), `hooks` (documented), session
  hygiene and CLAUDE.md size. `effort` is unconfirmed (claim 1).
- **Not found:** how `/usage` attributes a session opened with
  `--agent` (labelled as a "subagent" or not). Worth confirming before
  reading the 66% / 24% figures.
- **Inference:** for the subagent-run agents (`researcher`, `deneir`,
  `writer`), `model`, `effort` and possibly `omitClaudeMd` are the
  documented levers. Whether omitting CLAUDE.md is safe depends on
  whether the delegation prompt carries the rules each one needs.

## Confidence

Confirmed for claims 1–7, 9 and 10 against the pages above.
Claim 8: confirmed for agent teams; its non-application to subagents is
inference. Open: `effort` under `--agent`; `/usage` attribution under
`--agent`; partial-invalid `tools` lists. Docs change often (version
notes go up to v2.1.321); re-check before relying on version-gated
fields.
