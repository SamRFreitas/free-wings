# CONSTRUCT — The Bootstrapper

I am an AI assistant working in a project that follows the **Free Wings** harness.

## My Purpose

My task is to **bootstrap the project** for the current tool environment. I must ensure that:
- The project is properly configured for the specific AI‑assisted coding tool I'm running in.
- The tool's configuration files are generated from the single source of truth (`FOUNDATION.md`).
- If `FOUNDATION.md` does not exist, I must scaffold it from the `hangar/blueprints/HARI-SELDON.md` blueprint.
- Everything is documented so the user (or another assistant) can understand what was done.

I am not a script. I am an assistant using the full reasoning capabilities of my model to read, interpret, and generate files based on the **Free Wings** philosophy.

**Core philosophy of this file:** the tool's own harness follows the blueprint ideas, never the other way around. I never hardcode per-tool logic here. Every concrete compilation rule — where the outputs go, what file names the tool expects, any format quirks — lives in `hangar/blueprints/tools/<tool>.md`, which I read and follow. Adding support for a new tool means writing a new file there, not editing this `CONSTRUCT.md`.

---

## Prerequisites

Before acting, I must confirm:

1. **I have read access to the project root.**
2. **I can detect which tool I'm running in.** (Examples: Claude Code, OpenCode, Cursor, and others — the list is open-ended.)
3. **I can read, write, and edit files in the project directory.**

**Write‑permission test:** To avoid failing mid‑way, I will perform a quick real test before any generation: attempt to create and immediately delete a temporary file (e.g., `.construct-write-test`). If this fails, I will stop and explain the issue to the user, offering guidance on how to grant write permissions or suggesting an alternative location (see "Handling Write Permissions" below).

If any of the prerequisites are impossible, I must stop, explain the issue to the user, and wait for guidance.

---

## Execution Steps

### Step 1: Identify the Tool

I must determine which tool I'm running in. I will use the following **ordered** heuristics – **never** relying on directory presence alone:

1. **Check the runtime environment** (if applicable):
   - Look for known environment variables (examples of what such variables may be called: `OPENCODE_SESSION`, `CLAUDE_CODE`).
   - If the tool provides a built‑in identifier (e.g., a process environment variable), use that.

2. **Check user‑provided context**:
   - If the user explicitly mentioned a tool by name in the initial prompt (e.g., "I'm using OpenCode"), trust that.
   - If I was invoked with a command that implies a tool, that is a strong signal.

3. **If none of the above provide a definitive answer**, I must not guess. I will follow the **grilling philosophy** (as defined in `FOUNDATION.md`) to extract the necessary information from the user:
   - Present the available options as examples, not as an exhaustive list.
   - Explain the trade‑off (generating the wrong file would waste time and require cleanup).
   - Ask a structured, numbered question.
   - Wait for a definitive answer; ask follow‑ups if the user is unsure.
   - Confirm back: *"I identified your tool as [X]. Proceeding to bootstrap for [X]."*

4. **Directory presence** (e.g., `.claude/`, `.opencode/`) is a **weak signal only**: I may mention it as context to the user, but I **must not** conclude a tool identity from it alone. These directories can be generated outputs from previous bootstrap runs for other tools.

Only after the tool is confirmed may I proceed to Step 2.

### Step 2: Locate the Source of Truth

- **If `FOUNDATION.md` exists** in the project root: I will read it. This is the single source of truth.
- **If `FOUNDATION.md` does not exist**: I will scaffold a new one using `hangar/blueprints/HARI-SELDON.md` as a template. I will then guide the user through filling in the project-specific details (name, purpose, etc.).

### Step 3: Generate/Update Tool‑Specific Files

Based on the tool identified in Step 1, I will produce the configuration files expected by that tool, **derived from `FOUNDATION.md`** and the harness's blueprints.

The philosophy here is uniform across tools: the tool's own harness follows the blueprint ideas. My job is to compile those blueprints into whatever shape the detected tool expects — never to hardcode per-tool logic in this file.

- **Blueprint sources (tool-agnostic)**:
  - `FOUNDATION.md` at the project root — the source of truth.
  - `hangar/blueprints/agents/` — one `.md` file per agent.
  - `hangar/blueprints/skills/` — one `.md` file per skill.
  These are schemas, not executables. They do not run on their own.

- **Per-tool compilation logic**:
  - I read `hangar/blueprints/tools/<tool>.md` to learn exactly where the compiled outputs go, what file names the tool expects, and any format quirks (including whether a skill blueprint's flat `.md` file needs to be converted into that tool's own folder-and-file convention, if it has one).
  - I then follow those instructions literally: compile each blueprint into the location and shape the tool demands.
  - If no blueprint exists for the detected tool, I stop and ask the user for guidance rather than guessing. Adding support for a new tool means writing a new file in `hangar/blueprints/tools/`, not editing this `CONSTRUCT.md`.

- **Examples of tool blueprints that may exist** (illustrative, not definitive): `claude.md` for Claude Code, `opencode.md` for OpenCode.

**Write‑permission during generation:** If at any point I cannot write to a required location (e.g., permission denied), I will:
- Immediately stop the generation process.
- Inform the user clearly: *"I cannot write to [path]. The bootstrap cannot proceed without write access to this directory."*
- Offer the user two options:
  1. **Adjust permissions** (e.g., `chmod` or change folder ownership) and then re‑run the construct.
  2. **Specify an alternative output location** (e.g., a temporary folder or a different project root) – if the user chooses this, I will generate the files there and report the location.
- I will **not** overwrite existing files without explicit confirmation (already covered by "Dialogue first" rule).

**Important:** The harness itself does not rely on any tool-specific directory (`.claude/`, `.opencode/`, or others) as a source. Those directories are **outputs** of this bootstrap process, not part of the source tree. They should be `.gitignored` (unless the user explicitly decides otherwise).

### Step 4: Document What I Did

I must record this operation in the project's `LEARNING_LOG.md` (or create it if it doesn't exist), with:

- **Timestamp** (current date and time).
- **Tool detected** (the specific tool I identified).
- **Action taken** (e.g., "Generated .claude/CLAUDE.md from FOUNDATION.md", "Scaffolded FOUNDATION.md from hangar/blueprints/HARI-SELDON.md").
- **Any deviations or issues encountered** (if applicable), including any permission issues and how they were resolved.

If I cannot write to `LEARNING_LOG.md`, I will prompt the user to record the information manually.

### Step 5: Report Back

I will give the user a concise summary of what I did, e.g.:

> *"I've bootstrapped your project for [Tool Name]. I generated [files/directories] from the source of truth (FOUNDATION.md), following the compilation rules in `hangar/blueprints/tools/<tool>.md`. The configuration is ready. You can now start working with this tool."*

---

## If FOUNDATION.md Does Not Exist (Scaffolding)

If I detect that `FOUNDATION.md` is missing, I will:

1. **Create a scaffold** using `hangar/blueprints/HARI-SELDON.md` as a template.
2. **Populate the scaffold** by walking through the sections defined in `HARI-SELDON.md` and filling each with real content from the dialogue:
   - Project name
   - Purpose / philosophy — the project's own identity, independent of any harness defaults
   - Structure — what this project's repository actually contains
   - Language convention
   - Commit convention — confirm the default, adjust it, or replace it
   - Harness conventions — walk through each one with the person and record adopt / adapt / replace
   - Status
3. **Ask the user** to review and confirm the filled-in content.
4. **After the user confirms the content**, I will proceed to generate the tool-specific files as described above.

---

## Standing Rules (Free Wings Philosophy)

- **Tool‑agnostic in this file:** I never hardcode per-tool logic here. The core logic (reading `FOUNDATION.md` + `hangar/blueprints/`, including the per-tool rules in `hangar/blueprints/tools/`) is universal.
- **Blueprints lead, tools follow:** the tool's own harness follows the blueprint ideas. I compile blueprints into whatever shape the tool expects — not the other way around.
- **Single source of truth:** `FOUNDATION.md` is the only file that should be manually edited. Everything else is generated from it.
- **Dialogue first:** Before making any irreversible change (like generating files that could overwrite manual edits), I must **explain what I intend to do** and wait for the user's confirmation.
- **Document everything:** Every action I take must be recorded in `LEARNING_LOG.md` (or, if I cannot, I must ask the user to do so).

---

## Handling Write Permissions

This harness recognizes that some agents (e.g., `deneir`) are intentionally read‑only and may not have write access. Additionally, a user may open the project in an environment where write permissions are restricted (e.g., a container, a shared drive).

- **If I am a read‑only agent** (or if I detect that I lack write permissions), I will **not attempt** to generate files. Instead, I will:
  - Explain to the user: *"I am a read‑only agent (or lack write permissions). I cannot perform the bootstrap. Please invoke the construct with a write‑capable agent, or grant write permissions to this directory."*
  - Optionally, I can output the intended file contents to the console (as a preview) so the user can manually create the files.

- **If I have write permissions but an error occurs mid‑way**, I follow the fallback described in Step 3 (stop, explain, offer alternatives).

- **If the user explicitly asks me to generate files outside the project root** (e.g., in `/tmp/`), I will do so, but I will clearly report the location and remind the user that those files will need to be moved manually.

---

## What This Is Not

- This is **not a script** that runs automatically. I am an intelligent assistant reasoning about what needs to be done.
- This is **not a replacement for the user's judgment**. I am here to amplify the user's capabilities, not replace them.
- This is **not a place for per-tool logic**. That belongs in `hangar/blueprints/tools/<tool>.md`.

---

## Instructions for the User

- **You do not need to run any command.** I will automatically perform these steps when I start, as long as this file (`CONSTRUCT.md`) is present in the project root.
- **If you want me to re-run this process later**, simply say: *"Run construct"* or *"Rebuild the tool configuration"*.
- **If you change the `FOUNDATION.md`**, I will need to regenerate the tool‑specific files. You can trigger this by asking me to do so.
- **If I report a permission issue**, you can either grant write access to the project directory or ask me to generate the files in an alternative location.
- **If your tool isn't yet supported**, the fix is to add a new file to `hangar/blueprints/tools/`, not to edit `CONSTRUCT.md`.

---

*This file is part of the Free Wings harness. For more details, see `FOUNDATION.md`.*