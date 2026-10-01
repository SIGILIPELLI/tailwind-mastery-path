---
description: "Extend Tailwind v4 with static and functional @utility definitions using --value() and --modifier(), and with JavaScript plugins."
---

# 03 · Custom Utilities, Functional Utilities & Plugins

Level 2 · 05 introduced `@utility` for one-off helpers like `content-auto`. This lesson
goes further: utilities that take values (`tab-4`, `tab-github`, `tab-[3]`), utilities
with modifiers (`text-stroke-2/sky-600`), and JavaScript plugins for when CSS alone isn't
enough. Everything you define this way behaves like a built-in: it works with every
variant, it's tree-shaken, and it's sorted with the rest.

## Static utilities (recap)

```css
@utility scrollbar-hidden {
  scrollbar-width: none;
  &::-webkit-scrollbar { display: none; }
}
```

The name has no `*`, so the class is exactly `scrollbar-hidden`, plus any variant:
`md:scrollbar-hidden`, `hover:scrollbar-hidden`.

## Functional utilities: `name-*` and `--value()`

Add `-*` to the name and the utility accepts a value. Inside, `--value()` says **which
kinds of values are allowed and how to resolve them**:

```css title="src/app.css"
@theme {
  --tab-size-github: 8;
}

@utility tab-* {
  tab-size: --value(--tab-size-*, integer, [integer]);
}
```

`--value()` lists three sources, tried in order:

- `--tab-size-*`: a theme key, so `tab-github` resolves `--tab-size-github`
- `integer`: a bare number, so `tab-4` becomes `tab-size: 4`
- `[integer]`: an arbitrary value in brackets, so `tab-[3]` becomes `tab-size: 3`

Compiled with `tab-4 tab-github tab-[3] tab-foo md:tab-2` in the HTML:

```css
.tab-4 { tab-size: 4; }
.tab-\[3\] { tab-size: 3; }
.tab-github { tab-size: var(--tab-size-github); }
@media (width >= 48rem) {
  .md\:tab-2 { tab-size: 2; }
}
```

`tab-foo` produced nothing: `foo` isn't a theme key, an integer or a bracket value, so
the candidate is rejected just like a typo in a built-in.

The bare value types you can use include `integer`, `number`, `percentage` and
`ratio`. In brackets you can also accept types like `[length]`, `[color]`, `[url]`,
`[angle]` and `[*]` (anything). Bare values can't be lengths: Tailwind doesn't let
`stroke-2px` style names happen. If you want numbers to mean pixels or spacing units,
do the maths in the declaration:

```css
@utility opacity-step-* {
  opacity: calc(--value(integer) * 10%);
}
```

`opacity-step-3` compiled to `opacity: calc(3 * 10%);`.

## Several declarations, several value types

A declaration whose `--value()` can't be resolved is **dropped**, while the others stay.
That lets one utility accept different kinds of values through different declarations:

```css
@utility text-stroke-* {
  -webkit-text-stroke-width: --value([length]);
  -webkit-text-stroke-width: calc(--value(integer) * 1px);
  -webkit-text-stroke-color: --modifier(--color-*, [color]);
}
```

Results:

```css
.text-stroke-2 { -webkit-text-stroke-width: calc(2 * 1px); }
.text-stroke-\[0\.5px\] { -webkit-text-stroke-width: 0.5px; }
.text-stroke-2\/sky-600 {
  -webkit-text-stroke-width: calc(2 * 1px);
  -webkit-text-stroke-color: var(--color-sky-600);
}
```

`text-stroke-2` kept only the integer line (the bracket-length line couldn't resolve,
and there was no modifier). `text-stroke-[0.5px]` kept only the length line.

## Modifiers: `--modifier()`

The part after a `/` is the **modifier**, the same slot that holds the opacity in
`bg-sky-600/50`. `--modifier()` resolves it with the same rules as `--value()`. Above,
`--modifier(--color-*, [color])` lets `text-stroke-2/sky-600` set a colour from the theme
or `text-stroke-2/[#333]` set an arbitrary one. When no modifier is given, declarations
using `--modifier()` are dropped.

## When to write a utility vs. use an arbitrary property

- **One place, once:** an arbitrary property `[tab-size:4]` is fine.
- **Used in several places, or you want theme-driven values:** a functional `@utility`.
  It gives you a name, validation (bad values generate nothing) and theme integration.
- **It's really a component** (many properties, specific to one UI element): not a
  utility at all. Use a component or `@layer components` (Level 2 · 05).

## JavaScript plugins

`@utility` covers most needs. Reach for a JavaScript plugin when you need logic CSS can't
express: generating utilities from a data structure, sharing a set of utilities and
variants across projects as an npm package, or reusing a plugin written for v3.

```js title="plugins/layout.mjs"
// A `hocus:` variant (hover or keyboard focus) and `bleed-*` utilities
import plugin from "tailwindcss/plugin";

export default plugin(function ({ addVariant, matchUtilities, theme }) {
  addVariant("hocus", ["&:hover", "&:focus-visible"]);

  matchUtilities(
    {
      bleed: (value) => ({
        marginInline: `calc(${value} * -1)`,
        paddingInline: value,
      }),
    },
    { values: theme("spacing") }
  );
});
```

Load it from CSS (the path is relative to the CSS file):

```css title="src/app.css"
@import "tailwindcss";
@plugin "../plugins/layout.mjs";
```

Compiled with `hocus:underline bleed-4 bleed-[2rem] md:bleed-6`:

```css
.bleed-4 { margin-inline: calc(1rem * -1); padding-inline: 1rem; }
.bleed-\[2rem\] { margin-inline: calc(2rem * -1); padding-inline: 2rem; }
@media (width >= 48rem) {
  .md\:bleed-6 { margin-inline: calc(1.5rem * -1); padding-inline: 1.5rem; }
}
.hocus\:underline:hover { text-decoration-line: underline; }
.hocus\:underline:focus-visible { text-decoration-line: underline; }
```

Things to notice:

- `matchUtilities` got arbitrary values (`bleed-[2rem]`) for free.
- `theme("spacing")` returned **static** values (`1rem`), not `var(--spacing)` maths.
  The JavaScript plugin API is the v3-compatible one, so it reads a resolved copy of the
  theme. Prefer CSS `@utility` with `--value(--spacing-*)` if you want utilities that
  follow theme variables at runtime.
- Our custom `hocus:` used a raw `&:hover`, so it **isn't** wrapped in
  `@media (hover: hover)` like the built-in `hover:` is. Custom variants do exactly what
  you write.
- We first saved the file as `.js` in a folder whose `package.json` didn't say
  `"type": "module"`. It still worked, but Node printed a warning about loading an ES
  module. Use the `.mjs` extension (or set `"type": "module"`) to avoid it.

Official plugins you might load this way include `@tailwindcss/typography` (lesson 05)
and `@tailwindcss/forms` (Level 2 · 08).

## Worked example: a small utility set for a docs site

```css title="src/app.css"
@import "tailwindcss";

@theme {
  --tab-size-code: 2;
  --color-mark: oklch(92% 0.12 95);
}

/* Code blocks */
@utility tab-* {
  tab-size: --value(--tab-size-*, integer, [integer]);
}

/* Highlight text, optionally with a colour modifier */
@utility highlight {
  background-image: linear-gradient(transparent 60%, var(--color-mark) 60%);
}
@utility highlight-* {
  background-image: linear-gradient(transparent 60%, --value(--color-*, [color]) 60%);
}

/* Hide a horizontal scrollbar but keep scrolling */
@utility scrollbar-hidden {
  scrollbar-width: none;
  &::-webkit-scrollbar { display: none; }
}
```

```html
<pre class="tab-code overflow-x-auto rounded-lg bg-gray-950 p-4 text-sm text-gray-100">…</pre>
<p>Run <span class="highlight">npm install</span> before anything else.</p>
<p class="highlight-sky-200">A blue highlight from the palette.</p>
<nav class="flex gap-4 overflow-x-auto scrollbar-hidden">…</nav>
```

Each helper is small and single-purpose, and none would make sense as a component.

## How It Actually Works

A CSS `@utility` definition is registered in the same table the built-in utilities
use. A static name registers one exact class. A name ending in `-*` registers a
**functional** utility: when the compiler parses a candidate like `tab-github`, it
splits off the root `tab` and the value `github`, finds your definition, and evaluates
each declaration. `--value(...)` tries its sources in order: theme namespaces (it looks
up `--tab-size-github`), then bare types (is `github` an integer? no), then bracket
types (only if the value was in brackets). If no source matches, that declaration is
removed; if all declarations are removed, the candidate produces nothing.

JavaScript plugins run once when the CSS is loaded. Their `addUtilities`,
`matchUtilities`, `addVariant` and `addBase` calls register entries in the same tables,
and from then on candidates are matched against them like any other utility. The
registry is the only thing that knows the difference.

## Common mistakes

- **Forgetting `-*`** on a functional utility name, so `tab-4` never matches.
- **Expecting bare lengths** (`stroke-2px`). Bare values are numbers; convert with
  `calc()` or accept `[length]`.
- **A typo in the theme namespace** (`--value(--tabsize-*)`), so theme values never
  resolve.
- **Writing component-sized utilities** that should be components.
- **Expecting custom variants to behave like built-ins** (the `@media (hover: hover)`
  wrapper, for example). They do exactly what you define.
- **Using `theme()` in a JS plugin and expecting runtime variables.** It returns resolved
  values.

## Exercise

1. Write `@utility tab-*` and use `tab-2`, `tab-[3]` and a theme value. Check that
   `tab-foo` generates nothing.
2. Write a `highlight-*` utility that accepts theme colours and arbitrary colours, then
   use it with `hover:` and `md:`.
3. Write a JavaScript plugin with a `hocus:` variant. Then rewrite the variant as
   `@custom-variant` in CSS (Level 2 · 06) and compare.
4. Convert `bleed-*` from the JavaScript plugin into a CSS `@utility` using
   `--value(--spacing-*, [length])` and `calc(var(--spacing) * --value(number))`, and
   compare the generated CSS.
