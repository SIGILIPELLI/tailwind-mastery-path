---
title: "Tailwind CSS Cheat Sheet: Quick Reference, Beginner to Advanced"
description: "A free Tailwind CSS cheat sheet: 20 quick-reference tables from every level of the Tailwind CSS Mastery Path, each linked to its full lesson."
---

# Tailwind CSS Cheat Sheet

Quick-reference tables from **20 lessons** of the free Tailwind CSS Mastery Path, in course order from beginner to advanced. Each section links to the full lesson with explanations and runnable examples.

## Level 1 · Entry

### [03 · Utility Basics: Spacing, Sizing &amp; the Scale](level-1/03-utilities-spacing-sizing.md)

*The naming pattern*

| Prefix | Property | Sides / axes |
|--------|----------|--------------|
| `p` | padding | `pt` `pr` `pb` `pl` (top/right/bottom/left), `px` `py` (inline/block axes), `ps` `pe` (inline start/end) |
| `m` | margin | same pattern: `mt`, `mx`, `my`, `ms`, `me`… |
| `gap` | gap | `gap-x`, `gap-y` |
| `w`, `h` | width, height | `min-w`, `max-w`, `min-h`, `max-h` |
| `size` | width **and** height | — |
| `inset`, `top`, `left`… | position offsets | `inset-x`, `inset-y` |

### [04 · Typography Utilities](level-1/04-typography.md)

*Font size: a scale with built-in line heights*

| Class | Font size | Default line height |
|-------|-----------|---------------------|
| `text-xs` | 0.75rem | 1rem |
| `text-sm` | 0.875rem | 1.25rem |
| `text-base` | 1rem | 1.5rem |
| `text-lg` | 1.125rem | 1.75rem |
| `text-xl` | 1.25rem | 1.75rem |
| `text-2xl` | 1.5rem | 2rem |
| `text-3xl` | 1.875rem | 2.25rem |
| `text-4xl` | 2.25rem | 2.5rem |
| `text-5xl` … `text-9xl` | 3rem … 8rem | 1 |

### [05 · Colors, Opacity, Backgrounds &amp; Borders](level-1/05-color-backgrounds-borders.md)

*The palette*

| Prefix | Property |
|--------|----------|
| `text-` | `color` |
| `bg-` | `background-color` |
| `border-` (also `border-t-`, `border-x-`…) | `border-color` |
| `outline-` | `outline-color` |
| `ring-` | ring colour (a `box-shadow`) |
| `shadow-` | shadow colour |
| `decoration-` | `text-decoration-color` |
| `accent-` | `accent-color` (checkboxes, radios, range) |
| `caret-` | `caret-color` |
| `fill-`, `stroke-` | SVG `fill`, `stroke` |
| `from-`, `via-`, `to-` | gradient stops |

### [06 · Layout &amp; Flexbox Utilities](level-1/06-flexbox-layout.md)

*Flex containers*

| Utility | CSS |
|---------|-----|
| `flex` | `display: flex` |
| `flex-row` / `flex-col` / `flex-row-reverse` | `flex-direction` |
| `flex-wrap` / `flex-nowrap` | `flex-wrap` |
| `justify-start` `center` `end` `between` `around` `evenly` | `justify-content` |
| `items-start` `center` `end` `baseline` `stretch` | `align-items` |
| `content-*` | `align-content` (multi-line) |
| `gap-4`, `gap-x-6`, `gap-y-2` | `gap` |

### [08 · Hover, Focus &amp; Other States: group and peer](level-1/08-states-variants.md)

*The everyday state variants*

| Variant | CSS it adds | Typical use |
|---|---|---|
| `hover:` | `:hover` inside `@media (hover: hover)` | Buttons, links, rows |
| `focus:` | `:focus` | Inputs: style while editing |
| `focus-visible:` | `:focus-visible` | Focus rings on buttons and links (keyboard focus only) |
| `focus-within:` | `:focus-within` | Highlight a wrapper when any field inside has focus |
| `active:` | `:active` | Pressed feedback |
| `disabled:` | `:disabled` | Dimmed, non-interactive controls |
| `first:` / `last:` / `only:` | `:first-child`, `:last-child`, `:only-child` | Removing the border on the last list item |
| `odd:` / `even:` | `:nth-child(odd/even)` | Striped table rows |
| `empty:` | `:empty` | Hide an empty container |
| `required:` / `invalid:` / `user-invalid:` | matching pseudo-classes | Form validation styling |
| `placeholder:` / `file:` / `marker:` / `selection:` | `::placeholder`, `::file-selector-button`, `::marker`, `::selection` | Pseudo-elements |

### [09 · Responsive Design with Breakpoint Variants](level-1/09-responsive.md)

*The default breakpoints*

| Prefix | Theme variable | Value | Pixels at a 16px root font size |
|---|---|---|---|
| `sm:` | `--breakpoint-sm` | `40rem` | 640px |
| `md:` | `--breakpoint-md` | `48rem` | 768px |
| `lg:` | `--breakpoint-lg` | `64rem` | 1024px |
| `xl:` | `--breakpoint-xl` | `80rem` | 1280px |
| `2xl:` | `--breakpoint-2xl` | `96rem` | 1536px |

## Level 2 · Intermediate

### [01 · Design Tokens with @theme](level-2/01-theme-directive.md)

*A theme variable is a CSS variable with a reserved name*

| Namespace | Example variable | Utilities it creates |
|---|---|---|
| `--color-*` | `--color-brand-600` | `bg-brand-600`, `text-`, `border-`, `ring-`, `fill-`, `stroke-`, `shadow-`, `decoration-`, … |
| `--font-*` | `--font-display` | `font-display` |
| `--text-*` | `--text-hero` | `text-hero` (font size) |
| `--font-weight-*` | `--font-weight-heavy` | `font-heavy` |
| `--tracking-*` / `--leading-*` | `--tracking-snug` | `tracking-snug` / `leading-*` |
| `--spacing` / `--spacing-*` | `--spacing-gutter` | `p-gutter`, `m-`, `gap-`, `w-`, `h-`, `inset-`, … |
| `--radius-*` | `--radius-card` | `rounded-card` |
| `--shadow-*` | `--shadow-card` | `shadow-card` |
| `--breakpoint-*` | `--breakpoint-3xl` | the `3xl:` variant |
| `--container-*` | `--container-reading` | `max-w-reading`, `w-reading`, and the `@3xl:`-style container variants |
| `--animate-*` | `--animate-wiggle` | `animate-wiggle` |
| `--ease-*` | `--ease-snappy` | `ease-snappy` |

### [04 · Arbitrary Values, Properties &amp; CSS Variables](level-2/04-arbitrary-values.md)

*When to use which*

| Situation | Use |
|---|---|
| Value used once, and specific to this element | Arbitrary value `w-[37rem]` |
| Value repeated in several places | A theme token (`@theme`) |
| Value changes at runtime | A CSS variable + `(--var)` class |
| CSS property with no utility | Arbitrary property `[mask-type:…]` |
| Styling children you don't control | Arbitrary variant `[&_p]:…` or `*:` |
| A long or repeated arbitrary class | A custom utility (lesson 05) |

### [06 · Advanced Variants: aria, data, has, not &amp; Custom Variants](level-2/06-advanced-variants.md)

*`nth-*` for positional styling*

| Class | Selector |
|---|---|
| `nth-3:font-bold` | `:nth-child(3)` |
| `nth-[3n+1]:clear-left` | `:nth-child(3n+1)` |
| `nth-last-2:mb-0` | `:nth-last-child(2)` |

### [07 · Container Queries in Tailwind](level-2/07-container-queries.md)

*The two halves: a container and a query*

| Variant | Size | | Variant | Size |
|---|---|---|---|---|
| `@3xs:` | 16rem | | `@lg:` | 32rem |
| `@2xs:` | 18rem | | `@xl:` | 36rem |
| `@xs:` | 20rem | | `@2xl:` | 42rem |
| `@sm:` | 24rem | | `@3xl:` | 48rem |
| `@md:` | 28rem | | … up to `@7xl:` | 80rem |

### [09 · Transitions, Transforms &amp; Animation](level-2/09-transitions-animation.md)

*Transitions*

| Utility | `transition-property` |
|---|---|
| `transition` | a sensible default list: colours, `opacity`, `box-shadow`, `transform`, `translate`, `scale`, `rotate`, filters, `display`, … |
| `transition-colors` | colour properties only (`color`, `background-color`, `border-color`, `fill`, …) |
| `transition-opacity` | `opacity` |
| `transition-transform` | `transform, translate, scale, rotate` |
| `transition-shadow` | `box-shadow` |
| `transition-all` | `all` |
| `transition-[height]` | whatever you put in brackets |

## Level 3 · Advanced

### [02 · Cascade Layers, Preflight &amp; Specificity](level-3/02-layers-preflight.md)

*The four layers*

| Layer | Contains | Beaten by |
|---|---|---|
| `theme` | `:root` variables from `@theme` | everything |
| `base` | Preflight, your `@layer base` rules, base styles from plugins | components, utilities |
| `components` | your `@layer components` rules | utilities |
| `utilities` | every generated utility, `@utility` definitions | nothing in a layer |
| *(no layer)* | any CSS not inside a layer | **beats all layers** |

### [04 · Accessibility with Tailwind](level-3/04-accessibility.md)

*Colour contrast: measure, don't guess*

| Text colour | Ratio on white | Normal text (4.5) | Large text (3) |
|---|---|---|---|
| `gray-400` | 2.60 | fail | fail |
| `gray-500` | 4.84 | pass | pass |
| `gray-600` | 7.56 | pass | pass |
| `slate-500` | 4.76 | pass | pass |
| `sky-500` | 2.71 | fail | fail |
| `sky-600` | 4.02 | **fail** | pass |
| `sky-700` | 5.86 | pass | pass |
| `red-500` | 3.81 | fail | pass |
| `red-600` | 4.77 | pass | pass |
| `emerald-600` | 3.65 | fail | pass |
| `emerald-700` | 5.36 | pass | pass |
| `amber-500` | 2.13 | fail | fail |
| `amber-600` | 3.20 | fail | pass |

### [08 · Migrating from v3 to v4](level-3/08-migrating-v3-v4.md)

*What it converted*

| v3 | v4 | Why |
|---|---|---|
| `shadow-sm` / `shadow` | `shadow-xs` / `shadow-sm` | the scale shifted down one step to make room |
| `rounded-sm` / `rounded` | `rounded-xs` / `rounded-sm` | same shift |
| `blur-sm` / `blur` | `blur-xs` / `blur-sm` | same shift |
| `ring` (3px) | `ring-3` | bare `ring` is now 1px |
| `outline-none` | `outline-hidden` | `outline-none` now really sets `outline-style: none` |
| `!rounded-full` | `rounded-full!` | important modifier moved to the end |
| `bg-[--accent]` | `bg-(--accent)` | new variable shorthand |
| `flex-grow-0` / `flex-shrink` | `grow-0` / `shrink` | deprecated aliases removed |
| `decoration-slice`, `overflow-ellipsis` | `box-decoration-slice`, `text-ellipsis` | deprecated aliases removed |

### [09 · Production Builds &amp; Performance](level-3/09-production-performance.md)

*Real sizes*

| Page | Minified | gzip | Brotli |
|---|---|---|---|
| Level 1 landing page | 15,211 B | 3,764 B | 3,213 B |
| Level 2 dashboard | 18,933 B | 4,563 B | 3,977 B |

## Level 4 · Master

### [03 · RTL, Logical Utilities &amp; Internationalization](level-4/03-rtl-logical.md)

*Logical utilities*

| Physical | Logical | CSS property (compiled) |
|---|---|---|
| `ml-4` / `mr-4` | `ms-4` / `me-4` | `margin-inline-start` / `margin-inline-end` |
| `pl-2` / `pr-2` | `ps-2` / `pe-2` | `padding-inline-start` / `-end` |
| `left-0` / `right-0` | `start-0` / `end-0` | `inset-inline-start` / `-end` |
| `border-l-4` | `border-s-4` | `border-inline-start-width` (+ style) |
| `rounded-l-lg` | `rounded-s-lg` | `border-start-start-radius` + `border-end-start-radius` |
| `text-left` | `text-start` | `text-align: start` |
| `float-left` | `float-start` | `float: inline-start` |
| `scroll-ml-4` | `scroll-ms-4` | `scroll-margin-inline-start` |

### [04 · Print, Contrast, Motion &amp; Other Media Variants](level-4/04-media-variants.md)

*The variants, tested*

| Variant | Media query |
|---|---|
| `print:` | `@media print` |
| `contrast-more:` / `contrast-less:` | `@media (prefers-contrast: more / less)` |
| `forced-colors:` | `@media (forced-colors: active)` |
| `motion-safe:` / `motion-reduce:` | `@media (prefers-reduced-motion: no-preference / reduce)` |
| `pointer-fine:` / `pointer-coarse:` / `pointer-none:` | `@media (pointer: …)` |
| `any-pointer-coarse:` and friends | `@media (any-pointer: …)` |
| `portrait:` / `landscape:` | `@media (orientation: …)` |
| `noscript:` | `@media (scripting: none)` |
| `inverted-colors:` | `@media (inverted-colors: inverted)` |

### [07 · Tailwind with Server-Rendered Templates &amp; Static Sites](level-4/07-server-templates.md)

*Reusing markup: partials, macros and components*

| Stack | Reuse mechanism |
|---|---|
| Jinja / Django | macros (`{% macro %}`), `{% include %}` with `with` context, Django inclusion tags |
| Rails | partials (`render "shared/badge", status:`), ViewComponent |
| Laravel | Blade components (`<x-badge :status="$status" />`) |
| Go | `{{ template "badge" . }}` blocks |
| Static generators | includes / shortcodes (Hugo partials, Eleventy shortcodes) |

### [08 · Debugging Tailwind](level-4/08-debugging.md)

*A catalogue of real bugs from this course*

| Symptom | Question | Cause | Fix | Lesson |
|---|---|---|---|---|
| Classes built as `` `bg-${color}-500` `` missing | 1 | scanner sees fragments | full class names or a variable | L3·01 |
| `border-{{ accent }}-500` missing, though in rendered HTML | 1 | template interpolation | mapping in code | L4·07 |
| `bg-[--brand]` does nothing | 3 | v3 syntax compiles to `background-color: --brand` | `bg-(--brand)` | L2·04 |
| `text-(--size)` changes colour, not size | 1 | no type hint | `text-(length:--size)` | L2·04 |
| Error message shown for a *valid* field | 2 | unnamed `peer` matches any earlier invalid sibling | `peer/name` | L1·08 |
| `class="note p-2"` ignores `p-2` | 2 | unlayered CSS beats layers | `@layer components` | L2·05 |
| Third-party widget ignores utilities | 2 | its CSS is unlayered | `@import … layer(components)` | L3·02 |
| `hidden` element won't show with `flex` | 2 | Preflight's important `[hidden]` rule | remove the attribute | L3·02 |
| Brand override in a section doesn't apply | 3 | theme token resolved on `:root` | `@theme inline` | L4·02 |
| `scale-*` doesn't animate | 2 | own CSS transitions `transform`, v4 uses `scale` | `transition-transform` | L2·09 |
| `@md:` styles never apply | 2 | `@container` on the same element | container on a wrapper | L2·07 |
| `@container` element has zero width | 2 | size containment on a shrink-to-fit box | give it a width | L2·07 |
| No focus ring in high-contrast mode | 2 | rings are box-shadows, removed in forced colours | `outline-hidden` | L3·04 |
| `tailwind-merge` drops a colour | – | unknown custom token guessed as colour | `extendTailwindMerge` | L3·06 |
| Audit reports 1:1 contrast after theme switch | – | measured mid-transition | wait, or disable transitions while switching | L3·10 |
| Opacity lost after v3→v4 upgrade | 1 | `bg-opacity-*` removed, not migrated | `bg-x/90` | L3·08 |

### [09 · Testing Tailwind UIs](level-4/09-testing.md)

*Three layers of tests*

| Layer | Checks | Speed | Catches |
|---|---|---|---|
| Unit (Level 3 · 10) | the class strings recipes return | milliseconds | broken variants, merge conflicts |
| Rendered assertions | computed styles in a real browser | ~0.1–0.5 s each | contrast, focus, overflow, media variants |
| Visual regression | screenshots compared with a baseline | ~1 s each | anything visible, including things you didn't think to assert |

---

Go deeper with the full [Tailwind CSS Mastery Path](index.md) curriculum or test yourself with the [interview questions](interview-questions.md).
