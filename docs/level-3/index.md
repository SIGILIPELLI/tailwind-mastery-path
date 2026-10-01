---
description: "Level 3 of Tailwind CSS: the compiler, layers and Preflight, plugins, accessibility, typography, class merging, design systems, migration."
---

# Level 3 · Advanced <span class="level-badge">How It Really Works</span>

Goal: understand Tailwind well enough to predict what it will do, fix it when it doesn't,
and build things other developers depend on — a design system, a shared component
library, a migration plan.

Level 3 is where "it works on my page" turns into "I know why it works". Three themes run
through it:

1. **The engine is predictable.** The compiler scans text for class-shaped strings,
   generates CSS for the ones it recognises, and sorts them into cascade layers. Once you
   know that, missing classes (lesson 01) and classes that don't win (lesson 02) stop
   being mysteries.
2. **Measure, don't assume.** Several lessons test claims in a real browser: contrast
   ratios of palette colours, focus indicators in forced-colours mode, what the v3→v4
   upgrade tool actually changes, how CSS size scales. Some of these checks found real
   problems in this course's own earlier examples, and the lessons say so.
3. **Build for other people.** Custom utilities, plugins, class-merging helpers, token
   packages and component recipes are all about making the right thing easy for the
   developers who use your work.

## Modules

1. [How the Engine Works: Scanning & Generation](01-how-the-engine-works.md) — candidates, why dynamic class names fail, `@source`, safelisting
2. [Cascade Layers, Preflight & Specificity](02-layers-preflight.md) — the four layers, unlayered CSS, what Preflight removes
3. [Custom Utilities, Functional Utilities & Plugins](03-custom-utilities-plugins.md) — `--value()`, `--modifier()`, JavaScript plugins
4. [Accessibility with Tailwind](04-accessibility.md) — measured contrast, focus in forced-colours mode, hiding, targets
5. [Long-Form Content & the Typography Plugin](05-typography-plugin.md) — `prose`, modifiers, `not-prose`, custom prose themes
6. [Tailwind in Component Frameworks: Class Merging & Variant APIs](06-frameworks-class-merging.md) — clsx, tailwind-merge, cva, and custom-token gotchas
7. [Building a Design System on Tailwind](07-design-systems.md) — token tiers, a restricted palette, a shared token package
8. [Migrating from v3 to v4](08-migrating-v3-v4.md) — the upgrade tool on a real project, and what it missed
9. [Production Builds & Performance](09-production-performance.md) — real sizes, what makes CSS grow, caching, rendering costs
10. [Project — A Component Library](10-project-component-library.md) — recipes, tests, a generated gallery, and an accessibility audit

## What you need before starting

- **Levels 1 and 2**, or equivalent experience: theme tokens, dark mode, variants,
  container queries and `@apply`/`@utility`.
- **JavaScript modules and npm** for lessons 03, 06, 07 and the project. The examples
  run on Node.js with ES modules (`import`/`export`).
- **Comfort with DevTools**: the Elements, Computed and Rendering panes. Lesson 04 uses the
  Rendering pane's emulation of `forced-colors` and `prefers-color-scheme`.
- For lesson 06, basic familiarity with at least one component framework (React, Vue or
  Svelte) helps, though the libraries themselves are framework-agnostic.

The tools used and their versions are named in each lesson: Tailwind 4.3.3, the
Typography plugin 0.5.20, the forms plugin 0.5.11, tailwind-merge 3.7.0, cva 0.7.1 and
clsx 2.1.1. Lesson 08 also uses Tailwind 3.4.19 and the official upgrade tool.
