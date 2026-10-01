---
description: "Level 1 of Tailwind CSS: what the compiler does, v4 installation, spacing, type, colour, Flexbox, Grid, states, breakpoints, and a landing-page project."
---

# Level 1 · Entry <span class="level-badge">Foundations</span>

Goal: style a complete, responsive page with nothing but utility classes, and be able to
say exactly what CSS each class produces and why one class beats another.

Three ideas carry you through all ten modules:

1. **Tailwind is a compiler, not a stylesheet you download.** It scans your files for
   class names and generates only the CSS those classes need. When a class "doesn't
   work", the question is almost always "did the compiler see it, and what did it
   generate?". Every lesson shows the generated CSS so you build that habit.
2. **Utilities map to CSS you already know.** `p-4` is `padding`, `grid-cols-3` is a grid
   template, `items-center` is `align-items`. Tailwind doesn't replace CSS knowledge; it
   gives it shorter names. Where a lesson depends on a CSS concept, it links to the
   matching page on the [HTML & CSS course](https://sigilipelli.github.io/html-css-mastery-path/).
3. **Variants are prefixes that wrap a rule.** `hover:`, `focus-visible:`, `md:` and
   `group-hover:` all work the same way: they add a pseudo-class, a selector or a media
   query around an ordinary utility.

## Modules

1. [What Tailwind Is (and Isn't)](01-what-tailwind-is.md) — the utility-first idea, what the compiler generates, and the honest trade-offs
2. [Installing Tailwind v4: CLI, Vite & Source Detection](02-installation.md) — the one-line stylesheet, two real setups, and which files get scanned
3. [Utility Basics: Spacing, Sizing & the Scale](03-utilities-spacing-sizing.md) — how names map to values, and the spacing scale built on one variable
4. [Typography Utilities](04-typography.md) — sizes with paired line heights, weight, tracking, wrapping and clamping
5. [Colors, Opacity, Backgrounds & Borders](05-color-backgrounds-borders.md) — the oklch palette, opacity modifiers, gradients, rings and shadows
6. [Layout & Flexbox Utilities](06-flexbox-layout.md) — display, flex items measured, the `min-w-0` fix, positioning and z-index
7. [Grid Utilities](07-grid.md) — `minmax(0, 1fr)`, spans, arbitrary templates and subgrid
8. [Hover, Focus & Other States: group and peer](08-states-variants.md) — state variants, parent and sibling states, and the unnamed-peer leak
9. [Responsive Design with Breakpoint Variants](09-responsive.md) — mobile first, `max-*` ranges, custom breakpoints and the viewport tag
10. [Project — A Landing Page](10-project-landing-page.md) — a full responsive page, checked at three widths

## What you need before starting

- **HTML and CSS basics:** elements, classes, the box model, and what Flexbox and Grid
  are for. You don't need to be an expert, but Tailwind is much easier if you can read
  the CSS it generates. If that's new, start with
  [HTML & CSS Mastery Path](https://sigilipelli.github.io/html-css-mastery-path/).
- **Node.js** (a current LTS release) and npm, to run the compiler. Check with `node -v`.
- An editor. VS Code with the official **Tailwind CSS IntelliSense** extension gives you
  autocompletion and shows the generated CSS on hover (lesson 02).
- A browser with DevTools. You'll use the Elements and Computed panes constantly.

No accounts, cloud services or paid tools are needed anywhere in this course.

## How the examples were checked

Code labelled `css` under "compiled" or "from the compiler" is real output from
**Tailwind CSS 4.3.3** (`@tailwindcss/cli`), trimmed to the relevant rules. Measurements
such as column widths, computed `display` values and file sizes were taken by rendering
the example in Chromium. Where a lesson says "we tested", that's what it means. Small
details (exact byte counts, the order of declarations) can differ between Tailwind
releases; the behaviour taught here is stable across v4.

!!! note "About versions"
    This course teaches Tailwind CSS **v4**, which moved configuration from
    `tailwind.config.js` into CSS (`@import "tailwindcss"` plus `@theme`), changed several
    defaults, and rewrote the engine. Most v3 tutorials you'll find online still use the
    old setup. If you're maintaining a v3 project, Level 3 · 08 covers the migration
    step by step. Run `npm ls tailwindcss` to see which version a project uses.
