---
title: "Tailwind CSS Interview Questions and Answers (180 Questions)"
description: "180 Tailwind CSS interview questions with answers and explanations, from beginner to advanced, organised by level. Free, from the Tailwind CSS Mastery Path."
---

# Tailwind CSS Interview Questions and Answers

**180 questions** with the correct answer and a short explanation, organised from beginner (Level 1) to advanced (Level 4). Every question comes from a lesson in the free Tailwind CSS Mastery Path, so you can open the lesson to go deeper.

!!! tip "Practise, don't just read"
    Each lesson also ends with an interactive quiz, and the [final quiz](quiz.md) tests all four levels. Try to answer each question below before reading the answer.

## Level 1 · Entry

### [01 · What Tailwind Is (and Isn't)](level-1/01-what-tailwind-is.md)

**Q1.** What is Tailwind CSS?

**Answer:** A utility-first CSS framework: you style elements by composing small single-purpose classes in your markup

Utilities such as p-4 and flex replace hand-written class rules.

**Q2.** What does Tailwind v4 generate in your final CSS?

**Answer:** Only the utilities your source files actually use, found by scanning your templates

Unused classes are not emitted.

**Q3.** What is the main trade-off of utility-first CSS?

**Answer:** Longer class lists in markup in exchange for no naming, no dead CSS and styles co-located with structure

Components and partials keep repetition manageable.

**Q4.** Why isn't Tailwind the same as inline styles?

**Answer:** Utilities support states, breakpoints, themes and design tokens, which inline styles cannot

hover:, md: and dark: variants are the key difference.

### [02 · Installing Tailwind v4: CLI, Vite &amp; Source Detection](level-1/02-installation.md)

**Q5.** What is the entire CSS entry file for Tailwind v4 in the simplest case?

**Answer:** @import "tailwindcss";

v4 replaced the three @tailwind directives with a single import.

**Q6.** Which package provides the Vite plugin in v4?

**Answer:** @tailwindcss/vite

Add it to vite.config and import tailwindcss in your CSS.

**Q7.** How does v4 decide which files to scan for class names?

**Answer:** Automatic source detection that respects .gitignore, plus @source to add or exclude paths

No tailwind.config.js is required.

**Q8.** Which command builds CSS with the standalone CLI package?

**Answer:** npx @tailwindcss/cli -i input.css -o output.css

Add --watch during development.

### [03 · Utility Basics: Spacing, Sizing &amp; the Scale](level-1/03-utilities-spacing-sizing.md)

**Q9.** What does p-4 compile to in v4?

```html
<div class="p-4">
```

**Answer:** padding: calc(var(--spacing) * 4)

Spacing utilities multiply the --spacing theme variable (0.25rem by default), so p-4 is 1rem. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q10.** What does mx-auto compile to?

```html
<div class="mx-auto">
```

**Answer:** margin-inline: auto

v4 emits logical margin-inline, centring block elements with a width. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q11.** What do w-1/2 and h-screen compile to?

```html
<div class="w-1/2 h-screen">
```

**Answer:** width: calc(1 / 2 * 100%); height: 100vh

Fractions become percentages; screen units use the viewport. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q12.** What does size-8 compile to?

```html
<div class="size-8">
```

**Answer:** width: calc(var(--spacing) * 8); height: calc(var(--spacing) * 8)

size-* sets width and height together. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

### [04 · Typography Utilities](level-1/04-typography.md)

**Q13.** What does text-xl font-bold compile to (v4 default theme)?

```html
<div class="text-xl font-bold">
```

**Answer:** font-size: var(--text-xl); font-weight: var(--font-weight-bold)

Utilities reference theme variables, which carry the real values. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q14.** What does leading-tight do?

```html
<div class="leading-tight">
```

**Answer:** line-height: var(--leading-tight)

Line height is separate from font size (unless using text-xl/6 shorthand). Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q15.** What does truncate do?

```html
<div class="truncate">
```

**Answer:** text-overflow: ellipsis; white-space: nowrap; overflow: hidden

A common one-class single-line ellipsis. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q16.** How do you set font size and line height together?

**Answer:** With a slash modifier such as text-lg/7

The modifier sets line-height.

### [05 · Colors, Opacity, Backgrounds &amp; Borders](level-1/05-color-backgrounds-borders.md)

**Q17.** What does bg-blue-500 compile to?

```html
<div class="bg-blue-500">
```

**Answer:** background-color: var(--color-blue-500)

Colours are theme variables (OKLCH in v4). Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q18.** What does bg-black/50 produce in v4?

```html
<div class="bg-black/50">
```

**Answer:** background-color: color-mix(in oklab, var(--color-black) 50%, transparent)

The slash sets colour alpha without a separate opacity utility. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q19.** What does rounded-lg border border-gray-300 compile to?

```html
<div class="rounded-lg border border-gray-300">
```

**Answer:** border-radius: var(--radius-lg); border-style: var(--tw-border-style); border-width: 1px; border-color: var(--color-gray-300)

border sets a 1px width; colour is separate. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q20.** What does shadow-md use in v4?

```html
<div class="shadow-md">
```

**Answer:** box-shadow: var(--tw-inset-shadow), var(--tw-inset-ring-shadow), var(--tw-ring-offset-shadow), var(--tw-ring-shadow), var(--tw-shadow)

Shadows compose through variables so rings and shadows can stack. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

### [06 · Layout &amp; Flexbox Utilities](level-1/06-flexbox-layout.md)

**Q21.** What does flex items-center justify-between compile to?

```html
<div class="flex items-center justify-between">
```

**Answer:** display: flex; align-items: center; justify-content: space-between

Main-axis alignment is justify-*; cross-axis is items-*. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q22.** What does flex-1 compile to?

```html
<div class="flex-1">
```

**Answer:** flex: 1

Grow and shrink to fill available space. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q23.** What does gap-4 compile to?

```html
<div class="gap-4">
```

**Answer:** gap: calc(var(--spacing) * 4)

gap works in flex and grid containers. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q24.** Which utility changes the main axis to vertical?

**Answer:** flex-col

flex-row is the default.

### [07 · Grid Utilities](level-1/07-grid.md)

**Q25.** What does grid grid-cols-3 gap-4 compile to?

```html
<div class="grid grid-cols-3 gap-4">
```

**Answer:** display: grid; grid-template-columns: repeat(3, minmax(0, 1fr)); gap: calc(var(--spacing) * 4)

Equal-width columns via minmax(0, 1fr). Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q26.** What does col-span-2 do?

```html
<div class="col-span-2">
```

**Answer:** grid-column: span 2 / span 2

The item spans two tracks. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q27.** How do you build a responsive auto-fit grid with an arbitrary minimum width?

**Answer:** grid-cols-[repeat(auto-fit,minmax(16rem,1fr))]

Arbitrary values accept any CSS value.

**Q28.** When is grid better than flex?

**Answer:** When layout is two-dimensional (rows and columns) rather than a single line of items

Choose per layout.

### [08 · Hover, Focus &amp; Other States: group and peer](level-1/08-states-variants.md)

**Q29.** What does hover:bg-red-500 compile to?

```html
<div class="hover:bg-red-500">
```

**Answer:** @media (hover: hover) { &amp;:hover { background-color: var(--color-red-500)

v4 wraps hover styles in @media (hover: hover) so touch devices don't get sticky hover. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q30.** What does focus-visible:ring-2 do?

```html
<div class="focus-visible:ring-2">
```

**Answer:** &amp;:focus-visible

Only shows for keyboard-style focus. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q31.** What does the group / group-hover: pair do?

**Answer:** Styles a child when a marked ancestor (class group) is hovered

peer and peer-checked: do the same for siblings.

**Q32.** What does disabled:opacity-50 target?

```html
<div class="disabled:opacity-50">
```

**Answer:** &amp;:disabled

State variants map to pseudo-classes. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

### [09 · Responsive Design with Breakpoint Variants](level-1/09-responsive.md)

**Q33.** What does md:flex compile to (v4 default)?

```html
<div class="md:flex">
```

**Answer:** @media (width &gt;= 48rem) { .md\:flex { display: flex

Breakpoints are rem-based and mobile-first: md is 48rem. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q34.** What does mobile-first mean in Tailwind?

**Answer:** Unprefixed utilities apply to all sizes and prefixed ones (sm:, md:) apply from that breakpoint up

Start with the small layout.

**Q35.** What does max-md:hidden do?

```html
<div class="max-md:hidden">
```

**Answer:** @media (width &lt; 48rem)

max-* variants apply below the breakpoint. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q36.** Which tag is needed for responsive layouts to work on phones?

**Answer:** &lt;meta name="viewport" content="width=device-width, initial-scale=1"&gt;

Without it phones render a desktop-width page.

### [10 · Project — A Landing Page](level-1/10-project-landing-page.md)

**Q37.** What is a good order for building a landing page?

**Answer:** Structure and content first, then spacing and typography, then colour and polish, mobile first

Iterate.

**Q38.** How do you keep repeated markup like cards manageable?

**Answer:** Extract a component (partial, template or framework component) rather than @apply-ing everything

Utility repetition lives in components.

**Q39.** Why use semantic HTML with Tailwind?

**Answer:** Utilities style appearance only; elements and ARIA provide meaning and accessibility

Use header, main, nav, button.

**Q40.** How do you check the page is responsive?

**Answer:** Resize or use device emulation at each breakpoint and test real content

Content breaks layouts, not widths alone.

## Level 2 · Intermediate

### [01 · Design Tokens with @theme](level-2/01-theme-directive.md)

**Q41.** What does the @theme directive let you do in v4?

```html
@theme {
  --color-brand: oklch(0.6 0.2 250);
}

<div class="bg-brand">
```

**Answer:** --color-brand: oklch(0.6 0.2 250)

Variables defined in @theme generate matching utilities such as bg-brand. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q42.** What does defining --color-brand in @theme create?

**Answer:** Utilities like bg-brand, text-brand and border-brand, plus a CSS variable usable anywhere

Theme variables are the design tokens.

**Q43.** How do you add a custom breakpoint in v4?

**Answer:** Define --breakpoint-3xl: 120rem in @theme, generating the 3xl: variant

Variants follow variable names.

**Q44.** What is the effect of --color-*: initial in @theme?

**Answer:** Resets the default colour palette so you can define your own

Use to restrict a design system.

### [02 · Customizing Colors, Fonts, Spacing &amp; Breakpoints](level-2/02-customizing-theme.md)

**Q45.** A theme sets --spacing: 0.5rem. What does p-4 become?

```html
@theme {
  --spacing: 0.5rem;
}

<div class="p-4">
```

**Answer:** padding: calc(var(--spacing) * 4)

The calc multiplies the variable, so the value is 2rem with that theme. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q46.** How do you extend rather than replace the default theme?

**Answer:** Add new variables in @theme; only the namespaces you reset (like --color-*: initial) are replaced

v4 merges by default.

**Q47.** Where can you reference a theme token outside utilities?

**Answer:** As a CSS variable such as var(--color-brand) in custom CSS or arbitrary values

Tokens are real custom properties.

**Q48.** How do you organise a large theme?

**Answer:** Keep tokens in a dedicated CSS file imported into your entry, grouped by namespace

Version tokens like code.

### [03 · Dark Mode](level-2/03-dark-mode.md)

**Q49.** What does dark:bg-gray-900 compile to by default?

```html
<div class="dark:bg-gray-900">
```

**Answer:** @media (prefers-color-scheme: dark)

The default dark variant follows the OS setting. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q50.** How do you make dark mode class-based in v4?

```html
@custom-variant dark (&:where(.dark, .dark *));

<div class="dark:bg-black">
```

**Answer:** &amp;:where(.dark, .dark *)

Override with @custom-variant dark (&amp;:where(.dark, .dark *)). Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q51.** Why avoid a flash of the wrong theme on load?

**Answer:** Apply the stored theme class before first paint with a tiny inline script in the head

Respect system preference as the default.

**Q52.** How can semantic tokens simplify dark mode?

**Answer:** Define --color-surface and change its value per theme so components need fewer dark: variants

Theme variables switch under .dark.

### [04 · Arbitrary Values, Properties &amp; CSS Variables](level-2/04-arbitrary-values.md)

**Q53.** What does w-[13px] compile to?

```html
<div class="w-[13px]">
```

**Answer:** width: 13px

Square brackets accept arbitrary values. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q54.** What does bg-(--brand) do in v4?

```html
<div class="bg-(--brand)">
```

**Answer:** background-color: var(--brand)

Parentheses are shorthand for a CSS variable value. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q55.** What does [mask-type:luminance] do?

```html
<div class="[mask-type:luminance]">
```

**Answer:** mask-type: luminance

Arbitrary properties emit any declaration. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q56.** What is a good reason to avoid arbitrary values everywhere?

**Answer:** They bypass the design system, so prefer theme tokens for anything reused

Use for one-offs.

### [05 · Reusing Styles: Components, @apply &amp; @utility](level-2/05-reusing-styles.md)

**Q57.** What is the recommended way to reuse a button style?

**Answer:** A component in your framework or template engine

Keep utilities in one place.

**Q58.** When is @apply reasonable?

**Answer:** For styling third-party markup you cannot add classes to, or small base rules

It loses the utility-first benefits.

**Q59.** What does @utility let you define in v4?

```html
@utility card-contain {
  contain: layout paint;
}

<div class="card-contain">
```

**Answer:** contain: layout paint

Custom utilities work with variants automatically. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q60.** What is the cost of heavy @apply use?

**Answer:** Larger CSS and hidden coupling, undermining the point of utilities

Prefer components.

### [06 · Advanced Variants: aria, data, has, not &amp; Custom Variants](level-2/06-advanced-variants.md)

**Q61.** What does group-hover:underline compile to?

```html
<div class="group-hover:underline">
```

**Answer:** &amp;:is(:where(.group):hover *)

The variant matches descendants of a hovered .group. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q62.** What does has-[:checked]:bg-blue-50 target?

```html
<div class="has-[:checked]:bg-blue-50">
```

**Answer:** &amp;:has(:checked)

Style a parent that contains a checked input. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q63.** What does data-[state=open]:rotate-180 target?

```html
<div class="data-[state=open]:rotate-180">
```

**Answer:** &amp;[data-state="open"]

Style based on data attributes set by headless UI libraries. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q64.** What does not-hover:opacity-75 do?

```html
<div class="not-hover:opacity-75">
```

**Answer:** &amp;:not(*:hover)

not-* negates another variant. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

### [07 · Container Queries in Tailwind](level-2/07-container-queries.md)

**Q65.** What does @md:flex do with a @container parent?

```html
<div class="@md:flex">
```

**Answer:** @container (width &gt;= 28rem)

Container query variants respond to the nearest @container ancestor's width. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q66.** How do you mark an element as a query container?

**Answer:** Add the @container class

The class sets container-type: inline-size.

**Q67.** Why use container queries for components?

**Answer:** The component adapts to the space it is given, not the viewport

Great for reusable cards.

**Q68.** What does @max-md: do?

**Answer:** Applies below the container breakpoint

Range variants exist too.

### [08 · Styling Forms](level-2/08-forms.md)

**Q69.** What does the @tailwindcss/forms plugin do?

**Answer:** Resets form controls to a consistent, easily styled baseline

Load with @plugin.

**Q70.** How do you style invalid inputs?

**Answer:** Use invalid: or aria-invalid: variants

Pair with visible error text.

**Q71.** What does focus:outline-2 focus:outline-blue-500 add?

```html
<div class="focus:outline-2 focus:outline-blue-500">
```

**Answer:** &amp;:focus

Keep a visible focus indicator for accessibility. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q72.** Why associate labels with inputs?

**Answer:** So screen readers and clicks work: use for/id or wrap the input in the label

Placeholders are not labels.

### [09 · Transitions, Transforms &amp; Animation](level-2/09-transitions-animation.md)

**Q73.** What does transition duration-200 ease-out compile to?

```html
<div class="transition duration-200 ease-out">
```

**Answer:** transition-duration: var(--tw-duration, var(--default-transition-duration))

The duration utility sets a variable and a transition property group. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q74.** Which animation utility spins an element?

**Answer:** animate-spin

Use motion-reduce: to respect user preferences.

**Q75.** Why add motion-reduce: variants?

**Answer:** Users who prefer reduced motion should not get large animations

Accessibility.

**Q76.** What does starting:opacity-0 enable?

```html
<div class="starting:opacity-0">
```

**Answer:** @starting-style

Animate elements as they enter the DOM without JS. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

### [10 · Project — A Dashboard Layout](level-2/10-project-dashboard.md)

**Q77.** What does the dashboard project combine?

**Answer:** Layout (grid and flex), components, dark mode, responsive design and a custom theme

Level 2 capstone.

**Q78.** How do you make a data table responsive?

**Answer:** Wrap it in an overflow-x-auto container or reflow into cards on small screens

Keep headers readable.

**Q79.** How do you keep contrast accessible?

**Answer:** Check text and background colours against WCAG ratios, including in dark mode

Test with tooling.

**Q80.** How should you organise components?

**Answer:** Small, composable and styled with theme tokens

Consistency scales.

## Level 3 · Advanced

### [01 · How the Engine Works: Scanning &amp; Generation](level-3/01-how-the-engine-works.md)

**Q81.** How does the v4 engine produce CSS?

**Answer:** It scans sources for candidate class names and compiles only the matching utilities (plus theme variables used)

Written for speed in Rust-based tooling.

**Q82.** Why must class names appear as complete strings in source?

**Answer:** The scanner matches whole tokens, so dynamically built names like `bg-${color}-500` are not found

Map to full class names.

**Q83.** What does @source do?

**Answer:** Adds extra paths or inline safelists for the scanner (and can exclude paths)

Use for classes outside detected files.

**Q84.** What does the compiled output contain besides utilities?

**Answer:** Theme variables, preflight base styles and cascade layers

Layers: theme, base, components, utilities.

### [02 · Cascade Layers, Preflight &amp; Specificity](level-3/02-layers-preflight.md)

**Q85.** What are CSS cascade layers used for in Tailwind v4?

**Answer:** Predictable ordering: theme, base, components, utilities so utilities win over base and components

Real @layer rules.

**Q86.** What does Preflight do?

**Answer:** Applies a modern CSS reset (box-sizing, margins, headings) as base styles

Customise in @layer base.

**Q87.** Where should you put custom base styles?

**Answer:** In @layer base so utilities can still override them

Order matters.

**Q88.** What is the benefit of layers for third-party CSS?

**Answer:** You can place it in a lower layer so utilities override it without !important

Control specificity.

### [03 · Custom Utilities, Functional Utilities &amp; Plugins](level-3/03-custom-utilities-plugins.md)

**Q89.** What does @plugin do in v4?

**Answer:** Loads a Tailwind plugin (such as typography or forms) from CSS

No config file needed.

**Q90.** What can a plugin add?

**Answer:** Utilities, variants and base styles

Written with the plugin API.

**Q91.** When should you write a plugin instead of @utility?

**Answer:** When you need programmatic generation (many values) or shared distribution

Simple one-offs use @utility.

**Q92.** How do you add a custom variant in CSS?

**Answer:** @custom-variant name (selector);

For example themes or states.

### [04 · Accessibility with Tailwind](level-3/04-accessibility.md)

**Q93.** What does sr-only do?

**Answer:** Hides content visually but keeps it available to screen readers

Use for labels and skip links.

**Q94.** Why prefer native elements like &lt;button&gt; over styled divs?

**Answer:** They provide semantics, keyboard support and focus behaviour for free

Add ARIA only when needed.

**Q95.** What should focus styles guarantee?

**Answer:** A clearly visible indicator for keyboard users

Don't remove outlines without replacement.

**Q96.** What does the motion-safe: variant do?

**Answer:** Applies styles only when the user has not requested reduced motion

Pair with motion-reduce.

### [05 · Long-Form Content &amp; the Typography Plugin](level-3/05-typography-plugin.md)

**Q97.** What does the typography plugin's prose class do?

**Answer:** Applies beautiful defaults to unstyled HTML such as markdown content

Use prose-lg, dark:prose-invert.

**Q98.** Why use prose for CMS or markdown output?

**Answer:** You cannot add utilities to generated HTML

Style once at the container.

**Q99.** How do you customise prose?

**Answer:** Element modifiers like prose-a:text-blue-600 or theme variables

Keep it consistent.

**Q100.** How do you opt an element out of prose styles?

**Answer:** Wrap it in not-prose

Useful for embedded components.

### [06 · Tailwind in Component Frameworks: Class Merging &amp; Variant APIs](level-3/06-frameworks-class-merging.md)

**Q101.** Why use a class-merging helper such as tailwind-merge?

**Answer:** Later conflicting utilities should win (for example p-2 overriding p-4) which plain concatenation cannot guarantee

CSS order, not class order, decides.

**Q102.** What does clsx or cn() do?

**Answer:** Builds class strings conditionally from arrays, objects and falsy values

Combine with tailwind-merge.

**Q103.** What does a variants helper like cva provide?

**Answer:** Typed component variants (size, intent) mapped to class lists

Keeps components consistent.

**Q104.** Why not build class names by string concatenation?

**Answer:** The scanner cannot see partial names, so styles go missing

Use complete class strings.

### [07 · Building a Design System on Tailwind](level-3/07-design-systems.md)

**Q105.** What is a design system built on Tailwind?

**Answer:** Tokens in @theme plus shared components and documented usage rules

Tokens are the contract.

**Q106.** Why constrain the palette and spacing scale?

**Answer:** Consistency and faster decisions

Reset defaults you don't want.

**Q107.** How do you document components?

**Answer:** Examples with usage, variants and accessibility notes (Storybook or similar)

Make it discoverable.

**Q108.** How should design tokens change over time?

**Answer:** Versioned and communicated, with deprecations before removal

Treat as an API.

### [08 · Migrating from v3 to v4](level-3/08-migrating-v3-v4.md)

**Q109.** What replaced the three @tailwind directives in v4?

**Answer:** A single @import "tailwindcss"

Migrate with the upgrade tool.

**Q110.** Where does configuration live in v4?

**Answer:** In CSS (@theme, @plugin, @source); tailwind.config.js is optional via @config

CSS-first configuration.

**Q111.** What happened to bg-opacity-50 style utilities?

**Answer:** They were removed; use the slash opacity modifier such as bg-black/50

Check the upgrade guide.

**Q112.** What should you do before upgrading a large project?

**Answer:** Run the official upgrade tool on a branch, review the diff and test visually

Several defaults changed.

### [09 · Production Builds &amp; Performance](level-3/09-production-performance.md)

**Q113.** What affects production CSS size most?

**Answer:** Only used utilities are emitted, so size grows with distinct classes, not with the framework

Typically small.

**Q114.** Which step minimises CSS for production?

**Answer:** Build with minification (the CLI --minify or bundler) and serve compressed

Gzip or Brotli.

**Q115.** How do you avoid layout shift from fonts?

**Answer:** Preload critical fonts and use font-display: swap with sensible fallbacks

Measure with Lighthouse.

**Q116.** Why cache static CSS aggressively?

**Answer:** Fingerprinted filenames let browsers cache for a long time

Bust on change.

### [10 · Project — A Component Library](level-3/10-project-component-library.md)

**Q117.** What does the component library project produce?

**Answer:** A set of accessible, themeable components built from utilities and tokens

Level 3 capstone.

**Q118.** Which tests help a component library?

**Answer:** Visual regression, accessibility checks and unit tests of variant logic

Automate in CI.

**Q119.** How do you version a library?

**Answer:** Semantic versioning with a changelog

Communicate breaking changes.

**Q120.** What belongs in a component's API?

**Answer:** Props for variants, sizes, states and an escape hatch for classes

Keep it small.

## Level 4 · Master

### [01 · Conventions at Scale: Class Order, Linting &amp; Review](level-4/01-conventions-at-scale.md)

**Q121.** How do you keep class lists readable at scale?

**Answer:** Use the official Prettier plugin to sort classes and extract components

Consistent ordering reduces diffs.

**Q122.** What is a good linting practice?

**Answer:** Lint for conflicting or unknown classes with an ESLint plugin

Catch typos early.

**Q123.** What belongs in contribution guidelines?

**Answer:** Naming, token usage, component structure and review checklist

Shared conventions.

**Q124.** Why define when to extract a component?

**Answer:** Prevent both premature abstraction and copy-paste sprawl

Rule of three.

### [02 · Multi-Brand &amp; Runtime Theming](level-4/02-multi-brand-theming.md)

**Q125.** How can multiple brands reuse the same components?

```html
@theme {
  --color-brand: oklch(0.6 0.2 250);
}

/* each brand overrides the variable on a wrapper: [data-brand="b"] { --color-brand: ... } */
<div class="bg-brand">
```

**Answer:** background-color: var(--color-brand)

Components use semantic tokens whose values switch per brand scope. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q126.** What is the benefit of CSS variables for theming?

**Answer:** Values can change at runtime per scope without rebuilding CSS

Per-tenant themes.

**Q127.** How do you scope a theme to part of the page?

**Answer:** Set the variables on a wrapper element via data attribute or class

Children inherit.

**Q128.** What should components avoid hard-coding?

**Answer:** Brand colours and spacing; use tokens

Tokens carry brand.

### [03 · RTL, Logical Utilities &amp; Internationalization](level-4/03-rtl-logical.md)

**Q129.** What does ps-4 compile to?

```html
<div class="ps-4">
```

**Answer:** padding-inline-start: calc(var(--spacing) * 4)

Logical utilities adapt to writing direction. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q130.** What does ms-auto compile to?

```html
<div class="ms-auto">
```

**Answer:** margin-inline-start: auto

Use instead of ml-* for RTL support. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q131.** What does rtl:space-x-reverse relate to?

```html
<div class="rtl:flex-row-reverse">
```

**Answer:** &amp;:where(:dir(rtl), [dir="rtl"], [dir="rtl"] *)

rtl: and ltr: variants target direction. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q132.** Why prefer logical properties for international sites?

**Answer:** Layouts mirror automatically for right-to-left languages

start/end beat left/right.

### [04 · Print, Contrast, Motion &amp; Other Media Variants](level-4/04-media-variants.md)

**Q133.** What does print:hidden compile to?

```html
<div class="print:hidden">
```

**Answer:** @media print

Hide navigation when printing. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q134.** What does motion-reduce:transition-none use?

```html
<div class="motion-reduce:transition-none">
```

**Answer:** @media (prefers-reduced-motion: reduce)

Honour user preference. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q135.** What does contrast-more:border-2 use?

```html
<div class="contrast-more:border-2">
```

**Answer:** @media (prefers-contrast: more)

Support high contrast preferences. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q136.** What does portrait:flex-col respond to?

```html
<div class="portrait:flex-col">
```

**Answer:** @media (orientation: portrait)

Orientation variant. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

### [05 · Modern CSS Through Tailwind](level-4/05-modern-css-features.md)

**Q137.** What does text-balance compile to?

```html
<div class="text-balance">
```

**Answer:** text-wrap: balance

Evens out line lengths in headings. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q138.** What does aspect-video compile to?

```html
<div class="aspect-video">
```

**Answer:** aspect-ratio: var(--aspect-video)

Reserve space for media without padding hacks. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q139.** What does inset-0 produce?

```html
<div class="inset-0">
```

**Answer:** inset: 0px

Shorthand for top, right, bottom and left. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q140.** When can you rely on newer CSS features like :has()?

**Answer:** When your browser support policy covers them, with graceful fallbacks otherwise

Check support data.

### [06 · Working with Existing CSS &amp; Third-Party Components](level-4/06-interop-existing-css.md)

**Q141.** How do you mix Tailwind with existing CSS?

**Answer:** Import it into layers so existing rules and utilities have predictable precedence

@layer controls order.

**Q142.** What does the important modifier look like in v4?

**Answer:** A trailing exclamation mark such as p-4!

The leading form is deprecated.

**Q143.** How do you migrate gradually from legacy CSS?

**Answer:** Introduce Tailwind for new components and refactor old ones as you touch them

Avoid long-lived branches.

**Q144.** What prefix option avoids clashes with existing class names?

**Answer:** @import "tailwindcss" prefix(tw); giving tw:flex

v4 uses a variant-style prefix.

### [07 · Tailwind with Server-Rendered Templates &amp; Static Sites](level-4/07-server-templates.md)

**Q145.** How do you use Tailwind with server templates (Rails, Django, Blade)?

**Answer:** Point @source at template paths and use complete class names in templates

The scanner reads any text.

**Q146.** What do you do about classes built from backend variables?

**Answer:** Map each value to a full class string or safelist it with @source inline()

Static strings are required.

**Q147.** Why include the built CSS in the deployment artifact?

**Answer:** Production needs compiled CSS; the build runs in CI

Don't compile at runtime.

**Q148.** How do you handle partials and components across templates?

**Answer:** Keep utility strings inside partials so changes happen in one place

Reuse via includes.

### [08 · Debugging Tailwind](level-4/08-debugging.md)

**Q149.** A class doesn't apply. What is the first thing to check?

**Answer:** That the class name appears as a complete string in a scanned file and the build is running

Check @source and DevTools for the rule.

**Q150.** How can you find where a utility is overridden?

**Answer:** Inspect the element in DevTools to see which rule and layer wins

Check specificity and order.

**Q151.** Why might a styled element appear unstyled in production only?

**Answer:** Dynamic class names or paths missing from @source were not scanned

Compare builds.

**Q152.** What does `--watch` do in the CLI?

**Answer:** Rebuilds CSS when files change during development

Not for production.

### [09 · Testing Tailwind UIs](level-4/09-testing.md)

**Q153.** What can you test about Tailwind styles?

**Answer:** Behaviour via accessibility and visual regression tests, not individual utility output

Test outcomes.

**Q154.** What is visual regression testing?

**Answer:** Comparing screenshots against baselines to catch unintended changes

Use stable fonts and data.

**Q155.** Why run axe or similar accessibility checks in CI?

**Answer:** Catch contrast, label and ARIA issues early

Complement manual testing.

**Q156.** How do you make visual tests stable?

**Answer:** Freeze data and time, disable animations and use consistent rendering environments

Reduce flake.

### [10 · Capstone — A SaaS Marketing Site &amp; App Shell](level-4/10-capstone.md)

**Q157.** What is the capstone's aim?

**Answer:** A production-ready design system or app with tokens, components, accessibility, theming and docs

Bring it together.

**Q158.** What evidence shows quality?

**Answer:** Accessibility audits, visual tests, performance budgets and documentation

Measure.

**Q159.** What belongs in the README?

**Answer:** Setup, tokens, component usage, contribution and release process

Handover.

**Q160.** How do you manage change in the system?

**Answer:** Semantic versions, a changelog and deprecation policy

Treat as a product.

## Mixed questions across all levels

**Q161.** What does this compile to?

```html
<div class="p-4 gap-2">
```

**Answer:** padding: calc(var(--spacing) * 4); gap: calc(var(--spacing) * 2)

Utilities multiply the --spacing theme variable, so the final size follows the theme. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q162.** Which breakpoint rule does md:flex emit in v4?

```html
<div class="md:flex">
```

**Answer:** @media (width &gt;= 48rem) { .md\:flex { display: flex

Breakpoints are mobile-first and rem-based. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q163.** How is dark mode applied by default?

```html
<div class="dark:bg-gray-900">
```

**Answer:** @media (prefers-color-scheme: dark)

The default dark variant follows the OS preference; override it with @custom-variant for a manual toggle. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q164.** What does w-[13px] produce?

```html
<div class="w-[13px]">
```

**Answer:** width: 13px

Arbitrary values are emitted as written. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q165.** What does bg-black/50 compile to?

```html
<div class="bg-black/50">
```

**Answer:** background-color: color-mix(in oklab, var(--color-black) 50%, transparent)

The slash modifier sets the colour's alpha. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q166.** What does hover:bg-red-500 do on touch devices?

```html
<div class="hover:bg-red-500">
```

**Answer:** @media (hover: hover) { &amp;:hover { background-color: var(--color-red-500)

v4 only applies hover styles where hover is supported. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS.

**Q167.** What do you put in a CSS file to start Tailwind v4?

**Answer:** @import "tailwindcss";

One import replaces the three v3 directives.

**Q168.** Why does `bg-${color}-500` fail to produce a style?

**Answer:** The scanner looks for complete class names, so partial strings are not found

Map values to full class names or safelist them with @source inline().

**Q169.** When should you use @apply?

**Answer:** Sparingly, for styling markup you cannot add classes to; prefer components for reuse

Heavy @apply undermines utility-first.

**Q170.** Where do design tokens live in v4?

**Answer:** In CSS custom properties defined in @theme, which also generate utilities

CSS-first configuration.

**Q171.** How do you reuse a button style across an app?

**Answer:** Extract a framework component or template partial and keep utilities there

One source of truth.

**Q172.** Why use tailwind-merge with a component's class prop?

**Answer:** Conflicting utilities (p-2 vs p-4) are resolved by CSS order, not class order, so merging picks the intended one

cn() = clsx + tailwind-merge.

**Q173.** What does motion-reduce: let you do?

**Answer:** Turn off or soften animations for users who prefer reduced motion

Accessibility.

**Q174.** What makes a form control accessible?

**Answer:** A real label tied to the input, visible focus styles and clear error text

Utilities style, semantics inform.

**Q175.** How should you keep utility class order consistent?

**Answer:** Use the official Prettier plugin to sort classes

Reduces noisy diffs.

**Q176.** What does the container query variant @md: respond to?

**Answer:** The width of the nearest @container ancestor, not the viewport

Components adapt to the space they get.

**Q177.** What does rtl: help with and what is better?

**Answer:** Direction-specific overrides; prefer logical utilities such as ps-4 and ms-auto that adapt automatically

Logical properties mirror for RTL.

**Q178.** How do you theme several brands with the same components?

**Answer:** Components use semantic tokens (such as bg-brand) whose variable values are switched per scope

CSS variables change at runtime.

**Q179.** What should you check when a class does not apply?

**Answer:** That it appears as a complete string in a scanned file, then inspect the generated rule in DevTools

Check @source and layer order.

**Q180.** What shows a Tailwind design system is production-ready?

**Answer:** Accessibility and visual-regression tests in CI, documentation and a versioned release process

Treat tokens as an API.

## Practical coding questions

### A responsive card: classes and the CSS they generate

Write the class list for a card (rounded, white, padded, shadowed; a row from the md breakpoint with a gap; gray hover and dark-mode backgrounds), then build it with the Tailwind v4 CLI and read the utilities layer of the generated CSS. The expected output below is the generated utilities layer from a real build; compare the order of rules and the shape of the variants.

Starter code:

```
/* input.css */
@import "tailwindcss" source(none);
@source "./index.html";

<!-- index.html: TODO - write the one-line class list for a card that
     - has extra-large rounded corners, a white background, 1.5rem padding (p-6) and a medium shadow
     - from the md breakpoint up is a row (flex) with vertically centred items and a 1.5rem gap (6 spacing steps)
     - gets a gray-50 background on hover and a gray-900 background in dark mode -->
<article class="TODO">
  ...
</article>

/* build */
npx @tailwindcss/cli -i input.css -o out.css

/* Then read the utilities layer of out.css: what rules did Tailwind generate for your classes, in what order? */
```

Expected output:

```text
@layer utilities {
  .rounded-xl {
    border-radius: var(--radius-xl);
  }
  .bg-white {
    background-color: var(--color-white);
  }
  .p-6 {
    padding: calc(var(--spacing) * 6);
  }
  .shadow-md {
    --tw-shadow: 0 4px 6px -1px var(--tw-shadow-color, rgb(0 0 0 / 0.1)), 0 2px 4px -2px var(--tw-shadow-color, rgb(0 0 0 / 0.1));
    box-shadow: var(--tw-inset-shadow), var(--tw-inset-ring-shadow), var(--tw-ring-offset-shadow), var(--tw-ring-shadow), var(--tw-shadow);
  }
  @media (hover: hover) {
    .hover\:bg-gray-50:hover {
      background-color: var(--color-gray-50);
    }
  }
  @media (width >= 48rem) {
    .md\:flex {
      display: flex;
    }
    .md\:items-center {
      align-items: center;
    }
    .md\:gap-6 {
      gap: calc(var(--spacing) * 6);
    }
  }
  @media (prefers-color-scheme: dark) {
    .dark\:bg-gray-900 {
      background-color: var(--color-gray-900);
    }
  }
}
```

??? success "Show a solution"

    ```
    /* input.css */
    @import "tailwindcss" source(none);
    @source "./index.html";

    <!-- index.html -->
    <article class="rounded-xl bg-white p-6 shadow-md md:flex md:items-center md:gap-6 hover:bg-gray-50 dark:bg-gray-900">
      ...
    </article>

    /* build */
    npx @tailwindcss/cli -i input.css -o out.css
    ```

    Each class becomes one rule; variants wrap or qualify the selector (a media query for md:, :hover inside @media (hover: hover), and a prefers-color-scheme query for dark:). Verified by compiling the classes with Tailwind CSS v4.3 and printing the utilities layer.

### Design tokens, a custom utility and a custom variant

Define theme tokens, a custom breakpoint and spacing base in @theme, add a custom utility with @utility and a custom variant with @custom-variant, then compile classes that use them. The expected output shows the generated utilities and the theme variables Tailwind kept because they were used.

Starter code:

```
/* input.css */
@import "tailwindcss" source(none);
@source "./index.html";

/* TODO: in an @theme block define
     --color-brand: oklch(0.62 0.19 255);  --color-brand-dark: oklch(0.45 0.17 255);
     --breakpoint-3xl: 120rem;             --spacing: 0.5rem;
   TODO: a custom utility text-pretty-balance that sets text-wrap: balance (@utility)
   TODO: a custom variant theme-night that matches [data-theme=night] and its descendants (@custom-variant)  */

<!-- index.html -->
<div class="bg-brand text-brand-dark p-4 3xl:p-8 theme-night:bg-brand-dark text-pretty-balance">...</div>

/* Then read out.css: which rules and which theme variables were generated, and what does p-4 become with the new --spacing? */
```

Expected output:

```text
@layer utilities {
  .bg-brand {
    background-color: var(--color-brand);
  }
  .p-4 {
    padding: calc(var(--spacing) * 4);
  }
  .text-pretty-balance {
    text-wrap: balance;
  }
  .text-brand-dark {
    color: var(--color-brand-dark);
  }
  @media (width >= 120rem) {
    .\33 xl\:p-8 {
      padding: calc(var(--spacing) * 8);
    }
  }
  .theme-night\:bg-brand-dark:where([data-theme=night], [data-theme=night] *) {
    background-color: var(--color-brand-dark);
  }
}

/* theme variables emitted (only those used) */
--spacing: 0.5rem
--color-brand: oklch(0.62 0.19 255)
--color-brand-dark: oklch(0.45 0.17 255)
```

??? success "Show a solution"

    ```
    /* input.css */
    @import "tailwindcss" source(none);
    @source "./index.html";
    @theme {
      --color-brand: oklch(0.62 0.19 255);
      --color-brand-dark: oklch(0.45 0.17 255);
      --breakpoint-3xl: 120rem;
      --spacing: 0.5rem;
    }
    @utility text-pretty-balance { text-wrap: balance; }
    @custom-variant theme-night (&:where([data-theme=night], [data-theme=night] *));

    <!-- index.html -->
    <div class="bg-brand text-brand-dark p-4 3xl:p-8 theme-night:bg-brand-dark text-pretty-balance">...</div>
    ```

    Variables in @theme become real CSS custom properties and generate matching utilities (bg-brand, 3xl:p-8); @utility and @custom-variant add your own. Only the variables actually used are emitted, and p-4 is now calc(var(--spacing) * 4) with the 0.5rem base. Verified by compiling with Tailwind CSS v4.3.

---

More free practice: the [Tailwind CSS Mastery Path](index.md) curriculum, the [Interview Prep Pack](https://learnmasterypath.com/interview-pack/) and the [free resume ATS check](https://learnmasterypath.com/resume/).
