---
name: multi-agent-plan
description: >-
  Analyzes a task, feature, or plan and produces an optimized plan for
  splitting the work across multiple agents (via the Agent tool or Workflow
  tool) so they can run in parallel without clashing. Builds a dependency
  graph to determine sequencing, and a file/resource conflict map to
  guarantee agents in the same parallel batch never touch the same files,
  shared types, config, or generated artifacts (lockfiles, migrations,
  schema) at the same time — recommending git worktree isolation or
  re-sequencing when overlap is unavoidable. Produces a concrete phased plan
  (parallel batches of agent assignments, each with scope, owned files,
  dependencies, and isolation strategy) and only spawns agents once the user
  approves the split. Use this whenever the user wants to parallelize,
  divide, split, or fan out a task across multiple agents/subagents, asks
  how to break up work so agents don't conflict or step on each other, wants
  to speed up a big feature by working on pieces simultaneously, or is about
  to launch several Agent/Workflow calls on related work and needs to know
  how to shard it safely.
---

# Multi-Agent Work Split

Turn one task into a plan that several agents can execute at the same time without stepping on each other. The output is a **plan**, not a fait accompli — spawn nothing until the user has seen the split and approved it. Launching a fleet of agents costs real time and tokens, and a bad split (two agents editing the same file, or an agent starting before its dependency lands) is more expensive to untangle than it would have been to just do the work serially.

The whole point of this skill is the conflict analysis. Splitting work evenly by line count or file count is easy and usually wrong — the split that matters is the one where no two agents in the same batch write to the same place at the same time, and nothing runs before what it depends on is done.

Three reference files back this up — skim `references/worked-examples.md` once to calibrate on realistic plans, then reach for the other two as each step needs them:
- `references/conflict-patterns.md` — a catalog of recurring shared-resource clashes (lockfiles, shared types, migrations, barrel files, registration points, …) and the resolution each calls for. Use it during Step 4/5 instead of reasoning about overlap from scratch.
- `references/execution-templates.md` — concrete `Agent`/`Workflow` call shapes for Step 8, including worktree isolation and the post-batch reconciliation step.

## Step 1 — Understand the actual task

Read enough of the relevant code/docs to know what "done" looks like and which files are in play. Don't split a task you only know from its one-sentence description — a plan built on guessed file boundaries will misplace the conflicts it's supposed to catch. For a large or unfamiliar codebase, use the Explore agent or a quick `grep`/`find` pass first.

If the task is genuinely small (touches one or two files, or is a few minutes of work), say so and recommend doing it directly instead of splitting — the coordination overhead of multiple agents isn't worth it below a certain size.

## Step 2 — Decompose into work units along real seams

Break the task into units along boundaries that already exist in the codebase or domain, not arbitrary even chunks:

- Separate packages/services/modules
- Separate layers (schema/migration → backend/API → frontend/UI → docs)
- Separate features or routes/components that don't share internals
- Tests vs. implementation, when they can genuinely proceed independently
- Independent files within a directory, if nothing about their content couples them

A good work unit has a name, a one-sentence scope, and a concrete set of files/directories it owns. If two candidate units would both need to touch the same core file to do their job, that's a signal they may not be separable — don't force a split.

## Step 3 — Map dependencies (the DAG)

For every pair of units, ask: does one need the other's output to start or to compile/run correctly? Typical ordering: shared types/schema → backend logic → API surface → frontend/consumers → docs. Record this as a dependency graph — it's what turns the flat unit list into ordered phases later (topological order: nothing starts before everything it depends on has landed).

Don't invent dependencies that aren't real just to be cautious — false dependencies serialize work that could have run in parallel, defeating the purpose of splitting it up.

## Step 4 — Map conflicts (the "do not clash" guarantee)

This is separate from dependencies and just as important: two units can have zero dependency relationship and still clash if they'd write to the same file, or if they'd want to add to the same shared type definition, config file, README/CHANGELOG, or generated artifact (lockfile, schema migration, snapshot).

For every pair of units with no dependency edge, check for file/resource overlap. Be specific — name the actual file path each unit would touch, don't reason about it in the abstract.

Flag high-risk shared resources explicitly rather than letting them slide into a parallel batch:

- `package.json`/lockfiles, `go.mod`, `Cargo.toml`, and similar manifests
- Shared type/interface definitions that multiple units extend
- Shared config files
- Migrations/schema files where ordering or numbering matters
- A shared README or top-level doc multiple units would each want to append to
- Barrel/index files and central registration points (routers, DI containers, reducer registries)

These aren't exhaustive — see `references/conflict-patterns.md` for the full catalog (14 patterns) with the resolution each one calls for, and worked examples of each in `references/worked-examples.md`.

## Step 5 — Resolve every conflict before batching

For each conflicting pair, pick one:

1. **Merge** the two units into one agent's scope, if the overlap is central to both — splitting them was never really viable.
2. **Sequence** them — one unit owns the shared file and finishes first; the other depends on it (this converts a conflict edge into a dependency edge).
3. **Isolate** with a git worktree so each agent mutates its own copy of the shared file, and add an explicit merge/reconciliation step after the batch to combine the results. Use this only when the overlap is small and mechanical (e.g. two agents each adding one independent entry to the same list) — if the merge would require real judgment, prefer option 2. In this environment: the `Agent` tool takes `isolation: "worktree"`, and `Workflow`'s `agent()` takes `opts.isolation: 'worktree'`.

Never leave a flagged conflict unresolved and silently put both units in the same parallel batch — that's the exact failure mode this skill exists to prevent.

## Step 6 — Group into parallel batches

A unit can join a batch only if every dependency it has is already satisfied by an earlier batch, and it has no unresolved conflict with anything else in the same batch. This naturally produces phases: batch 1 is everything with no dependencies, batch 2 is everything that only depended on batch 1, and so on. Units within a batch are the ones that actually get spawned together in parallel; batches themselves run in order.

## Step 7 — Present the plan

Show the plan before doing anything else. Use this shape (adapt freely, but keep phases and per-agent scope/files/isolation explicit):

```markdown
## Work-split plan: <task>

### Work units
| ID | Scope | Owns (files/dirs) | Depends on |
|----|-------|--------------------|------------|
| A  | ...   | ...                | —          |
| B  | ...   | ...                | A          |

### Conflicts found & resolution
- A ↔ C both touch `shared/types.ts` → merged into a single unit
- B ↔ D both touch `package.json` → sequenced, B owns it, D depends on B

### Phases
**Phase 1 (parallel)**
- Agent A — <scope> — owns `path/one`, `path/two` — shared working tree
- Agent C — <scope> — owns `path/three` — shared working tree

**Phase 2 (parallel, after Phase 1)**
- Agent B — <scope> — owns `path/four` — worktree isolation (touches shared config; reconcile after)

### Notes
- <any risk called out in Step 4 that's worth the user's attention>
```

If the whole task collapses into a single phase with one unit, say so plainly — that's a valid outcome of this analysis, not a failure to find parallelism.

## Step 8 — Only after approval, execute

Once the user confirms the split (or adjusts it), help kick it off using the ordering and isolation decisions from the plan — don't re-derive them:

- A single batch of a few agents in the current turn → the `Agent` tool, one call per unit, all in the same message so they run concurrently; add `isolation: "worktree"` exactly where Step 5 called for it.
- Multiple phases, or enough units that scripting the fan-out is worth it → the `Workflow` tool, using `parallel()` for each batch and running batches in sequence (or `pipeline()` if later phases can start per-unit rather than waiting for the whole batch) — but only when the user has separately opted into multi-agent orchestration per that tool's own rules.

See `references/execution-templates.md` for concrete call shapes for both, plus the reconciliation-step pattern for any unit Step 5 resolved via worktree isolation.

If the user only wanted the plan, stop after Step 7.
