---
description: "How Tailwind v4's @theme turns CSS variables into design tokens and utilities: namespaces, emitted output, inline and static modes."
---

# 01 · Design Tokens with @theme

In Level 1 every colour, size and font came from Tailwind's default theme. Real projects
have their own brand colours, typefaces and radii. In Tailwind v4 you define them in CSS,
in an `@theme` block, and each one becomes **both** a CSS variable and a family of
utilities. This lesson explains the mechanism: what a theme variable is, how its name
decides which utilities appear, and what ends up in the compiled file. Lesson 02 then
uses it to customise a whole theme.

## A theme variable is a CSS variable with a reserved name

```css title="src/app.css"
@import "tailwindcss";

@theme {
  --color-brand-50: oklch(97% 0.02 160);
  --color-brand-600: oklch(52% 0.13 160);
  --font-display: "Inter Tight", ui-sans-serif, system-ui, sans-serif;
  --radius-card: 0.875rem;
}
```

With just that, these classes work:

```html
<article class="rounded-card bg-brand-50 p-6">
  <h2 class="font-display text-brand-600">Quarterly report</h2>
  <a class="ring-1 ring-brand-600/40">…</a>
</article>
```

The **prefix** of the variable name (its namespace) decides which utilities it feeds:

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

These are the namespaces the default `theme.css` itself uses. Counting its variables
shows how the defaults are distributed: about 288 of them are colours, and the rest are
spread over `--text`, `--font`, `--shadow`, `--radius`, `--breakpoint`, `--container` and
the others above.

A variable whose name isn't in a namespace (say `--my-token`) is allowed in `@theme`,
but it doesn't create any utility.

## What the compiler emits

Compiling the example above with only `bg-brand-50`, `text-brand-600`, `font-display`
and `rounded-card` in the HTML gave:

```css
@layer theme {
  :root, :host {
    /* …the default font variables… */
    --color-brand-50: oklch(97% 0.02 160);
    --color-brand-600: oklch(52% 0.13 160);
    --font-display: "Inter Tight", ui-sans-serif, system-ui, sans-serif;
    --radius-card: 0.875rem;
  }
}
@layer utilities {
  .rounded-card { border-radius: var(--radius-card); }
  .bg-brand-50 { background-color: var(--color-brand-50); }
  .font-display { font-family: var(--font-display); }
  .text-brand-600 { color: var(--color-brand-600); }
}
```

Three behaviours are worth knowing:

1. **Utilities reference the variable**, they don't copy its value. Change
   `--color-brand-600` at runtime (on `:root` or any element) and every utility that uses
   it follows. That's the basis of dark mode and multi-brand theming later on.
2. **Only variables that are used get emitted.** Unused theme variables, including the
   hundreds of default colours, are left out. In our test, `--my-token` (unused) was not
   in the output. A variable counts as used if a generated utility needs it **or** if
   your own CSS refers to it with `var()`. A rule like
   `.note { border-color: var(--color-brand-600); }` kept `--color-brand-600` even when no
   utility used it.
3. **Some values are inlined.** `--shadow-card` didn't appear in `:root` at all; the
   `shadow-card` utility had the shadow written directly into it, because shadows need
   to be rewritten to support `shadow-{color}`.

### `@theme` vs `:root`

Why not just write `:root { --color-brand-600: … }`? You can, and the variable will work
with `var()`, but **no utilities are generated** from `:root`. Use `@theme` for design
tokens that should have classes, and plain `:root` (or any selector) for variables that
are only implementation details or that change at runtime.

Put `@theme` at the top level of the file. It's compile-time configuration, not a style
rule, so wrapping it in a selector or media query doesn't scope it. We tried both
`.x { @theme { --color-y: red; } }` and `@media (min-width: 1px) { @theme { … } }`.
Neither produced an error, and in both cases `--color-y: red` was simply hoisted into
`:root` with the selector and the media query ignored. If you want a value that changes
by context, keep the token in `@theme` and override the variable in a normal rule
(Level 2 · 03 does exactly this for dark mode).

## Keeping unused variables: `@theme static`

Sometimes you *want* every token in the output: JavaScript reads them with
`getComputedStyle`, or a third-party widget is styled with `var(--color-…)` from a file
Tailwind doesn't scan. Use the `static` option:

```css
@theme static {
  --color-chart-1: oklch(65% 0.15 250);
  --color-chart-2: oklch(70% 0.15 150);
}
```

These are always emitted, used or not.

## Referencing other variables: `@theme inline`

A theme variable whose value is another variable can surprise you:

```css
@theme {
  --font-body: var(--font-inter);   /* --font-inter is set by a font loader */
}
```

The utility becomes `font-family: var(--font-body)`, and `--font-body` is defined on
`:root` as `var(--font-inter)`. CSS resolves `var()` **where the variable is defined**,
so if `--font-inter` is set on `<body>` or some other element rather than `:root`, the
`:root` value has already resolved to nothing. `inline` puts the value straight into the
utility instead:

```css
@theme inline {
  --font-body: var(--font-inter);
}
```

Compiled, `font-body` became `font-family: var(--font-inter);`, which is resolved on the
element itself. Use `inline` whenever a theme value refers to a variable that's defined
somewhere other than `:root` (Next.js's `next/font` is a common example).

## Replacing the defaults

To extend the theme you just add variables. To **replace** a whole namespace, reset it
with `initial` and then define your own:

```css
@theme {
  --color-*: initial;
  --color-ink: #111;
  --color-paper: #fafafa;
  --color-accent: oklch(60% 0.2 30);
}
```

After this, `bg-red-500` no longer exists (we confirmed it generated nothing), but
`text-ink` and `bg-paper` do. `--*: initial` resets *everything*, including spacing and
breakpoints, which is rarely what you want.

**Order matters.** Theme blocks are processed top to bottom, and a reset clears every
variable in its namespace defined *before* it, even in another `@theme` block. We put
`@theme static { --color-accent: … }` before a `@theme { --color-*: initial; … }` block,
and `bg-accent` silently stopped existing. Moved after the reset, it worked. Keep the
reset at the top of your theme.

## Worked example: a token file for a small product

```css title="src/app.css"
@import "tailwindcss";

@theme {
  /* Brand */
  --color-brand-50: oklch(97.5% 0.02 165);
  --color-brand-100: oklch(94% 0.05 165);
  --color-brand-500: oklch(66% 0.14 165);
  --color-brand-600: oklch(56% 0.13 165);
  --color-brand-700: oklch(47% 0.11 165);

  /* Type */
  --font-display: "Inter Tight", ui-sans-serif, system-ui, sans-serif;

  /* Shape */
  --radius-card: 0.875rem;
  --shadow-card: 0 1px 2px rgb(0 0 0 / 0.06), 0 4px 12px rgb(0 0 0 / 0.05);

  /* Motion */
  --animate-wiggle: wiggle 0.6s ease-in-out 2;
  @keyframes wiggle {
    0%, 100% { rotate: -3deg; }
    50% { rotate: 3deg; }
  }
}
```

```html
<article class="rounded-card bg-white p-6 shadow-card ring-1 ring-brand-100">
  <h2 class="font-display text-xl font-semibold text-brand-700">Weekly digest</h2>
  <p class="mt-2 text-gray-600">You saved 14 articles this week.</p>
  <button class="mt-4 rounded-md bg-brand-600 px-4 py-2 text-white hover:bg-brand-700
                 hover:animate-wiggle">Open</button>
</article>
```

The `@keyframes` inside `@theme` is emitted only when `animate-wiggle` is actually used,
just like the variables. Everything in the HTML is a normal utility; the brand is defined
in one place, so a colour change is a one-line edit.

## How It Actually Works

When the compiler reads your CSS, it collects every declaration inside `@theme` blocks
into a **theme map**, starting from the defaults in `tailwindcss/theme.css` (which is
itself just a big `@theme` block imported by `@import "tailwindcss"`). Later declarations
overwrite earlier ones with the same name, and a `--namespace-*: initial` deletes every
current key in that namespace.

Each utility plugin looks up its values in particular namespaces. When it sees the class
`bg-brand-600`, the background plugin splits it into root `bg` and value `brand-600`,
looks for `--color-brand-600` in the theme map, finds it, and generates
`background-color: var(--color-brand-600)`. That's why naming is everything: a variable
called `--brand-color-600` would never be found.

After all utilities are generated, the compiler walks the output and your own CSS for
`var(--…)` references and emits only those theme variables into `@layer theme` (unless
`static`). `inline` variables are substituted into the utility at generation time, so
they never appear in `:root`.

## Common mistakes

- **Defining tokens in `:root`** and wondering why `bg-brand-600` doesn't exist. Only
  `@theme` creates utilities.
- **Naming outside a namespace**, like `--brand-600`. It becomes a variable but not a
  utility.
- **Expecting unused theme variables in the output** for JavaScript or unscanned CSS.
  Use `@theme static`.
- **A theme variable pointing at a variable defined on a non-root element**, which
  resolves to nothing. Use `@theme inline`.
- **Resetting a namespace after defining your own values in it.** The reset deletes them.
- **`--*: initial` by accident** when you meant `--color-*: initial`.
- **Nesting `@theme` inside a selector or media query** to scope a token. It's silently
  hoisted to `:root`; override the variable in a normal rule instead.

## Exercise

1. Add a five-step `brand` colour scale and use it for a button and a card. Inspect the
   compiled CSS and find exactly which `--color-brand-*` variables were emitted.
2. Add `--radius-card` and `--shadow-card` and confirm one is in `:root` and the other is
   inlined into its utility.
3. Write a small script that logs
   `getComputedStyle(document.documentElement).getPropertyValue('--color-brand-500')`.
   Make it return a value even though no class uses `brand-500`.
4. Replace the whole colour palette with five colours of your own, and confirm a default
   class like `bg-sky-500` no longer generates anything.
