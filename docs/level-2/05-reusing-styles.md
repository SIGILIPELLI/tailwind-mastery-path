---
description: "Avoid repeating long class lists: extract components first, then use @apply, @layer components, @utility and @reference correctly."
---

# 05 · Reusing Styles: Components, @apply & @utility

The first complaint about Tailwind is always the same: "my button has fifteen classes
and I use it in forty places". The answer is rarely "move the classes back into CSS". It's
usually "make the button a component". Tailwind does have CSS-side tools for reuse
(`@apply`, `@layer components`, `@utility`), and they're the right choice in some
situations. This lesson goes through the options in the order you should consider them,
and shows exactly what each one compiles to.

## Option 1: don't repeat it in the first place

Before reaching for any directive, check whether the repetition is real.

- **Loops.** Eight nav links rendered by a loop are written once:

    ```html
    {% for item in nav %}
      <a href="{{ item.url }}" class="rounded-md px-3 py-2 text-sm font-medium
               text-gray-700 hover:bg-gray-100">{{ item.label }}</a>
    {% endfor %}
    ```

- **Multi-cursor editing.** If the repetition is within one file (a table's cells, say),
  your editor can change all copies at once.

- **Components and partials.** This is the main answer. A button that appears in forty
  places should be one React/Vue/Svelte component, one Blade/Jinja/ERB partial, or one
  template macro:

    ```jsx title="Button.jsx"
    export function Button({ children, ...props }) {
      return (
        <button
          className="rounded-md bg-sky-700 px-4 py-2 font-medium text-white
                     hover:bg-sky-800 focus-visible:outline-2 focus-visible:outline-offset-2
                     focus-visible:outline-sky-600 disabled:opacity-50"
          {...props}
        >
          {children}
        </button>
      );
    }
    ```

A component reuses the *markup* as well as the classes: the `<button>` element, its
attributes, its icon slot. That's something no CSS technique can do. Level 3 · 06 covers
variants (primary/secondary, sizes) and merging classes passed in from outside.

## Option 2: `@apply` inside your own CSS

`@apply` copies the declarations of utilities into a rule you write. It's useful when you
need a real CSS class: markup you can't turn into components (Markdown output, a CMS
template, a third-party widget), or a codebase without a component system.

```css title="src/app.css"
@import "tailwindcss";

@layer components {
  .btn {
    @apply rounded-md px-4 py-2 font-medium hover:bg-sky-700
           focus-visible:outline-2 md:px-6;
  }
}
```

The compiled output (trimmed) shows variants turning into nested rules:

```css
@layer components {
  .btn {
    border-radius: var(--radius-md);
    padding-inline: calc(var(--spacing) * 4);
    padding-block: calc(var(--spacing) * 2);
    font-weight: var(--font-weight-medium);
    &:hover {
      @media (hover: hover) { background-color: var(--color-sky-700); }
    }
    &:focus-visible { outline-style: var(--tw-outline-style); outline-width: 2px; }
    @media (width >= 48rem) { padding-inline: calc(var(--spacing) * 6); }
  }
}
```

That's CSS nesting in the development output, which current browsers support natively.
Building with the CLI's `--minify` flag runs the output through Lightning CSS, and in our
test it flattened the nesting into ordinary rules: `.btn:hover{…}` inside
`@media (hover:hover)`, and a separate `.btn{…}` inside `@media (min-width:48rem)`.

### Why `@layer components`

Tailwind's stylesheet declares layers in this order: `theme, base, components,
utilities`. Rules in a later layer win over rules in an earlier one **regardless of
specificity**. Putting `.btn` in `components` means a utility on the same element always
overrides it:

```html
<a class="btn px-2">Compact button</a>
```

`.btn`'s padding is in `components`, `.px-2` is in `utilities`, so `px-2` wins, even
against `.btn`'s `md:` padding inside a media query. That's exactly the behaviour you
want: a base component class you can tweak with utilities.

If you write the class *outside* any layer, it beats every layered rule, including all
utilities. We gave two links `px-2` (8px) plus a class setting `padding-inline: 2rem`,
one class inside `@layer components` and one unlayered. The layered one computed to
**8px** (the utility won) and the unlayered one to **32px** (the utility lost).
Unlayered CSS always wins over layered CSS, so keep your own component classes in
`@layer components`.

## Option 3: `@utility` for things that behave like utilities

`@utility` registers a **new utility** with Tailwind itself. Unlike a plain class, it
works with every variant, is sorted with the other utilities, and is only emitted when
used:

```css title="src/app.css"
@utility content-auto {
  content-visibility: auto;
}

@utility scrollbar-hidden {
  scrollbar-width: none;
  &::-webkit-scrollbar { display: none; }
}
```

```html
<section class="md:content-auto">…</section>
<div class="flex overflow-x-auto scrollbar-hidden">…</div>
```

`md:content-auto` compiled to `@media (width >= 48rem) { .md\:content-auto
{ content-visibility: auto; } }`. A plain CSS class can't do that.

Use `@utility` for **small, single-purpose** helpers, especially CSS properties Tailwind
doesn't cover. You can use `@apply` inside one, but a multi-property "component" defined
as a utility has a catch: it lives in the utilities layer and has to compete with the
utilities you add next to it. In our test, `class="btn-primary px-2"` (where
`btn-primary` was an `@utility` with five declarations, including padding) happened to
let `px-2` win, because Tailwind's sort placed `.btn-primary` before `.px-2`. That ordering is an implementation detail you
shouldn't build on. Big component styles belong in `@layer components` or, better, a
real component. Level 3 · 03 covers functional utilities that take values
(`tab-4`, `tab-8`).

## Option 4: `@reference` for component-scoped CSS

Vue and Svelte `<style>` blocks, CSS Modules and Astro styles are compiled **separately**
from your main stylesheet. If you use `@apply` in them, Tailwind doesn't know your theme.
We compiled a file containing only `.title { @apply text-brand font-bold; }` and got:

```text
Error: Cannot apply unknown utility class `text-brand`. Are you using CSS modules
or similar and missing `@reference`?
```

`@reference` imports your main CSS **for reference only**: its theme, custom utilities
and variants become available, but none of its CSS is output again:

```vue title="Card.vue"
<style scoped>
@reference "../app.css";

.title {
  @apply text-brand font-bold;
}
</style>
```

With the reference added, the output was just the `.title` rule (plus a couple of
`@property` registrations), with no second copy of Preflight or the theme. If you don't
have custom theme values, `@reference "tailwindcss";` is enough.

A simpler alternative in these files is to skip `@apply` and use the theme variables
directly: `.title { color: var(--color-brand); }`.

## Worked example: Markdown content you can't add classes to

A blog renders Markdown to HTML, so you can't put classes on each `<a>` and `<code>`.
This is a legitimate case for a CSS class:

```css title="src/app.css"
@import "tailwindcss";

@layer components {
  .post-body {
    @apply text-gray-800 leading-7;

    & a { @apply font-medium text-sky-700 underline underline-offset-2 hover:text-sky-900; }
    & code { @apply rounded bg-gray-100 px-1 py-0.5 font-mono text-sm; }
    & h2 { @apply mt-10 mb-3 text-2xl font-semibold tracking-tight; }
    & p + p { @apply mt-4; }
  }
}
```

```html
<article class="post-body mx-auto max-w-prose">
  {{ rendered_markdown | safe }}
</article>
```

The template uses one class; the Markdown elements are styled through nesting. For a
full long-form content style, the Typography plugin (Level 3 · 05) does this job with a
lot more polish.

## How It Actually Works

`@apply` runs at compile time. For each class you list, Tailwind generates the utility as
usual, then takes its declarations and pastes them into your rule. If the utility has
variants, the variant's selector part becomes a nested rule (`&:hover`) and any at-rule
(`@media`) wraps those declarations, which is why the output uses CSS nesting. Nothing
links the two afterwards: the `.btn` rule doesn't depend on `.px-4` existing in the
output.

`@utility` is a different mechanism. It adds an entry to the same registry the built-in
utilities live in. When the scanner finds `md:content-auto`, Tailwind looks up
`content-auto`, finds your definition, applies the `md` variant, and sorts the result
with everything else. That's why custom utilities support variants and why they're
tree-shaken like built-ins.

`@reference` parses the referenced file to build the theme, utilities and variants, then
discards its output. It's import-for-types, applied to CSS.

## Common mistakes

- **Using `@apply` to recreate a component library in CSS** when you have a component
  framework. You lose the markup reuse and add a second place to look for styles.
- **Writing component classes outside `@layer components`.** Unlayered CSS beats every
  utility, so `class="btn px-2"` can't be adjusted.
- **Defining a big multi-property component with `@utility`** and expecting utilities next
  to it to win predictably.
- **`@apply` in Vue/Svelte/CSS Modules without `@reference`**, which fails with
  "Cannot apply unknown utility class".
- **`@import "tailwindcss"` in every component's style block** instead of `@reference`.
  Each one outputs a full copy of Preflight and the theme.
- **Extracting too early.** Wait until a pattern is repeated and stable before naming it.

## Exercise

1. Find the most repeated class list in your Level 1 landing page. Turn it into (a) a
   template loop or partial and (b) a `.btn` class with `@apply`. Which was cleaner?
2. Prove the layer behaviour: put `.btn` in `@layer components`, add `px-2` to one
   button, and confirm it applies. Move `.btn` out of the layer and watch it stop working.
3. Create `@utility scrollbar-hidden` and use it with a responsive variant
   (`md:scrollbar-hidden`).
4. If you have a Vue or Svelte project, write a scoped style block that uses `@apply`
   with a custom colour, see the error, and fix it with `@reference`.
