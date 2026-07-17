---
name: hand-off
description: Create a concise handoff.md that lets another agent resume work without rediscovering context. Use when pausing, transferring, or ending an unfinished task, or when a durable progress record is needed for a later agent.
---

# Hand Off

Create or refresh `handoff.md` in the project root, unless the user specifies another location. Do not modify task files as part of preparing the handoff.

Confirm task-relevant facts from the conversation, current plan, working-tree status and diff, relevant files, and commands actually run. Do not present assumptions, intended changes, unrelated working-tree changes, or unrun checks as facts. Do not include secrets or large raw logs.

Write exactly these six level-2 sections, in this order. Do not add a title or other sections to `handoff.md`:

```md
## Goal
## Current State
## Active Files
## Changes Made
## Failed attempts
## Next steps
```

- **Goal:** State the intended outcome and known acceptance criteria.
- **Current State:** State what is complete, in progress, blocked, and unknown. Report validation accurately; call a check passing only when it ran successfully.
- **Active Files:** List task-relevant paths with each file's role and whether it changed, needs review, or is the next place to work.
- **Changes Made:** Record completed, concrete changes and their effect. Include relevant validation and its result.
- **Failed attempts:** Record genuine unsuccessful approaches, errors, or dead ends and why they failed. Write `- None.` when there were none; do not invent failures.
- **Next steps:** Give a short, ordered set of actionable continuation steps. Include dependencies, blockers, and exact commands when known, starting with the most useful action.

If `handoff.md` already exists, replace stale content with a freshly verified handoff instead of appending duplicate history. Re-read the file before finishing and confirm that all six required sections are present, correctly named, and sufficient for another agent to resume without terminal history.
