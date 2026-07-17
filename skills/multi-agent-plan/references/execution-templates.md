# Execution templates

Concrete patterns for Step 8 — turning an approved plan into actual tool calls. Pick based on shape, not preference: a handful of agents in one or two phases fits a plain `Agent` call; several phases or enough units that the fan-out itself is worth scripting fits `Workflow`.

## Single batch, `Agent` tool

Every agent in a batch goes in the **same message**, as multiple tool calls, so they run concurrently — not one at a time in separate messages. Each prompt must state the unit's scope, its owned files (so it doesn't wander into another unit's territory), and anything it must NOT touch (the shared resources Step 4/5 flagged).

```
Agent({
  description: "Notification bell UI",
  prompt: "Implement the notification bell component in src/components/NotificationBell.tsx, consuming the API at src/api/notifications.ts (already implemented — read it, don't modify it). Scope is this one file only; do not touch README.md or any other component.",
})
Agent({
  description: "Notifications README update",
  prompt: "Add a short section to README.md describing the new notifications feature (bell UI + API). Scope is README.md only.",
})
```

When Step 5 called for **worktree isolation** on a unit (mechanical overlap on a shared file), add `isolation: "worktree"` to that call:

```
Agent({
  description: "Add priority field to task list view",
  prompt: "...",
  isolation: "worktree",
})
```

After a worktree-isolated batch completes, do the **reconciliation step** yourself (or in one small follow-up agent) rather than assuming git merged it cleanly — read what each worktree actually changed in the shared file, and combine by hand if the auto-merge left conflict markers or silently picked one side.

## Multiple phases, plain sequential `Agent` calls

If there are only two or three phases and no need to script it, just run each phase's batch, wait for it to complete, then run the next phase's batch in a following message. Don't start phase 2's agents until phase 1's have actually returned — that's the whole point of the dependency graph from Step 3.

## Multiple phases or many units, `Workflow` tool

Only use this when the user has separately opted into multi-agent orchestration (the `Workflow` tool's own gating rule) — this skill producing a plan doesn't itself satisfy that. When it applies, model each plan-phase as a `parallel()` call and run phases in sequence:

```js
export const meta = {
  name: 'notifications-feature-split',
  description: 'Execute the approved notifications feature work-split plan',
  phases: [
    { title: 'Phase 1: migration' },
    { title: 'Phase 2: api' },
    { title: 'Phase 3: ui + docs' },
  ],
}

phase('Phase 1: migration')
await agent('Write the DB migration for the notifications table in db/migrations/. Scope: that one new file only.', { label: 'migration' })

phase('Phase 2: api')
await agent('Implement the notifications API endpoint in src/api/notifications.ts, using the migration already landed. Scope: that one file only.', { label: 'api' })

phase('Phase 3: ui + docs')
await parallel([
  () => agent('Implement the notification bell in src/components/NotificationBell.tsx, consuming the existing API. Scope: that file only.', { label: 'ui', phase: 'Phase 3: ui + docs' }),
  () => agent('Add a README section describing the notifications feature. Scope: README.md only.', { label: 'docs', phase: 'Phase 3: ui + docs' }),
])
```

For a batch containing a worktree-isolated unit, pass `opts.isolation: 'worktree'` on that specific `agent()` call, same as the `Agent` tool. If a later phase needs to read back what a worktree-isolated agent produced (rather than it being merged automatically), have that agent return the changed content via a `schema`, and thread it into the next phase's prompt instead of assuming it landed in the shared tree.

## Reconciliation step template

For barrel files, registration points, or shared docs resolved via isolate + reconcile (see `references/conflict-patterns.md` #6, #7, #10, #11): after the batch, run one small agent (or do it yourself) whose only job is to read what every unit in that batch actually added and combine it into the shared file in one pass:

```
Agent({
  description: "Reconcile barrel file exports",
  prompt: "Units A, B, and C each added a new module: src/widgets/foo.ts, src/widgets/bar.ts, src/widgets/baz.ts. Add one export line for each to src/widgets/index.ts, keeping existing alphabetical order. Do not modify anything else in the file.",
})
```

This is deliberately a single agent, not another parallel batch — the reconciliation is the one place where a single owner is unavoidable.
