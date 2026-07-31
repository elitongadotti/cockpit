---
name: acceptance-audit
description: Audit a ticket's acceptance criteria against what the branch actually implements. Every item must be satisfied, and each must semantically match the implementation. Never edit or drop a criterion without the user's agreement.
when_to_use: before marking a ticket done, before opening a PR, when a reviewer asks "does this meet the AC?", after implementing against a ticket/gh issue
---
# Acceptance Audit

The acceptance criteria are a **contract you did not write**. They came from a
ticket, and someone else — a reviewer, a PM, whoever filed it — is relying on
them meaning what they say. Your job is to report whether the branch satisfies
them, not to make them satisfiable.

Two independent checks. An item passes only if it survives both.

## Check 1 — Is it satisfied?

For every item in the list, find the code that implements it and the evidence
that it works.

- **Cite location and evidence.** `path:line` for the implementation, plus how you
  confirmed it — the test that covers it, or the command you ran and its output.
- **Exercise the real path where you can.** A criterion is written from the user's
  point of view; a passing unit test on a handler is weaker evidence than driving
  the actual route, CLI, or UI. See `broken-window-check`.
- **No partial credit.** "Mostly done", "works except for empty input", "the happy
  path is covered" — all of those are **not satisfied**. Say so plainly.
- **Absent is not satisfied.** If you cannot find the implementation, the item
  fails. Do not infer it from an adjacent feature or from intent in the diff.

## Check 2 — Does it semantically match?

This is the check that gets skipped, and it catches the expensive failures. Code
can satisfy the *letter* of a criterion while doing something the author did not
ask for.

Compare what the criterion **means** against what the implementation **does**:

| Divergence | Example |
|---|---|
| Different scope | AC says "users can export their own data"; code exports all data for any authenticated user. |
| Different trigger | AC says "on save"; code runs on page unload. |
| Different actor | AC says "an admin can approve"; code lets any logged-in user approve. |
| Different failure mode | AC says "reject invalid input"; code silently coerces it. |
| Different unit or scale | AC says "within 5 seconds"; code has a 5-minute timeout. |
| Vacuous satisfaction | AC says "no duplicate records"; code satisfies it by rejecting every write. |
| Right output, wrong meaning | AC says "show the user's last login"; code shows the current session start. |

An item that is implemented but semantically divergent is **not a pass**. It is a
finding — often a more important one than an unimplemented item, because it looks
finished and will ship.

## The rule about editing the list

**Never delete, reword, narrow, or split an acceptance criterion on your own** —
not to reflect "what was actually built", not to resolve an ambiguity, not to
make a report come out green.

Rewriting the contract to match the implementation is the most consequential
version of fake-done: it destroys the record of what was asked for, and nobody
downstream can tell it happened. The ledger is not yours to edit.

When a criterion is ambiguous, untestable, obsolete, or contradicted by another
item, that is a **finding to report** — not a thing to fix silently. Say what is
wrong, propose the change, and let the user decide. Only edit the list after they
have explicitly agreed, and quote their decision when you do.

## Reporting — and when to stop

Produce a verdict per item:

```
AC-1  PASS       src/export.ts:42 — verified: `npm test -- export` (3 passing)
AC-2  FAIL       not implemented; no handler for the empty-cart case
AC-3  DIVERGENT  src/auth.ts:88 — AC says "an admin can approve", code accepts any
                 authenticated user. Scope is wider than asked.
AC-4  UNCLEAR    "must be fast" — no threshold given; cannot be verified as written
```

Then:

- **All PASS** → say so, with the evidence, and continue.
- **Any FAIL, DIVERGENT, or UNCLEAR** → **stop and tell the user.** Do not open the
  PR, do not mark the ticket done, and do not start closing the gaps until they
  have seen the list.

Say plainly which of the two checks failed and what the decision in front of them
is: change the code, change the criterion (their call, not yours), or accept the
gap deliberately. Recommend one — but the decision is theirs, and a divergence
between a ticket and a branch is frequently a sign that the ticket is the thing
that is wrong.

## Pairs with

- `spec-first` — the contract you write yourself; this audits one handed to you.
- `broken-window-check` — how to exercise a feature end-to-end for real evidence.
- `pr-from-diff` — the audit result is what the PR's "how to verify" section needs.
- `verifier` agent — run it on the diff for fake-done shortcuts, then audit the AC.
