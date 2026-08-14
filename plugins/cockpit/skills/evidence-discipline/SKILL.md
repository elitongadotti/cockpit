---
name: evidence-discipline
description: Label every factual claim with its basis — measured, documented, or unverified — and never state a number you did not obtain. Use when writing docs that record constraints, findings, or benchmark results.
when_to_use: writing or editing a constraints/findings doc, recording a benchmark, appending to a compatibility matrix, reporting whether a fix worked
---
# Evidence Discipline

A plausible chain of reasoning is not proof. The failure mode is confident prose:
a claim gets stated in the same tone as a measured fact, nobody can tell them
apart later, and a decision gets made on top of it.

Every factual claim has exactly one basis:

- **Measured** — you ran something and read the output. Cite the command.
- **Documented** — primary documentation says so. Cite the URL.
- **Unverified** — neither. Say so explicitly, or go verify.

If a claim fits none of the three, it is not a finding. Drop it or test it.

## Rules

- **Never write a number you did not obtain.** Run the command or omit the
  figure. "Roughly 2000 partitions" with no run behind it is invented.
- **Label the basis, don't imply it.** Same confident tone for a measurement and
  a guess is the whole problem.
- **Don't delete a superseded finding** in a doc that accumulates them — mark it
  superseded and keep the original. The history is why the doc is trustworthy.
- **Reasoning is not evidence.** "X is order-preserving, so the predicate will
  prune" is a hypothesis. Until a run confirms it, it is unverified.

## Machine-checkable form

A doc can opt into enforcement by declaring the tags it requires in its own
frontmatter. The `evidence-guard` hook then blocks edits that add an unlabeled
claim to that file:

```markdown
---
cockpit_evidence:
  require: ['[DOC]', '[EMPIRICAL]']
  applies_to_lines_matching: '^\|'
---
```

- `require` — the tags that count as a label. Pick whatever fits the doc:
  `[DOC]`/`[EMPIRICAL]`, `[MEASURED]`/`[SOURCE]`, `[RFC]`.
- `applies_to_lines_matching` — optional regex narrowing the rule to the lines
  that actually carry claims (table rows, bullets). Omit to check every added
  line.

Files without this frontmatter are never checked. The opt-in is per file, by
design: a README does not want this rule.

Two tags get extra verification when present in `require`:

- `[EMPIRICAL]` — must correlate with a real command in this session's
  evidence log. Claiming a measurement you never ran is the exact failure this
  guards.
- `[DOC]` — must carry a URL on the same line.

## Why a hook and not just discipline

The tag is a claim about a claim, so on its own it can still be wrong. Pairing
it with the evidence log is what makes it checkable: `[EMPIRICAL]` stops being a
declaration and becomes a statement that correlates with something that
happened. The guard cannot tell whether a finding is *true* — it can only tell
whether a basis was declared and, for measurements, whether anything was
actually run. That is a narrow guarantee, and it is the one worth automating.
