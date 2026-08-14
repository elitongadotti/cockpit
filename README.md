# cockpit

My personal marketplace of Claude Code (and Codex) plugins — skills, hooks and
agents I reuse across the projects I work on.

I based this repo on the:
- [loopkit](https://github.com/Archive228/loopkit) — MIT, © 2026 Archive228
- [ai-tools](https://github.com/it-bens/ai-tools) — MIT, © 2025 Martin Bens

So, yes, its not an original repo in its core ~~as everything else nowadays~~. And it still is the skillset I use as my day-to-day performance boost.

`cockpit` is MIT too — see [LICENSE](LICENSE), which carries both upstream
copyright notices.

## Install

Non-interactive (both default to **user** scope, so it loads in every project):

```bash
claude plugin marketplace add elitongadotti/cockpit   # or a full git URL / local path
claude plugin install cockpit@cockpit
```

Pass `--scope project` (shared via the repo's `.claude/settings.json`) or
`--scope local` (just you, just this repo) to scope it to one project — run it
from inside that project. Then `/reload-plugins` (or start a fresh session) to
activate; verify with `claude plugin list`.

The equivalents inside an interactive `claude` session are the `/plugin
marketplace add` and `/plugin install` slash commands, which prompt for scope.

Test locally without installing:

```bash
claude --plugin-dir ./plugins/cockpit
```

`--plugin-dir` is dev-only: it loads the plugin for **that one session, in that
one directory**, and nothing persists. Use the install steps above for day-to-day
use across projects.

## Build a plugin

A plugin is a folder with a manifest. Minimum viable plugin:

```
plugins/my-plugin/
└── .claude-plugin/
    └── plugin.json
```

**1. Manifest** — `.claude-plugin/plugin.json`:

```json
{
  "name": "my-plugin",
  "description": "What it does, in one line.",
  "version": "0.1.0",
  "author": { "name": "Your Name" },
  "license": "MIT"
}
```

**2. Add a skill** — `skills/<name>/SKILL.md` (auto-loads when the
`description` matches the task; you don't invoke it):

```markdown
---
name: my-skill
description: One line — this is the trigger the model matches on.
---
# My Skill
Short, imperative guidance the model follows.
```

**3. Add an agent** — `agents/<name>.md` (a subagent you can delegate to):

```markdown
---
name: my-agent
description: When to use this agent.
model: haiku
tools: [Read, Grep, Bash]
---
System prompt for the subagent.
```

**4. Add a hook** — `hooks/hooks.json` (the harness runs these on lifecycle
events; format is identical to `settings.json`):

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "\"${CLAUDE_PLUGIN_ROOT}/hooks/fmt\"" }
        ]
      }
    ]
  }
}
```

Referenced scripts sit next to `hooks.json` and must be executable
(`chmod +x`). Use `${CLAUDE_PLUGIN_ROOT}` for paths — never hardcode.

**5. Register it** in `.claude-plugin/marketplace.json`:

```json
{
  "plugins": [
    { "name": "my-plugin", "source": "./plugins/my-plugin", "version": "0.1.0" }
  ]
}
```

## Skill vs. hook (rule of thumb)

- **Skill** = *how to do task X when it comes up* — Markdown; the model decides
  to invoke it. Probabilistic.
- **Hook** = *always do Y on event Z* — JSON + script; the harness runs it every
  time. Deterministic.

## Codex compatibility

Each plugin also ships a `.codex-plugin/plugin.json` and is listed in
`.agents/plugins/marketplace.json`. See [AGENTS.md](AGENTS.md) for the full
compatibility contract.

## What's inside `cockpit`

| Kind  | Name | Purpose |
|-------|------|---------|
| skill | `spec-first` | Write `PROMPT.md` before a multi-step task |
| skill | `read-the-trace` | Root-cause from a stack trace |
| skill | `sql-review` | Review SQL/ORM for safety + perf |
| skill | `pycegerb-yoda` | PySpark + Iceberg: writes, schema, maintenance |
| skill | `acceptance-audit` | Audit a ticket's acceptance criteria against the branch |
| skill | `leak-check` | Catch secrets, PII and internal context before commit |
| skill | `writing-commit-messages` | Commit message about the focus, not the scaffolding |
| skill | `pr-from-diff` | Write an approvable PR description |
| skill | `context-budget` | Keep context lean on long sessions |
| skill | `shift-notes` | Between-session handoff file |
| skill | `broken-window-check` | Smoke-test last "done" feature |
| skill | `evidence-discipline` | Label a claim's basis: measured, documented, or unverified |
| skill | `using-cockpit` | Routing rules (injected at session start) |
| agent | `verifier` | Reviews a diff assuming it's broken |
| hook  | `session-start` | Injects `using-cockpit` as context |
| hook  | `pre-compact` | Captures decisions before compaction |
| hook  | `evidence-log` | Records every command run to `.cockpit/evidence.log` |
| hook  | `evidence-guard` | Blocks unlabeled claims in docs that opt in |

## The evidence guard

`evidence-guard` is the one hook here that can **deny** a tool call, so it is
deliberately narrow: it only acts on a file that asks for it, in that file's own
frontmatter.

```markdown
---
cockpit_evidence:
  require: ['[DOC]', '[EMPIRICAL]']
  applies_to_lines_matching: '^\|'
---
```

A file without that block is never checked — which is every file by default. Pick
whatever tag vocabulary suits the doc (`[MEASURED]`, `[SOURCE]`, `[RFC]`); only
two get extra verification when you list them:

- `[EMPIRICAL]` — must correlate with a command in `.cockpit/evidence.log`,
  written by the `evidence-log` hook. Claiming a measurement nothing ran is the
  failure this exists to catch.
- `[DOC]` — must carry a URL on the same line.

Only lines an edit **adds** are checked, so it works on docs that predate it.
Blank lines, headings, fences and table separators are skipped.

**This is a safety net, not a security boundary.** Anything with write access can
edit the frontmatter it reads, and the guard fails open on a malformed payload,
a missing `jq`, or an unreadable file. It catches an honest omission — that is
the whole claim. `.cockpit/evidence.log` records commands, so treat it like any
shell history: keep it out of git.
