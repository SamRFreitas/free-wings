---
name: write-article
description: Write a longer-form, publishable article from a target project's raw material (LEARNING_LOG, docs/observations/, ADRs) — for external readers, not just a personal record. Use when a piece of work or a theme has enough depth to deserve a standalone article.
---

# write-article

Produces a standalone article — for people outside the project to read,
not just the person who did the work. Different from `write-diary`:
a diary entry is a personal record of one session; an article picks one
theme or piece of work and develops it fully for an audience.

## Procedure

1. Read `/Users/samrfreitas/Lab/free-wings/.claude/agents/writer.md`
   for the writer's general themes and labeled project-specific
   parallels — apply the Clean-Code-style formatting theme especially
   here (one concept per section, real example, why before how).
2. Read the target project's `FOUNDATION.md`, plus whichever
   `LEARNING_LOG.md` entries, `docs/observations/` files, or ADRs the
   article is actually about.
3. Pick one real theme or piece of work as the article's spine — not a
   tour of everything that happened. Ground every claim in something
   that actually happened in the project; never invent an example.
4. Write the article to `docs/articles/<short-slug>.md` in the target
   project (create the folder if it doesn't exist yet).
5. Show the draft before treating it as final — this is meant for other
   people to read, so confirm it says what the person actually wants
   said before it's considered done.
