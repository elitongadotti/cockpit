---
name: code-writer
description: "Delegate boilerplate code generation to a cheap worker model. Use for tests, config, docstrings, type stubs, or any generation where over 80% is predictable from a reference file."
when_to_use: writing tests that mirror an existing test file, scaffolding config, repetitive stubs
---
# Code Writer

Spawn the `code-writer` agent instead of writing the boilerplate yourself:

```
Agent(subagent_type: "code-writer", prompt: "<what to generate>. Reference: <path>. Target: <path>")
```

Always name a reference file — without one the agent produces code that matches
nothing in the project. To build on what it just wrote, use that file as the
reference for the next call.

Review the result and make surgical edits for the small part that needs your
judgement.

---

<!-- Adapted from spotify/portal-ai-plugins (plugins/shunt/skills/code-writer/SKILL.md), Copyright 2026 Spotify AB, licensed under Apache-2.0. Modified: delegation targets a Claude Code subagent instead of an AiKA mode on the Portal CLI. See ../../LICENSE. -->
