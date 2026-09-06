# Reading List

Sources actually verified by the `researcher` agent before being cited
anywhere in this harness's design — not assumed from memory. Access
noted honestly for each: most loop-engineering sources are free blog
posts; Pressman's textbook is commercial, not open access.

## Loop engineering (freely accessible — all blog posts)

- **Addy Osmani** — ["Practical Loop Engineering"](https://addyo.substack.com/p/practical-loop-engineering)
  (Elevate, his own Substack). The original essay, published 2026-06-07
  — the canonical source the term traces back to. Osmani is a director
  on Google's Cloud AI team.
- **O'Reilly Radar** — ["Loop Engineering"](https://www.oreilly.com/radar/loop-engineering/),
  syndicated from Osmani's essay with his permission (2026-06-22).
- **IBM** — ["What Is Loop Engineering?"](https://www.ibm.com/think/topics/loop-engineering)
  — a clear vendor-neutral explainer of the same concept.
- Peter Steinberger's own compressed formulation ("stop prompting your
  agents and start designing the loops that prompt them") is credited as
  preceding/sparking Osmani's essay, but its exact original URL wasn't
  confirmed directly — worth tracking down properly before citing him
  by name in anything more formal than this list.

## Software engineering (Pressman — commercial, not open access)

- Pressman & Maxim, *Software Engineering: A Practitioner's Approach*,
  9th edition (ISBN 9781259872976) — confirmed as the most recent
  edition found; [publisher page](https://www.mheducation.com/highered/product/software-engineering-a-practitioners-approach-pressman.html).
  This is a paid textbook (purchase or library access), unlike the loop
  engineering sources above — noting the difference honestly rather than
  implying equal accessibility.

## What this grounded, concretely

The Architect agent's design (loop trigger → topology → verifier → stop
rule) is built directly from the structural elements these loop
engineering sources agree on — see `FOUNDATION.md` and
`.claude/agents/the-architect.md` for where that actually landed.
