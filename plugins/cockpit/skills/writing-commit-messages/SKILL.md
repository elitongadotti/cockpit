---
name: writing-commit-messages
description: Write a commit message that describes the focus of the change, not its supporting work. Use when committing.
when_to_use: writing a commit, "commit this", amending a message, squashing
---
# Writing Commit Messages
The subject line answers "what changed and why", in the imperative. The body is
only for what the subject can't hold: the reason, the trade-off, the thing that
will look wrong to whoever reads the diff later.

**Describe the focus, not the scaffolding.** When the commit isn't itself a test
or docs change, keep test and doc edits out of the subject and body — they're
supporting work.

- Tests: `*_test.*`, `*.bats`, `tests/`, `test/`.
- Docs: `*.md`, `README*`, `docs/`, and comment-only diffs.
- Never append "and tests", "+ docs", "with documentation" to a subject. No body
  paragraph enumerating test or doc files.
- If a test or doc change *explains* the focus — a test isolating the bug being
  fixed — describe the behavior, not the artifact.

No "fix stuff", no "wip", no restating the diff line by line. If you can't name
the focus in one line, the commit is probably two commits.

## Pairs with

- `pr-from-diff` — same discipline, larger unit: say what a reader needs, skip the noise.
