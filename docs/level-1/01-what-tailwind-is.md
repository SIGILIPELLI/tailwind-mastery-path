---
description: "What Tailwind CSS really is: a compiler that turns utility classes into CSS, why teams use it, its trade-offs, and what it generates."
---

# 01 · What Tailwind Is (and Isn't)

Tailwind CSS is a **utility-first** CSS framework. Instead of writing a stylesheet full of
component classes (`.card`, `.card-title`) and then applying them in HTML, you apply small,
single-purpose classes directly in your markup:

```html
<article class="rounded-lg border border-gray-200 p-6 shadow-sm">
  <h2 class="text-lg font-semibold text-gray-900">Roasted tomato soup</h2>
  <p class="mt-2 text-sm text-gray-600">45 minutes · serves 4</p>
</article>
```

Each class does one thing: `p-6` sets padding, `text-lg` sets font size and line height,
`text-gray-600` sets colour. You compose a design out of them in the HTML.

That's the part everyone sees. The part that makes it work is less visible: Tailwind is a
**compiler**. It reads your source files, finds the class names you actually used, and
generates exactly the CSS for those — nothing else. This course is about both parts: using
the utilities fluently, and understanding the engine well enough to extend it, debug it and
build design systems on it.

This course teaches **Tailwind CSS v4** (4.3.3 was the current release when it was written).
Version 4 changed a lot compared to v3 — configuration moved from a JavaScript file into
CSS, the engine was rewritten, and many defaults changed. If you're coming from v3,
Level 3 · 08 covers migration.

## What a utility becomes

We compiled the card above with the Tailwind CLI and looked at the generated CSS. Every
class maps to a small, readable rule:

```css
.p-6 {
  padding: calc(var(--spacing) * 6);
}
.text-lg {
  font-size: var(--text-lg);
  line-height: var(--tw-leading, var(--text-lg--line-height));
}
.text-gray-600 {
  color: var(--color-gray-600);
}
```

And the values come from **CSS custom properties** (variables) that Tailwind emits for the
tokens you used:

```css
@layer theme {
  :root, :host {
    --color-gray-600: oklch(44.6% 0.03 256.802);
    --spacing: 0.25rem;
    --text-lg: 1.125rem;
    ...
  }
}
```

A few things are worth noticing already:

- **The spacing scale is one variable.** `p-6` is `var(--spacing) * 6`, i.e. 1.5rem. Change
  `--spacing` and every spacing utility changes with it.
- **Colours are in `oklch()`** — a modern perceptual colour space (the
  [HTML & CSS Mastery Path](https://sigilipelli.github.io/html-css-mastery-path/level-2/02-color-backgrounds/)
  explains it).
- **Everything lives in cascade layers** (`@layer theme`, `@layer utilities`), which
  Level 3 · 02 explains.

## Only what you use

Because Tailwind generates CSS from your source, the output contains only the utilities you
wrote. We compiled two tiny pages:

```text
page with no classes:                  4,614 bytes
page with "p-4 text-lg font-bold":     5,341 bytes
```

The first number is Tailwind's **base styles** ("Preflight", a small reset) and the layer
setup; the utilities for three classes added about 700 bytes. A class you *don't* use —
`bg-red-500`, say — isn't in the output at all. That's why a Tailwind site's CSS stays small
even though the framework offers tens of thousands of possible classes: a typical page
uses a few hundred, and a whole site rarely more than a few thousand.

## Why teams use it

- **No naming.** You don't invent and maintain names like `.card__meta--compact`.
- **Local reasoning.** The styles for an element are on the element. Deleting a component
  deletes its styles; there's no orphaned CSS in some distant file.
- **A shared scale.** Spacing, colour, type and radius come from one set of tokens, so a
  team produces consistent designs without a style guide lecture.
- **A stylesheet that stops growing.** Adding the hundredth component reuses the same
  utilities as the first, so CSS size tracks the *variety* of styles, not the number of
  components.
- **Responsive and state styling inline.** `md:grid-cols-2`, `hover:bg-sky-600`,
  `dark:bg-gray-900` — no jumping between files and media queries.

## The honest trade-offs

- **Long class lists.** Markup gets noisy, especially for interactive components with many
  states. Component frameworks (React, Vue, Svelte, server templates) mitigate this by
  putting each pattern in one component (Level 3 · 06).
- **You still need to know CSS.** `flex`, `grid`, `min-w-0`, `sticky` are CSS concepts with
  shorter names. Tailwind doesn't explain *why* a flex item overflows; CSS knowledge does.
  If CSS is new to you, work through
  [HTML & CSS Mastery Path](https://sigilipelli.github.io/html-css-mastery-path/) alongside
  this course — this course links to it wherever a concept needs depth.
- **A build step.** Something has to scan your files and generate the CSS (lesson 02).
- **Dynamic class names don't work.** Because classes are found by scanning source text,
  building a class name at runtime (`"text-" + size`) produces a class that was never
  generated (Level 3 · 01).
- **It's a design-system tool, not a design.** Tailwind gives you a scale, not taste. The
  default palette and scale are good starting points; real products customise them
  (Level 2 · 01–02).

Tailwind is not a component library. There are no ready-made buttons or modals in the
framework itself; those come from you, or from separate libraries built on top of it.

## Worked example: the same card, two ways

Traditional CSS:

```html
<article class="recipe-card">
  <h2 class="recipe-card__title">Roasted tomato soup</h2>
  <p class="recipe-card__meta">45 minutes · serves 4</p>
</article>
```

```css
.recipe-card { border: 1px solid #e5e7eb; border-radius: 0.5rem; padding: 1.5rem; box-shadow: 0 1px 2px rgb(0 0 0 / 0.05); }
.recipe-card__title { font-size: 1.125rem; font-weight: 600; color: #111827; }
.recipe-card__meta { margin-top: 0.5rem; font-size: 0.875rem; color: #4b5563; }
```

Tailwind:

```html
<article class="rounded-lg border border-gray-200 p-6 shadow-sm">
  <h2 class="text-lg font-semibold text-gray-900">Roasted tomato soup</h2>
  <p class="mt-2 text-sm text-gray-600">45 minutes · serves 4</p>
</article>
```

The visual result is essentially the same. What differs is where decisions live. In the
first version, changing the card means finding `.recipe-card` in the CSS and checking
nothing else uses it. In the second, you change the classes on the element, and nothing
else can be affected. When the card appears in twenty templates, the Tailwind version needs
a component (or a template partial) so the class list isn't copied twenty times — that's
the real discipline of working with Tailwind.

## How It Actually Works

Tailwind v4 is a compiler with three steps:

1. **Scan.** It walks your project's files (respecting `.gitignore`, skipping binary files
   and `node_modules` by default) and extracts every string that *could* be a class name.
   It doesn't parse HTML or JavaScript — it tokenises text, which is why it finds classes in
   templates of any language, and why it can't see names that only exist at runtime.
2. **Match.** Each candidate is checked against Tailwind's utilities and variants.
   `hover:bg-sky-600` is split into a variant (`hover`) and a utility (`bg-sky-600`); the
   utility is resolved against the theme (`--color-sky-600`). Candidates that don't match
   anything (ordinary words like "soup") are discarded silently.
3. **Generate.** Matched utilities become CSS rules, sorted into a deterministic order,
   wrapped in the right layers and media queries, with the theme variables they depend on.
   The core engine is written in Rust and TypeScript and designed for incremental rebuilds,
   so recompiling after an edit usually takes milliseconds — our small builds reported
   "Done in 26ms".

The generated CSS is ordinary CSS. Browsers don't know Tailwind exists; there's no runtime
JavaScript. That means everything from the
[cascade](https://sigilipelli.github.io/html-css-mastery-path/level-1/08-cascade-specificity/)
to browser devtools works exactly as it does for any stylesheet.

## Common mistakes

- **Thinking Tailwind replaces CSS knowledge.** It's CSS with shorter names and a scale.
- **Copy-pasting long class lists** across templates instead of making a component.
- **Constructing class names dynamically.**
- **Fighting the scale** with arbitrary values everywhere (`p-[13px]`) — if a design needs
  new values repeatedly, add them to the theme.
- **Following v3 tutorials with v4.** Configuration, defaults and some class names changed.

## Exercise

1. Pick a small component from a site you use (a notification, a pricing card) and write it
   twice: once with your own CSS classes and once with Tailwind utilities. Keep both files;
   you'll build the Tailwind version in lesson 02.
2. Write down, for each Tailwind class you used, the CSS property and value you expect it to
   produce. You'll check your predictions against the compiler's output in the next lesson.
3. List three places in a project you know where CSS has grown without anyone daring to
   delete it. Would co-locating styles with markup have helped?
