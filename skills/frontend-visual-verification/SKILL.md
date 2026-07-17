---
name: frontend-visual-verification
description: Implement and verify visual UI changes in React frontends against a screenshot, mockup, Figma reference, or precise visual brief. Use when a request requires pixel-accurate layout or styling, visual-regression fixes, component polish, responsive alignment, or a claim that a rendered UI matches a reference. Require an inspect-change-render-compare loop; surface unavailable references, routes, states, viewports, or screenshot capability instead of guessing.
---

# Frontend Visual Verification

Make visual correctness observable. Treat a rendered page at a defined route, state, and viewport as the evidence—not a plausible-looking code change.

## Apply verification references

Read these references before verifying the rendered result and treat their checks as acceptance criteria alongside the visual contract:

- [Responsive design](references/responsive.md) — validate large-screen, tablet, and mobile layouts and rule out overlap, clipping, and overflow.
- [User experience](references/user-experience.md) — test the primary flow from its starting point to completion, eliminate unnecessary actions, and make the intended path unambiguous.
- [Accessibility](references/accessibility.md) — verify contrast and screen-reader support.

## Establish the visual contract

1. Read applicable repository instructions and identify the target route, component, styling system, and existing design tokens before editing.
2. Extract or confirm these details from the request and reference:
   - visual reference: screenshot, mockup, Figma frame, or precise written brief;
   - route and UI state, including data, authentication, feature flags, and open/closed interaction states;
   - viewport width and height, plus every required responsive breakpoint; include large-screen, tablet, and mobile viewports unless the request explicitly limits the scope;
   - scope boundaries, behavior that must remain unchanged, and acceptance criteria.
3. Use the existing design system, component primitives, and style conventions unless the request explicitly requires otherwise.
4. Ask for a missing critical detail, or state it as a blocker. Do not invent a reference layout, target viewport, app state, copy, asset, or responsive behavior.
5. If the request asks only for an audit or plan, inspect and report without modifying files.

## Capture evidence before changing code

1. Start the application using its documented project command. Do not guess a command when the repository provides one.
2. Navigate to the agreed route and reproduce the agreed state at the exact target viewport. Wait for fonts, images, loading states, and animations to settle.
3. Capture a baseline screenshot whenever browser or screenshot tooling is available.
4. Compare the baseline with the reference and list only observable discrepancies. Group them by:
   - page and component geometry: position, width, height, grid, alignment, and gaps;
   - typography: family, weight, size, line height, wrapping, and hierarchy;
   - visual styling: colors, borders, radii, shadows, opacity, and assets;
   - behavior visible in the frame: overflow, clipping, empty/loading/error state, and interaction state;
   - responsive differences at each required viewport.
5. Distinguish what is directly visible from what is only inferred. Do not claim pixel measurements that have not been measured or provided.

## Make controlled visual changes

1. Trace each discrepancy to its likely source component and style rule before changing it.
2. Change the smallest coherent set of files necessary to resolve the discrepancy. Preserve data flow, copy, accessibility semantics, and unrelated layout unless the request says to change them.
3. Prefer existing tokens, spacing scales, typography primitives, and component APIs over one-off values, duplicate markup, broad overrides, or `!important`.
4. Make one visual hypothesis per iteration where practical: state the discrepancy, apply its targeted fix, then render again. Group tightly coupled changes only when they must change together to form one layout correction.
5. Keep implementation changes separate from verification. A CSS or JSX edit is not evidence that the rendered result is correct.
6. Stop and surface a conflict when the reference contradicts an explicit project constraint, breaks required behavior, or requires an unavailable asset or state.

## Verify the rendered result

1. Render the same route, state, and viewport used for the baseline after each meaningful iteration.
2. Capture a comparison screenshot at every agreed desktop and responsive viewport, and at each requested interaction state.
3. Check for regressions beyond the reference frame: horizontal scroll, clipped or overlapping content, unexpected wrapping, layout shifts, broken hover/focus states, and invalid responsive behavior.
4. Run the smallest relevant automated checks for the changed code, such as linting, type checking, component tests, or an existing visual-regression test.
5. Exercise the primary user flow and verify its action count and clarity against [the user-experience reference](references/user-experience.md). Verify contrast and screen-reader support against [the accessibility reference](references/accessibility.md).
6. Claim a visual match only when the rendered evidence covers every agreed acceptance criterion. Otherwise, describe the remaining observable differences precisely.
7. If browser or screenshot tooling is unavailable, say that visual verification is unavailable. You may implement a best-effort code change, but label it **unverified** and do not claim visual parity.

## Report concisely

Use this structure unless the user requests another format. Omit empty sections.

```markdown
## Visual contract
- Reference: <source or unavailable>
- Rendered at: <route, state, and viewport(s)>
- Preserved: <behavior and scope boundaries>

## Changes
- <file/component>: <targeted visual correction>

## Verification
- <screenshot or rendered observation at each viewport/state>
- Responsive: <large-screen, tablet, and mobile result; overlap/overflow findings>
- User experience: <tested start-to-finish flow, action count, and ambiguity findings>
- Accessibility: <contrast and screen-reader findings>
- <automated check and result>

## Remaining differences or blockers
- <specific difference, missing input, or reason verification could not occur>
```

Do not use subjective completion language such as “looks good,” “polished,” or “matches” without tying it to the visual contract and rendered evidence.
