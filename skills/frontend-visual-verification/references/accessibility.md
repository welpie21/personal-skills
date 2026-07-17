# Accessibility

Verify accessibility in the rendered state, including interaction states that affect visibility or announcements.

## Contrast

- Check text, icons, controls, borders, and focus indicators against their actual foreground and background colors in default, hover, focus, disabled, error, and selected states.
- Ensure content remains understandable without color alone; pair color cues with text, icons, patterns, or other discernible indicators.
- Preserve clear visual hierarchy without using low-contrast text or controls.

## Screen-reader support

- Use semantic HTML and landmarks so screen-reader users can identify page regions, headings, lists, buttons, links, forms, and tables correctly.
- Give every interactive control an accessible name. Associate form inputs with visible labels, instructions, validation errors, and required status.
- Preserve a logical reading and focus order that matches the rendered interface. Keep focus visible and reachable with a keyboard.
- Announce meaningful dynamic updates, such as validation results, loading completion, errors, and success feedback, without causing unnecessary interruption.
- Provide useful text alternatives for informative images and hide decorative images from assistive technology.

Report the inspected states, the evidence used, and any limitation in screen-reader or contrast verification. Do not claim accessibility conformance without the evidence required by the project's standard.
