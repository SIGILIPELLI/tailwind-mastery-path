---
description: "Customise a Tailwind v4 theme: brand colour scales, web fonts, type sizes with built-in line heights, named spacing, containers and breakpoints."
---

# 02 · Customizing Colors, Fonts, Spacing & Breakpoints

Lesson 01 explained how `@theme` works. This one is practical: you'll take the default
theme and turn it into a specific product's visual language, one namespace at a time.
For each change you'll see what it does to the generated CSS, because every change
affects not just the class you wrote but everything built on the same variable.

## Colours: a brand scale

A single brand colour is rarely enough. Buttons need a hover shade, backgrounds need a
pale tint, text on tints needs a dark shade. Define a scale using the same step names as
the built-in palette (50 to 950) so the classes feel familiar:

```css title="src/app.css"
@import "tailwindcss";

@theme {
  --color-brand-50:  oklch(97.5% 0.02 250);
  --color-brand-100: oklch(94% 0.04 250);
  --color-brand-200: oklch(88% 0.07 250);
  --color-brand-500: oklch(62% 0.17 250);
  --color-brand-600: oklch(54% 0.18 250);
  --color-brand-700: oklch(46% 0.16 250);
  --color-brand-900: oklch(30% 0.10 250);
}
```

Why `oklch`? Its three numbers are **lightness**, **chroma** (colourfulness) and **hue**.
Keeping the hue fixed and stepping lightness down gives a scale that looks evenly spaced
to the eye, which is hard to do by hand with hex values. The default Tailwind palette is
defined in `oklch` too (Level 1 · 05).

You don't have to define all eleven steps. Define the ones your design uses; any
`bg-brand-300` you didn't define simply won't generate. Keep the shades you do define
honest: check text contrast (for example `text-brand-700` on `bg-brand-50`) with your
browser's DevTools contrast checker or another WCAG tool before shipping.

**Semantic names** are a second layer some teams add on top:

```css
@theme {
  --color-surface: var(--color-white);
  --color-surface-muted: var(--color-gray-50);
  --color-text: var(--color-gray-900);
  --color-text-muted: var(--color-gray-600);
  --color-primary: var(--color-brand-600);
}
```

That gives `bg-surface` and `bg-primary`, but `--color-text-muted` produces the
awkward `text-text-muted`, because the class is always the utility prefix plus the colour
name. Choose semantic names that read well after any prefix: `--color-fg`,
`--color-fg-muted`, `--color-bg` and `--color-primary` give `text-fg-muted`, `bg-bg` and
`bg-primary`. Semantic colours really pay off with dark mode
(lesson 03), where only their values change.

## Fonts: replacing the default sans

The `font-sans` utility and the page's default font both come from `--font-sans`. The
base styles set `html { font-family: var(--default-font-family, …) }`, and the theme
defines `--default-font-family: var(--font-sans)`. So overriding `--font-sans` changes
the whole site, not just elements with `font-sans`:

```css title="src/app.css"
@import "tailwindcss";

@font-face {
  font-family: "Inter";
  src: url("/fonts/InterVariable.woff2") format("woff2");
  font-weight: 100 900;
  font-display: swap;
}

@theme {
  --font-sans: "Inter", ui-sans-serif, system-ui, sans-serif;
  --font-sans--font-feature-settings: "cv11", "ss01";
  --font-display: "Inter Tight", var(--font-sans);
}
```

Compiled, the theme layer contained:

```css
--font-sans: "Inter", ui-sans-serif, system-ui, sans-serif;
--default-font-family: var(--font-sans);
--default-font-feature-settings: var(--font-sans--font-feature-settings);
--font-sans--font-feature-settings: "cv11", "ss01";
```

and the `font-sans` utility set both `font-family` and `font-feature-settings`. The
double-dash suffix (`--font-sans--font-feature-settings`) is how Tailwind attaches extra
properties to a token. `@font-face` goes **outside** `@theme`. It's ordinary CSS.

Always end a font stack with fallbacks. If the web font fails to load, `ui-sans-serif,
system-ui, sans-serif` gives a reasonable system font instead of the browser's default
serif.

The path in `src: url(...)` is resolved by whatever serves the CSS. With Vite, files in
`public/` are served from `/`. With the CLI, the URL is relative to the output CSS file,
so check where your build writes it.

## Type sizes with built-in line height

A `--text-*` token can carry its own line height, letter spacing and weight, using the
same double-dash suffixes:

```css
@theme {
  --text-hero: clamp(2.5rem, 6vw, 4.5rem);
  --text-hero--line-height: 1.05;
  --text-hero--letter-spacing: -0.02em;
  --text-hero--font-weight: 800;
}
```

The generated utility:

```css
.text-hero {
  font-size: var(--text-hero);
  line-height: var(--tw-leading, var(--text-hero--line-height));
  letter-spacing: var(--tw-tracking, var(--text-hero--letter-spacing));
  font-weight: var(--tw-font-weight, var(--text-hero--font-weight));
}
```

Look at the `var(--tw-leading, …)` pattern. The token's line height is only a
*fallback*. If the element also has `leading-none`, that utility sets `--tw-leading: 1`
and wins, regardless of class order. You get good defaults that are easy to override.
The `clamp()` makes the headline fluid: never smaller than 2.5rem, never larger than
4.5rem, scaling with the viewport in between.

## Spacing: one variable, or named values

Every numeric spacing utility is `calc(var(--spacing) * n)`, and the default is
`--spacing: 0.25rem`. Changing it rescales `p-4`, `gap-6`, `w-12`, `mt-8` and the rest
all at once:

```css
@theme { --spacing: 0.2rem; }   /* p-4 is now 0.8rem instead of 1rem */
```

That's a big change to make on an existing project, since every component's spacing
shifts. It's more useful at the very start of a design system.

For **named** spacing values, add `--spacing-*` tokens:

```css
@theme { --spacing-gutter: 1.5rem; }
```

Compiled: `.p-gutter { padding: var(--spacing-gutter); }`, and `gap-gutter`, `mx-gutter`,
`w-gutter` and the other spacing utilities work as well. Named values are good for
layout constants that should be identical everywhere (page gutters, header height).

## Containers and max widths

`--container-*` tokens feed `max-w-*`, `w-*` and the container query sizes:

```css
@theme { --container-reading: 42rem; }
```

This gave both `.max-w-reading { max-width: var(--container-reading); }` and
`.w-reading`. The defaults run from `--container-3xs` to `--container-7xl`
(`--container-md` is 28rem, for example).

## Breakpoints

As you saw in Level 1 · 09, breakpoints are `--breakpoint-*` tokens. Overriding an
existing one moves it everywhere:

```css
@theme { --breakpoint-sm: 36rem; }
```

`sm:flex` then compiled to `@media (width >= 36rem)`. To start from a clean slate, reset
the namespace first: `--breakpoint-*: initial;` and then define every breakpoint you want.

## Worked example: a complete theme

```css title="src/app.css"
@import "tailwindcss";

@font-face {
  font-family: "Inter";
  src: url("/fonts/InterVariable.woff2") format("woff2");
  font-weight: 100 900;
  font-display: swap;
}

@theme {
  /* Colour */
  --color-brand-50:  oklch(97.5% 0.02 250);
  --color-brand-100: oklch(94% 0.04 250);
  --color-brand-600: oklch(54% 0.18 250);
  --color-brand-700: oklch(46% 0.16 250);
  --color-fg: var(--color-gray-900);
  --color-fg-muted: var(--color-gray-600);
  --color-line: var(--color-gray-200);

  /* Type */
  --font-sans: "Inter", ui-sans-serif, system-ui, sans-serif;
  --text-hero: clamp(2.5rem, 6vw, 4.5rem);
  --text-hero--line-height: 1.05;
  --text-hero--letter-spacing: -0.02em;

  /* Layout */
  --spacing-gutter: 1.5rem;
  --container-page: 72rem;
  --container-reading: 42rem;
  --breakpoint-xs: 30rem;

  /* Shape */
  --radius-card: 0.75rem;
}
```

```html
<main class="mx-auto max-w-page px-gutter">
  <h1 class="text-hero font-bold text-fg">Plan less. Ship more.</h1>
  <p class="mt-4 max-w-reading text-lg text-fg-muted">…</p>
  <div class="mt-8 grid gap-gutter xs:grid-cols-2">
    <section class="rounded-card border border-line p-6">…</section>
    <section class="rounded-card border border-line p-6">…</section>
  </div>
</main>
```

The HTML now talks in the product's vocabulary (`max-w-page`, `px-gutter`, `text-fg`,
`border-line`), and every value lives in one CSS file.

## How It Actually Works

Each utility plugin declares which namespaces it reads. `bg-*` reads `--color-*`, `p-*`
reads `--spacing-*` and falls back to multiplying `--spacing`, `max-w-*` reads
`--container-*` (and `--spacing-*`), and so on. When you add a token, you're adding a
key to a namespace, and every plugin that reads that namespace gets a new value. That's
why one `--color-brand-600` gives you `bg-`, `text-`, `border-`, `ring-`, `fill-` and
more.

Suffixed keys like `--text-hero--line-height` belong to the same token. The `text-*`
plugin looks for those suffixes and adds the extra declarations, wrapped in
`var(--tw-leading, …)` so a separate `leading-*` utility (which sets `--tw-leading`) can
override it. The base layer uses the `--default-*` variables (`--default-font-family`,
`--default-transition-duration`, …) as the site-wide defaults, which is why changing
`--font-sans` reaches elements that never had a font class.

## Common mistakes

- **Semantic names that read badly after a prefix**, like `--color-text-muted` →
  `text-text-muted`.
- **Putting `@font-face` inside `@theme`.** It's normal CSS; keep it outside.
- **Font stacks with no fallbacks.** If the font fails, the page shows the default serif.
- **Changing `--spacing` on a mature project** and rescaling every component at once.
  Add named `--spacing-*` values instead.
- **Skipping contrast checks** on custom shades.
- **Overriding one breakpoint so the names stop matching the order.** Tailwind sorts
  breakpoint rules by value, not by name. We set `--breakpoint-sm: 50rem` (larger than
  `md`'s 48rem), and `sm:` rules were emitted *after* `md:` rules, so `sm:p-1` beat
  `md:p-2` on wide screens. Keep the values increasing in name order.

## Exercise

1. Build a brand scale with five steps at a hue of your choice. Make a button
   (`bg-brand-600 hover:bg-brand-700`) and a tinted callout (`bg-brand-50
   text-brand-900`), and check the callout's contrast.
2. Load a variable web font with `@font-face`, make it the default sans, and confirm in
   DevTools that a paragraph without any font class uses it.
3. Add a `--text-hero` token with a fluid size and its own line height. Then add
   `leading-none` to one heading and confirm it overrides the token's line height.
4. Add `--spacing-gutter` and `--container-page` and refactor the Level 1 landing page
   to use them instead of `px-4 sm:px-6` and `max-w-6xl`.
