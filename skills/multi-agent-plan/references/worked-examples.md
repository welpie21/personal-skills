# Worked examples

Four calibration examples covering the shapes you'll actually see: a layered feature, a shared-config migration, an embarrassingly-parallel case, and a shared-type fan-out. Match new tasks against the closest shape here rather than starting the reasoning from scratch each time.

## 1. Layered feature (dependency-dominated)

**Task**: "Add a notifications feature: DB migration for a notifications table, a backend API endpoint, a frontend bell component, and update the README."

- Units: `migration` (owns `db/migrations/xxxx_notifications.sql`), `api` (owns `src/api/notifications.ts`, depends on `migration`), `ui` (owns `src/components/NotificationBell.tsx`, depends on `api`), `docs` (owns `README.md`).
- Conflicts: none on owned files. `docs` touches a shared file (conflict-pattern #6) but nothing else is editing `README.md`, so no batching conflict — just flag it as low-risk single-owner.
- Phases: **1** `migration` alone → **2** `api` alone → **3** `ui` + `docs` in parallel (both only depend on `api`, no file overlap).
- Lesson: a strict pipeline (schema → backend → frontend) still has real parallelism at the end — don't stop looking for batching once you've found the obvious dependency chain.

## 2. Shared-config migration (conflict-dominated)

**Task**: "Migrate 15 files from CommonJS to ESM across src/, plus update tsconfig and package.json's `type` field."

- Naive split (5 files/agent) looks fine file-wise, but `tsconfig.json`/`package.json` (conflict-pattern #4/#1) are shared config every file's module resolution depends on.
- Resolution: **sequence**, not parallelize. One unit updates `package.json`'s `"type"` field and `tsconfig.json`'s module settings first, alone.
- Phases: **1** config unit alone → **2** the 15 files split across N agents, each owning a disjoint file list — no cross-file conflict since each file's ESM conversion is independent once config is set.
- Lesson: a file-count-balanced split can still be wrong if it ignores a shared config dependency underneath it. Always check Step 4 even when Step 2's file split looks clean.

## 3. Embarrassingly parallel (no dependency, no conflict)

**Task**: "Write unit tests for our 8 API route handlers."

- 8 units, one per handler, each owning its own `*.test.ts` file. No dependencies (tests don't depend on each other), no file overlap.
- Collapses to a single phase, all 8 in parallel.
- Lesson: don't manufacture phases or shared-ownership concerns where none exist — a flat batch is a valid, common outcome, not a sign you didn't look hard enough.

## 4. Shared-type fan-out (the subtle one)

**Task**: "Add `priority` and `dueDate` fields to the shared `Task` interface, then update API validation, the DB schema, and three frontend components that render tasks."

- Units: `types` (owns the shared `Task` interface file — conflict-pattern #2), `schema` (owns the DB migration adding the columns), `api` (owns validation logic, depends on `types`), `ui-list`, `ui-card`, `ui-detail` (each owns one distinct frontend component that renders a `Task`, each depends on `types` only — they just read the type, they don't need `api` done since they're rendering, not persisting).
- Conflicts: `types` and `schema` don't share a file, so they're independent of each other even though both stem from "add these fields." The three UI units each own a distinct file and only import from `types` — no overlap between them.
- Phases: **1** `types` + `schema` in parallel (independent files, both foundational) → **2** `api` + `ui-list` + `ui-card` + `ui-detail` all in parallel (all four depend only on `types`, which is already done; none share a file).
- Lesson: when several units all stem from one shared change, the shared change itself is a single-owner unit everything else depends on — not something to merge every consumer into, and not something to split by consumer. Once it lands, true consumers that don't depend on *each other* fan out freely, even if they all "came from the same field addition."
