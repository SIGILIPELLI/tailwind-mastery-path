---
description: "Style Markdown and CMS content with the Tailwind Typography plugin: prose sizes and colours, element modifiers, not-prose, custom themes."
---

# 05 · Long-Form Content & the Typography Plugin

Utility classes work when you write the HTML. They don't help with HTML you *don't*
write: a blog post rendered from Markdown, a product description from a CMS, release
notes from an API. That content arrives as bare `<h2>`, `<p>`, `<ul>` and `<a>`
elements, and after Preflight it's an unreadable wall of same-sized, unspaced text.

The official **Typography plugin** solves this with one class, `prose`, which gives all
the descendant elements carefully tuned typographic defaults. This lesson covers how to
install it in v4, what it actually does (measured), how to adjust it per element, how to
opt parts out, and how to give it your own colours.

## Install and use

```bash
npm install -D @tailwindcss/typography
```

```css title="src/app.css"
@import "tailwindcss";
@plugin "@tailwindcss/typography";
```

```html
<article class="prose">
  {{ post.html | safe }}
</article>
```

That's it: headings get sizes and weights, paragraphs get spacing, lists get bullets,
links get underlines, and code, quotes, tables and images are styled. The examples here
use version 0.5.20 of the plugin.

## What it does, measured

We rendered a short article with `class="prose prose-slate"` in Chromium at 800px wide:

```text
article  max-width 655px (65ch)   font-size 16px   line-height 28px
h1       font-size 36px           font-weight 800
p        margin-top 20px
ul       list-style-type disc
blockquote  4px start border
inline <code>  ::before content "`"
```

Two of these deserve attention:

- **`max-width: 65ch`.** `prose` limits line length for readability. If your layout
  already controls width, add `max-w-none` to remove it.
- **Inline code gets literal backticks** added with `::before` and `::after`. Some
  designs want that; many don't. Remove them with
  `prose-code:before:content-none prose-code:after:content-none` (see modifiers below).

## Sizes

`prose-sm`, `prose-base` (the default), `prose-lg`, `prose-xl` and `prose-2xl` scale the
whole system together: font size, line height, heading sizes and all the spacing. With
`lg:prose-lg` added, the same article at 1280px measured font-size 18px, line-height
32px and an `h1` of 48px. Combine with breakpoints so phones get a slightly smaller
scale:

```html
<article class="prose lg:prose-lg">…</article>
```

## Colours and dark mode

The default colour scheme is gray. `prose-slate`, `prose-zinc`, `prose-neutral` and
`prose-stone` switch the grays to match your palette. For dark backgrounds,
`prose-invert` swaps every colour for its light counterpart:

```html
<article class="prose prose-slate dark:prose-invert">…</article>
```

The colours are CSS variables. In the compiled CSS, `prose` sets
`color: var(--tw-prose-body)` and defines values like
`--tw-prose-body: oklch(37.3% 0.034 259.733)`, and the colour classes just redefine those
variables. That's what makes a custom theme possible (below).

## Element modifiers

To adjust one element type inside the content, use a `prose-{element}:` modifier:

```html
<article class="prose prose-a:text-sky-700 prose-a:underline-offset-2
                prose-headings:tracking-tight prose-img:rounded-xl
                prose-code:before:content-none prose-code:after:content-none">
```

Compiled, a modifier targets descendants and skips anything inside `not-prose`:

```css
.prose-a\:text-sky-700 :is(:where(a):not(:where([class~="not-prose"], [class~="not-prose"] *))) {
  color: var(--color-sky-700);
}
```

The available modifiers include `prose-headings`, `prose-h1` … `prose-h4`, `prose-p`,
`prose-a`, `prose-strong`, `prose-em`, `prose-code`, `prose-pre`, `prose-blockquote`,
`prose-ul`, `prose-ol`, `prose-li`, `prose-img`, `prose-figure`, `prose-figcaption`,
`prose-table`, `prose-th`, `prose-td`, `prose-hr`, `prose-lead` and a few more. They
stack with other variants: `dark:prose-a:text-sky-400`.

In our test, `prose-headings:tracking-tight` gave the 36px `h1` a letter-spacing of
−0.9px, which is −0.025em.

## Opting out: `not-prose`

Content often contains embedded components: a newsletter signup, a code playground, a
custom callout. Wrap them in `not-prose` so the plugin's styles don't reach them:

```html
<article class="prose">
  <h2>Getting started</h2>
  <p>Install the package…</p>

  <div class="not-prose my-8 rounded-xl border border-sky-200 bg-sky-50 p-5">
    <p class="font-semibold text-sky-900">Before you start</p>
    <p class="mt-1 text-sm text-sky-800">You'll need Node.js 20 or newer.</p>
  </div>
</article>
```

In our test, a `<p>` inside `not-prose` had `margin-top: 0` (Preflight's value, not
prose's 20px), and a link inside it had no prose link colour. It still **inherits**
inherited properties like `color` and `font-size` from the article, though, so set
those explicitly on the component.

## A custom prose theme

To match your brand, define a utility that sets the plugin's variables, then use it
alongside `prose`:

```css title="src/app.css"
@import "tailwindcss";
@plugin "@tailwindcss/typography";

@utility prose-brand {
  --tw-prose-links: var(--color-emerald-700);
  --tw-prose-headings: var(--color-stone-900);
  --tw-prose-invert-links: var(--color-emerald-400);
}
```

```html
<article class="prose prose-stone prose-brand dark:prose-invert">…</article>
```

In our test, the article's links rendered in emerald-700 and its headings in stone-900.
The `--tw-prose-invert-*` variables are the ones `prose-invert` switches to, so you can
theme dark mode in the same place. Other variables follow the same naming:
`--tw-prose-body`, `--tw-prose-bold`, `--tw-prose-quotes`, `--tw-prose-code`,
`--tw-prose-pre-bg`, `--tw-prose-bullets` and so on. Check the plugin's compiled output
for the full list.

## Worked example: a blog post layout

```html
<main class="mx-auto max-w-3xl px-4 py-12 sm:px-6">
  <header class="mb-10">
    <p class="text-sm font-medium text-sky-700">Engineering</p>
    <h1 class="mt-2 text-3xl font-bold tracking-tight text-balance text-gray-900 sm:text-4xl">
      How we cut our CSS bundle in half
    </h1>
    <p class="mt-3 text-gray-600">By Priya Natarajan · 8 min read</p>
  </header>

  <article class="prose prose-slate max-w-none lg:prose-lg dark:prose-invert
                  prose-headings:scroll-mt-20 prose-headings:tracking-tight
                  prose-a:text-sky-700 prose-a:underline-offset-2 dark:prose-a:text-sky-400
                  prose-code:before:content-none prose-code:after:content-none
                  prose-code:rounded prose-code:bg-slate-100 prose-code:px-1
                  prose-pre:rounded-xl">
    {{ post.html | safe }}
  </article>
</main>
```

(The byline and title are sample content.) Decisions worth noting:

- The **title is outside `prose`** and styled with utilities, because it's part of the
  page design, not the content.
- `max-w-none` because the `<main>` already limits width.
- `prose-headings:scroll-mt-20` stops headings from hiding under a sticky header when
  someone follows a `#section` link.
- Inline code loses its backticks and gets a subtle background instead.

## How It Actually Works

The plugin registers `prose` and its size and colour classes as utilities. Each one
outputs a rule for the element itself (font size, colour, max width, the
`--tw-prose-*` variables) and many descendant rules: `.prose :where(p)`,
`.prose :where(h2)`, `.prose :where(a)` and so on. Those descendant selectors use
`:where()` so they add no specificity beyond the `.prose` class, and every one is
filtered with `:not(:where([class~="not-prose"], [class~="not-prose"] *))`, which is how
`not-prose` works.

That low specificity is deliberate: a class you put directly on an element inside the
content (if you can) will beat the prose rule. The element modifiers are variants that
generate the same kind of descendant selector, so `prose-a:text-sky-700` is effectively
"`text-sky-700` on every non-opted-out `<a>` inside this element".

## Common mistakes

- **Using `prose` for the whole page**, including navigation and layout. Wrap only the
  content.
- **Forgetting `max-w-none`** and wondering why the article is narrower than its column.
- **Not removing the inline-code backticks** when the design doesn't want them.
- **Forgetting `dark:prose-invert`**, leaving dark text on a dark background.
- **Embedding components without `not-prose`**, so they inherit list bullets, link
  underlines and margins.
- **Customising dozens of elements with modifiers** when a `prose-brand` utility setting
  variables would be cleaner.

## Exercise

1. Render a Markdown file (with headings, lists, a table, a blockquote, inline code and a
   code block) inside `prose`. Then try `prose-sm` and `prose-xl` and compare.
2. Remove the inline-code backticks and give inline code a background instead.
3. Add a callout component inside the article and protect it with `not-prose`. Check
   which styles still leak in by inheritance.
4. Create a `prose-brand` utility with your link and heading colours, including the
   inverted versions, and test it in dark mode.
