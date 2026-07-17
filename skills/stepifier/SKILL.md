---
name: stepifier
description: Transform long, unstructured context into a complete, ordered checklist of small executable steps. Use when an agent needs to extract an actionable plan from lengthy briefs, tickets, specifications, conversations, logs, repository notes, or pasted instructions; identify requirements, dependencies, validation, and unresolved ambiguities without losing constraints or inventing facts.
---

# Stepifier

Convert supplied context into an execution-ready plan that another agent can follow without rereading the source.

## Extract the working model

1. Read every supplied source before drafting the plan. Keep a compact requirements ledger containing the objective, deliverables, scope boundaries, constraints, named files or systems, decisions, dependencies, acceptance criteria, and unresolved items.
2. Apply the current user's explicit instructions first. Within the supplied context, treat a later explicit correction as superseding an earlier conflicting statement; flag conflicts that cannot be resolved this way.
3. Separate facts into three groups:
   - **Confirmed** — stated directly in the source.
   - **Assumptions** — reasonable but unstated details needed to proceed; label these clearly.
   - **Open questions/blockers** — details that materially change the work or cannot be safely inferred.
4. Preserve exact identifiers that affect execution, such as paths, commands, API names, branch names, dates, limits, and acceptance criteria. Do not replace them with vague summaries.

## Build the steps

1. State the desired outcome in one or two sentences, including the completed deliverable and its success condition.
2. Order work by dependency: establish prerequisites, inspect or prepare the current state, make each change, validate the result, then hand off or report it.
3. Break compound work into short, sequential steps. Give each step one primary action that can be completed, inspected, or verified before continuing. Split actions joined by “and” when either action can fail, requires a decision, or produces a separate artifact.
4. Make every step concrete. Name the target and the expected result; include a verification method whenever the source provides one or a safe, standard check is necessary.
5. Preserve conditional paths explicitly: write `If <condition>, <action>; otherwise, <alternative action>.` Do not silently choose a branch when the context does not determine it.
6. Add only operational detail that follows directly from the source or is necessary to make a step executable. Never fabricate filenames, commands, owners, requirements, results, or dates.
7. Keep implementation and validation distinct. Do not treat a change as complete until its stated acceptance criteria have been checked.

## Present the result

Use this format unless the user requests another one. Omit sections with no content.

```markdown
# <Task> — executable plan

**Outcome:** <completed deliverable and success condition>

**Constraints and decisions:**
- <confirmed constraint or decision>

## Steps

1. **<Imperative action>** — <specific target and expected result>.
   - Verify: <test, inspection, or acceptance check>.
2. **<Next imperative action>** — <specific target and expected result>.
   - Depends on: Step <n>.

## Decision points

- If <condition>, <action>; otherwise, <alternative>.

## Open questions / blockers

- <question, missing input, or conflict, plus why it matters>
```

For a small, unambiguous request, return only the outcome and numbered steps. For dense or multi-source context, include constraints, decisions, and questions so the plan remains traceable and safe.

## Check before responding

- Cover every explicit requirement, constraint, and acceptance criterion from the source.
- Place every prerequisite before the step that needs it.
- Keep steps actionable and ordered; remove duplicated prose.
- Mark uncertainty instead of presenting inference as fact.
- Respect the user's requested level of detail and output format.
- Plan only unless the user also asks to execute the steps.
