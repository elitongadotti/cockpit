# shunt

My version of [Spotify's shunt plugin](https://github.com/spotify/portal-ai-plugins),
rebuilt to run on Claude Code subagents instead of Spotify's Portal platform.

It keeps bulk file reading and boilerplate writing out of the main model's
context by handing that work to a cheap model. The savings come from the file
contents never reaching the expensive model at all — only a short answer does.

## How it works here

Three pieces, in the order they fire:

1. **Hook.** `check-file-size` (on `Read`) and `check-bash-read` (on `Bash`)
   count the target file's lines. Past `SHUNT_MIN_LINES`, default 350, they deny
   the call and name the skill to use.
2. **Skill.** The denial points the main model at `bulk-reader` or `code-writer`.
   Each skill is a few lines telling it to spawn an agent instead of doing the
   work itself.
3. **Agent.** `agents/bulk-reader.md` and `agents/code-writer.md` both declare
   `model: haiku`. The agent reads the files in its own context and returns only
   its answer.

Targeted work is never blocked: a `Read` with `offset` or `limit`, a piped
`cat file | grep`, and a redirect `cat file > out` all pass through. Subagents
are exempt too — otherwise the hook would block the very agent it asked for.

## Configuration

`SHUNT_MIN_LINES` sets the line count above which a read is redirected. Default
350. Set it lower to delegate more, higher to delegate less.

To change the worker model, edit `model:` in the two files under `agents/`.

## Requirements

`jq`, for the hooks to parse their input. Nothing else — the agents run on your
existing Claude Code session, so there is no API key to manage.

## Attribution

Derived from [`spotify/portal-ai-plugins`](https://github.com/spotify/portal-ai-plugins)
(`plugins/shunt`), Copyright 2026 Spotify AB, licensed under Apache-2.0. Full
license text in [LICENSE](LICENSE).

The technique is described by Dimitri Mazmanov in
[Portal by Spotify cut my Claude Code token usage by 90%](https://engineering.atspotify.com/2026/9/portal-by-spotify-cut-my-claude-code-token-usage-by-90)
(Spotify Engineering, 3 September 2026), which reports the 90% figure against
Spotify's own setup — not this port.

What changed: Spotify's version shells out to the Portal CLI, which runs the
work on an AiKA mode inside their platform. This one spawns a Claude Code
subagent pinned to Haiku, so the shell scripts and the API plumbing are gone.
The hooks also emit the documented `hookSpecificOutput.permissionDecision`
shape, fail open when `jq` is missing, and skip subagents. Each modified file
carries a note in its own header.
