---
name: bulk-reader
description: "Delegate bulk file reading to a cheap worker model. Use when you need to read files over 350 lines, answer questions across 3+ files, or summarize a large diff."
when_to_use: a read hook blocked a large file, a question spans several files, a diff is too big to read directly
---
# Bulk Reader

Do not read the file yourself. Spawn the `bulk-reader` agent instead:

```
Agent(subagent_type: "bulk-reader", prompt: "<question>. Files: <path> [<path> ...]")
```

The agent runs on a cheaper model and reads the files in its own context, so only
its answer comes back to you. Ask a follow-up by spawning it again with the same
paths — re-reading costs you nothing here.

Verify exact line numbers and values with a targeted `Read` (offset/limit) before
you use them in an edit.

---

<!-- Adapted from spotify/portal-ai-plugins (plugins/shunt/skills/bulk-reader/SKILL.md), Copyright 2026 Spotify AB, licensed under Apache-2.0. Modified: delegation targets a Claude Code subagent instead of an AiKA mode on the Portal CLI. See ../../LICENSE. -->
