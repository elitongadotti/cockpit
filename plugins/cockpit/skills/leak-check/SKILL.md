---
name: leak-check
description: Catch secrets, PII, and non-public context in a diff before it is committed or published. Use before any commit, and before pasting output anywhere public.
when_to_use: before commit, before opening a PR, before publishing a doc/gist/artifact, files with credentials or config, pasting real command output as an example
---
# Leak Check

Scan what is about to leave your machine — the staged diff, or the text you're
about to paste somewhere public. Two categories, different remedies.

## 1. Secrets

Grep the diff for: `api[_-]?key`, `secret`, `token`, `password`, `passwd`,
`credential`, `BEGIN [A-Z ]*PRIVATE KEY`, `AKIA[0-9A-Z]{16}`, `ghp_`, `sk-`,
`Bearer `, connection strings (`://user:pass@host`), and long base64/hex blobs.

For each hit, decide: real secret or placeholder?

1. **Real secrets move to env or a secrets manager** — never the repo. Reference
   them by variable name.
2. **Already committed means COMPROMISED.** Rotate it. Deleting the line does not
   help — the value is in the history, on every clone, and in any fork or CI cache.
   Rotation is the fix; removing the line is cleanup.
3. **Add a pre-commit secret scanner** so the next one is caught mechanically, and
   put the file pattern in `.gitignore`.

## 2. Non-public context

Quieter than secrets and just as hard to retract once published:

- **PII** — real names, emails, phone numbers, customer or employee identifiers.
- **Absolute and personal paths** — `/Users/<name>/…`, `~/projects/<repo>`. These
  also just break for everyone else.
- **Internal infrastructure** — hostnames, private URLs, IPs, account/project IDs,
  ARNs, cluster names.
- **Employer-specific names** — tables, buckets, datasets, services, teams,
  internal ticket IDs.
- **Machine-local context** — env dumps, cloud/CLI config files, transcripts, raw
  logs.

Replace with placeholders that still communicate shape: `user@example.com`,
`<account-id>`, `db.events`, `${PLUGIN_ROOT}`.

## Where it actually leaks

The diff is the obvious surface. These are the ones people miss:

- **Real command output pasted as an example** in a README, docstring, or skill —
  the most common leak in documentation.
- **Test fixtures** built from a production export.
- **Commit messages and PR descriptions** — not part of the diff, still public.
- **Screenshots** — a terminal or browser tab captures far more than the subject.
- **Error messages and stack traces** that embed a full path or a query with real
  values.
- **Lockfiles and configs** pointing at a private registry with a token in the URL.

## Judgement

Public-vs-private is contextual, and reversibility is asymmetric: leaving a detail
out costs a follow-up question, publishing one cannot be undone. When unsure,
redact and ask.

## Source

Adapted from `secret-scan` in
[loopkit](https://github.com/Archive228/loopkit) by Archive228 (MIT, © 2026 —
notice in [LICENSE](../../../../LICENSE)). The secret patterns and the
rotate-don't-delete rule come from there; the non-public-context section and the
leak surfaces above are additions.

## Pairs with

- `writing-commit-messages` — the message is public too.
- `pr-from-diff` — same review, at PR scope.
