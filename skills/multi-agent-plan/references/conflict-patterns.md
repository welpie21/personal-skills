# Conflict patterns catalog

Concrete shapes that shared-resource conflicts take in real codebases, and the resolution each one calls for (merge / sequence / isolate — see SKILL.md Step 5). Use this when doing Step 4's conflict map instead of reasoning about overlap from first principles each time — most conflicts you'll hit are one of these.

The recurring judgment call: **is the shared edit mechanical (each unit adds one independent, non-overlapping entry) or does it require judgment (the units actually need to agree on shape/naming/ordering)?** Mechanical edits are good worktree-isolation candidates with a trivial merge step. Judgment-heavy edits should be sequenced behind a single owner instead — a merge step can't manufacture agreement that didn't happen.

## 1. Package manifests & lockfiles
`package.json`, `package-lock.json`/`yarn.lock`/`pnpm-lock.yaml`, `Cargo.toml`/`Cargo.lock`, `go.mod`/`go.sum`, `requirements.txt`/`poetry.lock`.

Lockfiles are regenerated, not hand-merged — two agents installing different dependencies in parallel will produce lockfiles that silently clobber each other. **Sequence**: one unit (often the first phase, alone) adds all newly-needed dependencies up front; everyone else just imports what's already there.

## 2. Shared type/interface/schema definitions
A `types.ts`/`models.py` file, a GraphQL schema, a `.proto` file, a shared DTO — anything multiple units need to extend with new fields or new types.

If several units each need to add unrelated fields/types to the same definition file, that's still one file changing shape underneath everyone. **Sequence**: one unit owns the definition change and lands first; consumers depend on it. Do not isolate-and-merge this one — field additions from two directions often need the same line region (e.g. both adding to the same interface body), which is exactly what worktree isolation *can't* silently reconcile. See `references/worked-examples.md` for a fan-out plan built around this.

## 3. Database migrations / schema files
Migration files are frequently numbered or timestamp-ordered (`0007_add_x.sql`), and two agents generating "the next migration" in parallel will collide on the number or leave the schema in an order-dependent inconsistent state. **Sequence**: run migration-writing units one at a time, or **merge** them into a single "schema" unit if the feature needs more than one migration.

## 4. Shared config files
`tsconfig.json`, `.eslintrc`, build config (`webpack.config.js`, `vite.config.ts`), CI YAML.

Usually only needs a change if the task requires it (a new path alias, a new lint rule, a new CI step). **Sequence** it as a small, single-owner unit ahead of whatever depends on the new config — don't let two units both patch the same config independently even if their intended changes don't obviously overlap; config files are unusually sensitive to ordering and duplicate keys.

## 5. Generated / derived artifacts
An OpenAPI spec generated from route code, a generated GraphQL client, snapshot test files, a generated `.d.ts`.

These regenerate from source — the source (route definitions, GraphQL operations) is the real unit of work, and the generated artifact is a side effect, not something to hand-split. **Merge** work touching the same generation source into one unit, and treat regeneration as a fast final step (yours, or a dedicated last-phase unit) rather than something parallel agents each run.

## 6. Shared README / CHANGELOG / top-level docs
Multiple units each want to add a bullet describing what they did.

Low risk in principle (mechanical, append-only) but still a single file every unit would touch. **Isolate + reconcile** if you want per-unit doc updates in parallel, or simpler: **single-owner** it — have one unit (or you, the orchestrator) write the doc update once at the end, informed by what every other unit actually did.

## 7. Constants, feature flags, i18n translation files
A shared `constants.ts`, a feature-flag config, a `en.json` translation file multiple units need new keys in.

Same shape as #2 but usually lower-stakes since additions are more independent (each unit's new key is unlikely to semantically depend on another's). **Isolate + reconcile** is often fine here if each unit's addition is a genuinely distinct, non-overlapping key — check that two units don't pick the same key name.

## 8. Shared test fixtures / mock data / seed data
A shared `fixtures.ts` or DB seed script multiple test-writing units would extend.

Mechanical if each unit adds its own fixture; risky if units need to extend the *same* fixture object. **Isolate + reconcile** for the former, **sequence** for the latter.

## 9. Shared utility/helper modules
A `utils.ts` that multiple units want to add a function to, or — more dangerous — **modify an existing function in**, where other units depend on its current behavior.

Adding new, unrelated functions: usually fine, low overlap risk, but still same-file — **isolate + reconcile** or just avoid parallel edits by giving one unit ownership of new utility additions. Modifying existing shared behavior that other units rely on: this is a dependency, not just a conflict — the modifying unit must land first, and every consumer depends on it.

## 10. Barrel/index files
An `index.ts` that re-exports everything in a directory, updated every time a new module is added.

Every unit that adds a new module also wants to add one export line to the same barrel file. Classic mechanical-append case — **isolate + reconcile**, or simpler, treat the barrel update as a fast final step done once after the batch rather than by each unit individually.

## 11. Central registration points
A router file where each new endpoint gets registered, a DI container registration list, a `combineReducers`/store setup that lists every slice, a plugin registry.

Structurally identical to barrel files (#10) — same resolution. The distinguishing risk: registration order sometimes matters (e.g. middleware order, route specificity) — if it does, treat this as a **sequence**, not a mechanical merge, so the order is chosen deliberately rather than by merge-conflict-resolution luck.

## 12. Monorepo workspace config
`pnpm-workspace.yaml`, `nx.json`, `turbo.json`, root `tsconfig` project references.

Only touched if a unit adds a *new package*, not for changes within existing packages. **Sequence**: whichever unit adds a new package updates workspace config as part of landing that package, and nothing else should need to touch it.

## 13. Cross-cutting refactors (rename, type change, signature change)
A refactor that touches many call sites of one function/type across the codebase.

This isn't really N independent units — it's one semantic change with many mechanical touch points. If the call sites don't overlap file-wise, splitting by directory/file range can still work as a **parallel batch with no worktree needed** (no two units touch the same file) — but only if the rename/signature is already decided and fixed before the batch starts (i.e. treat "decide the new shape" as its own Phase 1 unit, sequenced ahead of the mechanical replacement batch).

## 14. Same file, non-overlapping regions
Two units need to touch different, well-separated parts of the same file (e.g. two independent route handlers in one large `routes.ts`), where the file itself isn't a shared abstraction, it's just where the code happens to live.

Best resolution is often to **split the file** first (if it's not already too disruptive) so each unit gets a real file boundary — turns a conflict into no-conflict. If splitting isn't reasonable, **isolate + reconcile**, since the regions are genuinely independent and the merge is mechanical.
