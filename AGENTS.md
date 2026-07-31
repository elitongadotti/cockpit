@README.md

# Development Guide

`cockpit` is a personal marketplace. Each plugin lives under `plugins/<name>/`
and is registered in both marketplace registries (Claude + Codex).

## Layout

```
cockpit/
├── .claude-plugin/marketplace.json     # marketplace registry (Claude host)
├── .agents/plugins/marketplace.json    # marketplace registry (Codex host)
├── plugins/<name>/                     # one folder per plugin
└── README.md                           # how to build a plugin (start here)
```

Each plugin:

```
plugins/<name>/
├── .claude-plugin/plugin.json          # manifest (Claude host)
├── .codex-plugin/plugin.json           # manifest (Codex host)
├── skills/<skill>/SKILL.md             # skills — auto-load on trigger
├── agents/<agent>.md                   # subagents
└── hooks/hooks.json                    # hook wiring (+ scripts alongside)
```

## Compatibility Contract

Plugins may support Claude Code, Codex, or both. Claim support only when the
plugin has the manifest, marketplace registration, and runtime wiring for
that host.

| Host        | Plugin manifest              | Marketplace registry             |
|-------------|------------------------------|----------------------------------|
| Claude Code | `.claude-plugin/plugin.json` | `.claude-plugin/marketplace.json` |
| Codex       | `.codex-plugin/plugin.json`  | `.agents/plugins/marketplace.json` |

## Conventions

- One SKILL.md per skill; frontmatter needs `name` + `description`
  (`when_to_use` optional). Keep bodies short — the description is the trigger.
- Cross-skill links use the skill `name`; don't reference a skill this
  marketplace doesn't ship (no broken `[[links]]`).
- Hook scripts must be executable (`chmod +x`) and reference
  `${CLAUDE_PLUGIN_ROOT}` for paths, never a hardcoded absolute path.
- Bump `version` in BOTH `plugin.json` files together.

## Adding a Skill From a Source

When a skill is based on an external source, do both of these before writing it.

**1. Validate the license.** Fetch it — never assume:

```bash
curl -sS https://api.github.com/repos/<owner>/<repo>/license | jq -r .license.spdx_id
```

| Upstream license | What you may do |
|---|---|
| MIT / Apache-2.0 / BSD | Copy text freely; reproduce their notice in `LICENSE`. |
| GPL / AGPL | **Do not copy text** into this MIT repo — copyleft would force the whole repo to GPL. Read it to learn the technique, then write independently from primary docs. |
| No license | All-rights-reserved by default. Treat as GPL above. |

Reading a repo to survey its subject matter is always fine. Only copying its
literal expression is restricted — ideas, techniques and structure are not.

**2. Cite the source** in a `## Source` section of the skill: the upstream repo,
its license, and whether text was copied or the skill was written independently.

## Skills Are Public and Load Everywhere

Everything under `plugins/` is published publicly, and **user scope is the
install default** — so assume a skill loads in every project the user opens,
including work repos. Project and local scope exist (see [README](README.md#install)),
but write for the broadest case, since you don't control how it gets installed.

- **Nothing employer- or repo-specific.** No internal table, bucket, cluster,
  dataset or service names. Write for any reader.
- **Only documented, verifiable practice.** Don't invent config keys, API options
  or version numbers. If a claim can't be traced to primary documentation, leave
  it out — and prefer linking the doc over restating a value that may drift.
- Mark version-sensitive defaults as such.

### Never Commit — Veto Before Staging

If content is not reusable by a stranger in an unrelated project, it does not
belong in `cockpit`. **Block it before it is staged, and say why** — a secret in
git history survives deleting the file, and this repo is public.

| Never | Instead |
|---|---|
| Secrets: API keys, tokens, passwords, connection strings, certificates | Name the env var: `$SNOWFLAKE_PASSWORD` |
| PII: names, emails, phone numbers, customer or employee identifiers | Placeholders: `user@example.com`, `<customer-id>` |
| Absolute or personal paths: `/Users/<name>/…`, `~/projects/<repo>` | `${CLAUDE_PLUGIN_ROOT}`, or a relative path |
| Internal hostnames, URLs, IPs, account/project IDs, ARNs | `<region>`, `<account-id>`, `example.internal` |
| Employer-specific names: tables, buckets, clusters, datasets, services, teams, ticket IDs | A generic example: `db.events` |
| Machine-local context: shell env dumps, `~/.aws/config`, cluster configs, transcripts, logs | Describe the shape; don't paste the file |
| Non-public knowledge: internal docs, private repo contents, unreleased plans | Cite public documentation instead |

**Applies to everything committed here** — skill bodies, hook scripts, agent
prompts, README, and example code and command output inside them. Real output
pasted as an example is the usual leak.

When something is genuinely needed but unsafe to publish, put it in local config
(`.claude/settings.local.json`) or per-user memory — not in `plugins/`. If you
are unsure whether a detail is public, leave it out and ask.
