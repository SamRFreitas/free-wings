# Reading List

Sources actually verified by the `researcher` agent before being cited
anywhere in this harness's design — not assumed from memory. Access
noted honestly for each: most loop-engineering sources are free blog
posts; Pressman's textbook is commercial, not open access.

## Loop engineering (freely accessible — all blog posts)

- **Addy Osmani** — ["Loop Engineering"](https://addyosmani.com/blog/loop-engineering/)
  (2026-06-07). The essay the term traces back to (checked 2026-11-16).
  It names the verifier, the stop condition and the maker/checker
  split; the four-part "trigger, topology, verifier, stop rule" naming
  is **not** in it. Osmani is a director on Google's Cloud AI team.
- **Addy Osmani** — ["Practical Loop Engineering"](https://addyo.substack.com/p/practical-loop-engineering)
  (Substack, 2026-08-14). A later post, not the June essay. *Correction:
  this link used to be listed here as the June essay; it is not.*
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

The Architect's exit and checks borrow the loop-engineering vocabulary
where the sources use it (verifier, stop condition). "Trigger" and
"topology" are the harness's own words — see
`hangar/blueprints/agents/the-architect.md`, "Loop anatomy", and
`docs/research/2026-11-16-harness-vocabulary.md`, section 6.
