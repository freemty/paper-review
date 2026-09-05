# paper-review

Version 1.1.0. Evidence-grounded academic reviews from a complete manuscript,
with optional independent role analysis and a main-thread fallback.

Read the actual paper and requested supplements; record missing pages, figures,
equations or experiments. Select review perspectives for the paper's risks, not
a fixed four reviewers. Reconcile disagreements against source evidence and
deliver the requested review in full. No field-by-field approval is required
unless the user asks for that interaction.

Use the current venue/year form supplied by the user or verify its official
instructions. Bundled venue formats are historical examples, not current rules.
Scores follow evidence; a requested target score cannot justify invented praise
or criticism. Nothing is submitted to a review portal automatically.

## Installation and invocation

Codex: install paper-review from the yuanbo-skills plugin marketplace; use
`$paper-review:paper-review` or the skill selector. Do not also install global
symlinks for these plugin skills.

Other Agent Skills hosts can install this skill folder through their supported
installer and use natural language or their selector. Claude's standalone skill
installation supports `/paper-review`; no Claude plugin manifest is distributed
by this repository.

Review-quality checks use `review-review/references/audit-rubric.md` as needed.
Missing evidence is unverified, not automatically a hallucination.

MIT.
