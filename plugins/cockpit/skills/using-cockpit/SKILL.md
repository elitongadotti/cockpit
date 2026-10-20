---
name: using-cockpit
description: Routing rules for the cockpit skills. Auto-injected at session start so the other skills trigger reliably. Read first.
when_to_use: injected by the session-start hook on startup / clear / compact
---
# Using Cockpit

This project has the **cockpit** plugin installed. Before acting, check whether the task matches one of the skills below and invoke it.

| If the task is... | Use skill |
|---|---|
| A multi-step task / "build X" / long job | `spec-first` — write `PROMPT.md` first |
| A crash, exception, or stack trace | `read-the-trace` |
| A new/changed SQL query or migration | `sql-review` |
| A PySpark job or a query on Iceberg tables | `pycegerb-yoda` |
| Checking a branch against a ticket's acceptance criteria | `acceptance-audit` |
| Committing, or pasting anything somewhere public | `leak-check` |
| Writing a commit message | `writing-commit-messages` |
| Opening a pull request | `pr-from-diff` |
| Reviewing a PR, branch or diff | `pr-review` |
| Long session, big files, accuracy dropping | `context-budget` |
| Ending or resuming a multi-session project | `shift-notes` |
| Starting a session after a "done" feature | `broken-window-check` |

After any code change, invoke the `verifier` agent against the diff and the spec.

Skills are guidance, not gates — apply judgement. If none match, proceed normally.
