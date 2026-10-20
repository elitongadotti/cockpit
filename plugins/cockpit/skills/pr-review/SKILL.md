---
name: pr-review
description: Review a pull request for contradictions, unproven claims, conflicts with the repo's other docs, and risks to recorded architecture decisions — on top of logic bugs.
when_to_use: reviewing a PR, branch or diff; "review this", "check this PR", before approving or merging
---
# PR Review

Hunt for four kinds of defect, in code, comments, docs, the PR body and commit
messages alike. Every finding names which kind it is.

1. **Contradiction.** Two statements in the PR that cannot both be true: code
   vs. its comment, PR body vs. diff, a variable's description vs. its
   validation, a test asserting the opposite of what the docs promise. Quote
   both sides with `file:line`. Do not pick the winner unless one side is
   proven (official documentation, or an executed test/command).
2. **Unproven claim.** A statement backed by neither official documentation nor
   empirical proof — just an assumption. Typical shapes: "this is safe", "the
   cloud provider does X", "no downtime", "never happens", "faster", a number
   nobody measured, or wording too vague to check ("should be fine in most
   cases"). Say what would prove it: the doc to cite, or the test/command to
   run. Either verify it yourself or mark it not verified; never settle it by
   reasoning.
3. **Conflict with the rest of the repo.** A statement in the PR that disagrees
   with a doc outside the diff (README, `docs/`, ADRs, examples, CHANGELOG, the
   repo's `CLAUDE.md`/`AGENTS.md`, other modules' variable descriptions), or a
   behaviour change that leaves another doc describing the old behaviour. Cite
   both locations.
4. **Risk to an architecture decision.** Anything that reverses, bypasses or
   quietly weakens a recorded decision: a new default that undoes it, a
   workaround that makes it moot, coupling the design kept apart, a breaking
   change to an interface consumers rely on. Name the decision, where it is
   recorded, and what in the diff puts it at risk. If the PR means to change
   the decision, the finding is that the record must change in the same PR.

## How to apply

- Read the repo's docs, its `CLAUDE.md`/`AGENTS.md` and the decision records
  for the touched area before writing findings. A review that read only the
  diff cannot catch 3 or 4.
- Rank findings by impact, architecture risk first.
- One paragraph per finding: what breaks, the minimum evidence that proves it,
  the fix in prose. Snippets only when prose cannot express it.
- When a kind turns up nothing, one line says so and names what was read
  ("no conflict with repo docs — read README.md, docs/listeners.md"), so a
  clean result is evidence, not silence.
- These four come on top of the usual review, not instead of it. Logic bugs
  still get reported. Typos and code style do not.

## Related

- `verifier` — checks a diff against the goal spec for fake-done shortcuts.
- `acceptance-audit` — checks the branch against a ticket's acceptance criteria.
- `sql-review`, `pycegerb-yoda` — the query- and engine-specific review layers.
