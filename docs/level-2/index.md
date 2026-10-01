---
description: "Level 2 of Tailwind CSS: @theme tokens, dark mode, arbitrary values, @apply and @utility, variants, container queries, forms and motion."
---

# Level 2 · Intermediate <span class="level-badge">Building Real UIs</span>

Goal: stop using Tailwind's defaults as-is and make it fit a real product: your colours
and fonts, a dark mode, components that adapt to where they're placed, forms that are
accessible, and motion that respects user settings. By the end you'll build an
application dashboard that uses nearly every tool in this level.

The ideas that carry this level:

1. **Your design system lives in CSS.** In v4, `@theme` turns CSS variables into design
   tokens *and* utilities. Utilities reference the variables, so changing a variable at
   runtime (for dark mode, or a second brand) restyles everything that uses it.
2. **Style from state that already exists.** ARIA attributes, data attributes, `:has()`
   and `:checked` already describe your UI's state. Variants like `aria-[current=page]:`,
   `data-[state=open]:` and `has-checked:` let you style from them instead of mirroring
   state into extra classes with JavaScript.
3. **Escape hatches are for exceptions.** Arbitrary values, `@apply` and custom variants
   are useful, and each has a clear "right time". A value used once is fine inline. A
   value used in ten places belongs in the theme.

## Modules

1. [Design Tokens with @theme](01-theme-directive.md) — namespaces, what gets emitted, `inline`, `static`, and resetting defaults
2. [Customizing Colors, Fonts, Spacing & Breakpoints](02-customizing-theme.md) — a brand scale, web fonts, type tokens with line heights, named spacing
3. [Dark Mode](03-dark-mode.md) — system preference, a class toggle with `@custom-variant`, no flash, and token-based themes
4. [Arbitrary Values, Properties & CSS Variables](04-arbitrary-values.md) — brackets, `(--var)`, type hints, arbitrary properties and variants, `!`
5. [Reusing Styles: Components, @apply & @utility](05-reusing-styles.md) — when to extract, cascade layers, and `@reference`
6. [Advanced Variants: aria, data, has, not & Custom Variants](06-advanced-variants.md) — styling from existing state
7. [Container Queries in Tailwind](07-container-queries.md) — components that respond to their own width, and the containment pitfalls
8. [Styling Forms](08-forms.md) — after Preflight, validation, `accent-*`, and the forms plugin
9. [Transitions, Transforms & Animation](09-transitions-animation.md) — v4's separate transform properties, `starting:`, keyframes, reduced motion
10. [Project — A Dashboard Layout](10-project-dashboard.md) — tokens, dark mode, an accessible drawer, container queries and a data table

## What you need before starting

- **Level 1**, or equivalent experience: you can build a responsive page with utilities
  and read the CSS Tailwind generates.
- **Basic JavaScript** for lessons 03, 06 and the project: selecting elements, adding
  event listeners, setting attributes. If that's new, see
  [JavaScript Mastery Path](https://sigilipelli.github.io/javascript-mastery-path/).
- **CSS custom properties.** Lessons 01–03 lean on how `var()` and inheritance work. The
  [Custom Properties](https://sigilipelli.github.io/html-css-mastery-path/level-2/09-custom-properties/)
  lesson on the HTML & CSS course covers what you need.

All examples were compiled with Tailwind CSS 4.3.3, and behaviour claims (what a variant
matches, whether a transition runs, what a layout measures) were checked by rendering
them in Chromium. Where browser support for a feature is still incomplete
(`field-sizing`, `@starting-style`), the lesson says so and shows a fallback.
