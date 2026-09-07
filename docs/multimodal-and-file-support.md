# Multimodal & File Support Across AI Coding Harnesses

> **Orientation, if this is the first doc you're reading:** this is a
> `docs/` file in the Free Wings harness repository itself (see
> `FOUNDATION.md` for the whole picture), not part of any target
> project. It records research findings from the `researcher` agent — a
> grounded recap of what's actually confirmed about how AI coding
> agents/harnesses handle images, audio, and video, plus a separate,
> explicitly-labeled exploratory section on what this could mean for
> Free Wings itself. Its nearest neighbor is `docs/reading-list.md`
> (same kind of "verified, cited, honest about confidence" record),
> though this file also has an exploratory second half that
> `reading-list.md` doesn't.

---

## Part 1 — What's actually confirmed

The throughline for everything below: **whether an agent can actually
*see* an image or *hear* audio depends on the underlying model, not
just on the tool/harness wrapping it.** A harness can have every piece
of wiring in place — a `Read` tool, an attachment button, a plugin
system — and still return nothing useful if the model configured
behind it isn't multimodal-capable. Every claim in this section should
be read with that caveat attached, not as a footnote to it.

### Claude Code

- **Images — confirmed, high confidence.** Claude's models are
  natively multimodal (trained on image input, not bolted on
  afterward). Claude Code exposes this through the `Read` tool: pointed
  at an image file path, `Read` returns the image for the model to
  actually look at, the same way it returns text for a source file.
  Any subagent that already has `Read` in its declared tools list gets
  this for free — no separate per-agent image configuration exists or
  is needed. This is consistent with how `Read`'s own tool description
  is documented in this harness's tool surface (image files render
  visually; PDFs render page-by-page).
- **Video — not native, confirmed absence.** There is no "watch a
  video" primitive in Claude Code. Handling video would mean an
  external step first — extracting frames (e.g. via `ffmpeg`) and then
  feeding those frames through `Read` as images — not something the
  harness does automatically today.
- **Audio — not native, confirmed absence.** Same shape as video: no
  built-in audio ingestion. Would require a separate transcription
  step (a speech-to-text tool or service) before any content from
  audio reaches the model as text.

### OpenCode (opencode.ai, built by Anomaly/anomalyco)

- **Images — supported, but explicitly gated on the model.** OpenCode's
  own documentation describes native image attachment (drag-and-drop,
  paste, or an attach button in its interface), which gets base64-encoded
  and passed through to whatever model is currently configured. The
  documentation is explicit that this only works if that specific model
  is declared vision-capable — the attachment mechanism exists
  independent of whether the active model can use it, which is exactly
  the throughline stated above, confirmed concretely in OpenCode's own
  docs rather than inferred.
- **Audio/video — confirmed absent natively.** OpenCode's documentation
  explicitly excludes both from what gets sent in a model request. No
  native ingestion path for either.
  Confidence: high for "not built in as of the docs checked this
  session"; this is an actively developed project, so this specific
  absence is worth re-checking if cited again after significant time
  has passed — the docs, not memory, are the actual authority here.
- **Extension mechanism — real and concrete.** OpenCode has an actual
  plugin system (JS/TS) and supports MCP servers, which means the
  audio/video gap isn't necessarily permanent — it's addressable by
  extension rather than requiring a change to OpenCode's core. A
  real, findable example already confirmed: **`alfaoz/opencode-see-image`**,
  a plugin that adds a `see_image` tool. It routes an image through a
  *separate*, explicitly vision-capable model, specifically so a
  text-only backend model can still get image content described to it
  in text form. This is itself a working demonstration of the
  throughline above: the plugin doesn't make the primary model
  multimodal — it works around a non-multimodal model by delegating to
  one that is.

### Why the difference exists: vendor-integrated vs. model-agnostic (checked 2026-09-07)

A specific framing was raised and checked before being accepted:
**"Claude Code é pago e já tem tudo pronto; OpenCode tem
alternativas"** (Claude Code is paid and already has everything ready;
OpenCode has workarounds instead). Two separate things were verified
here — the causal claim about *why* the image-support gap exists, and
the paid/free framing itself — and they turned out not to be the same
claim.

**The causal claim (vendor-integrated vs. model-agnostic) is
confirmed, and it's the real reason, not the paid/free split.**
Claude Code is a single-vendor product: Anthropic's own docs
(`code.claude.com/docs/en/model-config`, checked this session) list
Amazon Bedrock, Google Cloud's Agent Platform, and Microsoft Foundry as
"supported providers," but every one of those is still *Claude*
running on different cloud infrastructure — not a different model
vendor. A `ANTHROPIC_BASE_URL` override to point Claude Code at a
genuinely different model (e.g. via Ollama or LM Studio's
Anthropic-compatible endpoint) exists as a community workaround, but
it sits outside Claude Code's officially guaranteed path: the same
docs warn that "provider-specific IDs... often don't match these
patterns, leaving supported features disabled," require manually
declaring capabilities via environment variables, and never mention
vision/image support at all for non-Anthropic deployments — the
opposite of the "just works" guarantee Claude Code gives when it's
actually talking to Claude. So the vendor-integrated framing from Part
1 holds: Anthropic controlling both the tool and the model is what
lets Claude Code promise `Read` + vision work together by default,
and stepping outside that pairing is explicitly unsupported, not just
untested.

OpenCode's architecture is the mirror image by design, not by
necessity: genuinely model-agnostic from the start (any provider, any
local model, via its own docs and confirmed by its BYOK — "bring your
own key" — configuration), which is exactly why it can't make the same
default guarantee Claude Code makes, and exactly why a plugin like
`opencode-see-image` exists as a compensating mechanism rather than a
missing feature Anthropic simply hasn't built.

**The paid/free framing, checked separately, is inaccurate as stated
and worth correcting.** Both tools' CLI layer is free to obtain and
run:

- **Claude Code** is closed-source and proprietary (Anthropic PBC,
  "all rights reserved," no OSI-approved license — confirmed via
  multiple sources on its 2026 accidental source-code leak and
  Anthropic's subsequent DMCA action against a de-obfuscated
  republish), but the CLI itself has no purchase price. What actually
  costs money is *usage against Claude* — a Pro ($20/mo), Max
  ($100–200/mo), Team/Enterprise subscription, or pay-as-you-go API
  billing through Anthropic. Because Claude Code's officially
  supported path is Claude-only (see above), there is no way to run it
  for genuinely $0 through a supported configuration — the cost is
  structural, not a licensing fee on the tool.
- **OpenCode** is open-source and free to obtain and run, and its
  model-agnostic design is what makes a genuinely $0 configuration
  possible: it can be pointed at self-hosted local models at no cost
  (confirmed via OpenCode's own docs and its "Free (BYOK)" tier, which
  explicitly includes local-model support). It is not free by default,
  though — pointing OpenCode at Claude, GPT, or another hosted API
  costs exactly what that provider charges, same as Claude Code, and
  OpenCode also sells its own managed plans (OpenCode Go, ~$10/mo;
  Zen pay-as-you-go).

So "Claude Code is paid, OpenCode is free" oversimplifies two axes
into one. The accurate version: **both CLIs are free software to run;
the real, confirmed difference is that OpenCode's model-agnostic
design makes a genuinely $0 backend possible (local models), while
Claude Code's vendor-integrated design forecloses that option on its
officially supported path — the identical design choice that also
gives Claude Code its default `Read` + vision guarantee.** The
paid/free difference people notice in practice is a downstream
consequence of the vendor-integrated vs. model-agnostic split, not an
independent fact about the tools.

Sources checked this session: Anthropic's own
[Claude Code model configuration docs](https://code.claude.com/docs/en/model-config);
web search results summarizing Claude Code pricing structure (Pro/Max/API
billing, no standalone CLI fee) and OpenCode pricing structure (free
BYOK tier including local models, plus paid Go/Zen plans); web search
results confirming Claude Code's proprietary/closed-source status
(license file, 2026 source-leak/DMCA reporting) versus OpenCode's
open-source status.

### Summary of the throughline

| | Images | Audio | Video |
|---|---|---|---|
| Claude Code | Native, via `Read`, model-dependent on Claude's own vision capability | Not native — needs external transcription | Not native — needs frame extraction |
| OpenCode | Native attachment UI, gated on configured model's vision capability | Not native — no built-in path; plugin/MCP possible | Not native — no built-in path; plugin/MCP possible |

In both harnesses, the wiring (a tool, a UI, a plugin hook) is
necessary but not sufficient. The actual determining factor is always
one layer down: what the *configured model* can natively process.
`opencode-see-image` is the clearest confirmed evidence of this,
because it exists specifically to route around a model that can't see
images by calling a different model that can — the plugin doesn't add
"sight," it adds a bridge to a model that already has it.

---

## Part 2 — Exploratory ideas for Free Wings

**These are options to think about, not a decision, not a
recommendation to build, and not a scoped implementation plan.**
Deciding whether to act on any of this belongs to the person, and
actually building anything here (if it's decided worth doing) belongs
to `programmer`/`the-architect`, not to this research task. Nothing
below should be read as this agent recommending a next step.

### Option A — Document the existing Claude Code capability where it's already true

Claude Code subagents with `Read` in their tools list already see
images with zero extra configuration. That's a fact about the current
state, not a proposal. The open question is *where* that fact belongs:

- In `FOUNDATION.md` itself — but `FOUNDATION.md` is explicitly
  tool-agnostic by design ("Nothing in this file should ever assume a
  specific AI tool is reading it"). A Claude-Code-specific detail like
  "the `Read` tool sees images" would, by the harness's own stated
  rule, not belong there directly — it would belong in the generated,
  tool-specific `CLAUDE.md`, or in a doc `FOUNDATION.md` references
  without asserting itself.
- In a target project's own docs — makes sense if and when a specific
  project actually starts using images (e.g. screenshots, UI mockups,
  hardware photos) as part of its work. Shadow Glass, for instance,
  hasn't obviously needed this yet, but a future phase involving visual
  debugging (comparing a captured frame against an expected one) could.
- The honest baseline: this may not need any new file at all right
  now. It's already true and already works; documenting a true,
  already-working thing has value only when someone would otherwise be
  confused about whether it works — which is exactly this session's
  own starting confusion, worth weighing seriously (see Option B).

### Option B — A short, harness-agnostic explainer

This session's own conversation is itself the evidence for this
option's usefulness: the question "can an agent code see images or
hear audio" doesn't have one universal answer, and a newcomer adopting
this harness with no prior exposure to any of this would hit the same
confusion. A short explainer — a few paragraphs, not a spec — could
plainly lay out:

- The two-layer reality: tool/harness wiring, and underlying model
  capability, are separate questions with separate answers.
- How to actually check, for whichever harness someone is using —
  e.g., "does your subagent have `Read` in its tools list, and is the
  model you're running genuinely multimodal" for Claude Code; "is your
  configured model declared vision-capable in OpenCode's model config"
  for OpenCode.
- That audio and video are, as of the sources checked, not natively
  handled by either harness, and what the fallback paths look like
  (frame extraction, transcription, or a delegating plugin like
  `opencode-see-image`).

Whether this becomes a new `docs/` file in Free Wings, a section in an
existing file, or nothing at all (left to each target project to
figure out itself the first time it comes up) is an open question, not
resolved here.

### Option C — A lightweight, optional FOUNDATION.md convention

A more modest version of Option A/B: instead of a general explainer, a
documented *pattern* a target project could optionally follow — a
sentence or two in that project's own `FOUNDATION.md` noting something
like "backend model: [X], vision-capable: [yes/no/unconfirmed]." This
would be genuinely minimal (no new tooling, no new agent, no schema to
maintain) and would make an implicit, easy-to-forget fact explicit at
the one place a project's context already lives.

Honest weighing of this, both directions:

- **The case for it**: it's nearly free — a documentation convention,
  not infrastructure — and it directly addresses a real, demonstrated
  confusion (this session's own). It costs one sentence per project,
  optionally.
- **The case against it right now**: Free Wings currently has one
  target project (Shadow Glass), which hasn't yet had a task that
  needed image/audio/video handling by an agent. Writing a convention
  for a need that hasn't shown up yet risks exactly the kind of
  speculative scaffolding this harness's own philosophy (and the
  ponytail-mode ladder governing implementation work in this
  environment) both push against — "does this need to exist at all" is
  a fair question to ask before adding even a one-line convention.
  Doing nothing right now, and revisiting this the first time a real
  project actually needs multimodal input, is a legitimate, defensible
  answer too.

### Option D — Flagged separately: real new infrastructure

Anything beyond documentation — e.g., building or adopting an OpenCode
plugin for audio transcription, standing up a frame-extraction step as
a reusable Free Wings skill, or writing a new agent whose job is
multimodal ingestion — is a materially bigger decision than anything
above. It would mean new dependencies, new failure modes, and
maintenance burden, not just a paragraph of documentation. Explicitly
flagging this as its own separate decision, not something to fold in
casually alongside the documentation options above, and not something
this research task is proposing.

---

**Part 2 above is a set of options to consider, not a decision made or
a recommendation to act on immediately.** That call belongs to the
person, informed by what's written here — not to this agent.
