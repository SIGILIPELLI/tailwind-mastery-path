---
description: "CSS Grid with Tailwind: grid-cols and minmax(0, 1fr), spans and placement, arbitrary templates like auto-fit, subgrid and alignment."
---

# 07 · Grid Utilities

Grid is the layout tool for anything with rows *and* columns: galleries, dashboards, page
shells, forms with aligned labels. Tailwind's grid utilities cover the everyday cases with
short names and let you drop to any CSS grid template with an arbitrary value when you
need more. (For Grid itself, see
[CSS Grid Fundamentals](https://sigilipelli.github.io/html-css-mastery-path/level-2/05-grid-fundamentals/)
and [Grid in Depth](https://sigilipelli.github.io/html-css-mastery-path/level-2/06-grid-in-depth/).)

## Columns and rows

```css
.grid-cols-3 { grid-template-columns: repeat(3, minmax(0, 1fr)); }
.grid-rows-subgrid { grid-template-rows: subgrid; }
.auto-rows-fr { grid-auto-rows: minmax(0, 1fr); }
```

(From the compiler.) `grid-cols-{n}` works for any number, `grid-cols-none` removes the
template, and `grid-rows-{n}` does the same for rows.

### Why `minmax(0, 1fr)` matters — measured

Tailwind deliberately generates `minmax(0, 1fr)` rather than plain `1fr`. Plain `1fr` has an
automatic minimum of the content's minimum width, so one long unbreakable word can widen its
column. Three columns in a 600px grid, the middle one containing a long word:

```text
grid-cols-3                    -> [200, 200, 200]
grid-cols-[repeat(3,1fr)]      -> [115, 371, 115]
```

`grid-cols-3` kept the columns equal (the long word overflows its cell instead — pair with
`wrap-anywhere` on content that might contain long strings). The hand-written
`repeat(3,1fr)` let the content bully the layout. It's a good default to have.

## Spanning and placing

```html
<div class="grid grid-cols-4 gap-4">
  <div class="col-span-2 row-span-2">Feature</div>
  <div class="col-start-4">Pinned to column 4</div>
  <div class="col-span-full">Full width</div>
</div>
```

```css
.col-span-2 { grid-column: span 2 / span 2; }
.col-span-full { grid-column: 1 / -1; }
```

Also `col-start-{n}`, `col-end-{n}`, `row-start-{n}`, `row-end-{n}`, and `grid-flow-row`,
`grid-flow-col`, `grid-flow-dense` for the auto-placement direction.

## Arbitrary templates

Tailwind's named templates cover equal columns; everything else is an arbitrary value.
Inside square brackets, **underscores become spaces**:

```html
<div class="grid grid-cols-[200px_1fr]">sidebar + content</div>
<div class="grid grid-cols-[repeat(auto-fit,minmax(15rem,1fr))] gap-6">responsive gallery</div>
<div class="grid grid-rows-[auto_1fr_auto] min-h-dvh">header / main / footer</div>
```

These compile to exactly what you wrote:

```css
.grid-cols-\[200px_1fr\] { grid-template-columns: 200px 1fr; }
.grid-cols-\[repeat\(auto-fit\,minmax\(15rem\,1fr\)\)\] { grid-template-columns: repeat(auto-fit,minmax(15rem,1fr)); }
```

The `auto-fit` gallery is the most useful arbitrary template in practice: as many columns
as fit, each at least 15rem, no media queries. Add `min(15rem,100%)` inside the `minmax`
to stop it overflowing containers narrower than 15rem.

If you use a template more than once or twice, name it in the theme instead (Level 2 · 01)
so it becomes `grid-cols-gallery`.

## Subgrid

`grid-rows-subgrid` and `grid-cols-subgrid` let a child share its parent's tracks — the
standard fix for cards whose titles, bodies and buttons should line up across a row:

```html
<ul class="grid grid-cols-1 gap-6 md:grid-cols-3">
  <li class="row-span-3 grid grid-rows-subgrid gap-2 rounded-lg border border-gray-200 p-4">
    <h3 class="font-semibold">Short title</h3>
    <p class="text-sm text-gray-600">Body text of any length…</p>
    <a class="self-end text-sm font-medium text-sky-700" href="/a">Read more</a>
  </li>
  <!-- more cards -->
</ul>
```

Each card spans three of the parent's rows and uses them for its three children, so every
card's link sits at the same height.

## Alignment

`place-items-center`, `justify-items-start`, `items-center`, `justify-self-end`,
`self-start`, `place-content-center` — the same vocabulary as Flexbox, applied to grid
cells. `place-items-center` on a one-cell grid is the shortest way to centre something.

## Worked example: a page shell

```html
<body class="grid min-h-dvh grid-rows-[auto_1fr_auto]">
  <header class="border-b border-gray-200 p-4">…</header>
  <div class="grid md:grid-cols-[16rem_1fr]">
    <nav class="border-e border-gray-200 p-4 max-md:hidden" aria-label="Sections">…</nav>
    <main class="min-w-0 p-6">…</main>
  </div>
  <footer class="border-t border-gray-200 p-4 text-sm text-gray-500">…</footer>
</body>
```

Three rows (header and footer sized to content, the middle taking the rest), and from
`md` up a fixed 16rem sidebar beside the content. `min-w-0` on `<main>` stops wide
content such as tables or code from stretching the `1fr` column — the same minimum-size
issue as in Flexbox, because `1fr` in an arbitrary template is not `minmax(0, 1fr)`.

## How It Actually Works

For `grid-cols-{n}`, the compiler takes the numeric value and generates
`repeat(n, minmax(0, 1fr))` — a template, not a lookup, which is why any number works.

Arbitrary values go through a small **CSS value decoder**: the bracket contents are taken
literally, underscores are turned into spaces (write `\_` for a real underscore), and the
selector is escaped so that characters like `(`, `,` and `[` are valid in a class
selector — hence the heavily escaped `.grid-cols-\[repeat\(auto-fit\,…\)\]`. Tailwind
doesn't validate your grid syntax beyond basic structure; an invalid template produces a
CSS declaration the browser simply ignores, so if an arbitrary template "does nothing",
check the Computed pane for the property.

## Common mistakes

- **`grid-cols-[repeat(3,1fr)]`** where `grid-cols-3` would give the safer
  `minmax(0,1fr)`.
- **Spaces inside brackets** — `grid-cols-[200px 1fr]` is two separate (invalid) class
  names. Use underscores.
- **Forgetting `min-w-0`** on a `1fr` column holding wide content.
- **Too many arbitrary templates** instead of named theme values.
- **Grid for a single row of items** that should wrap by content — Flexbox with `flex-wrap`
  is simpler.

## Exercise

1. Build a responsive gallery with `grid-cols-[repeat(auto-fill,minmax(min(14rem,100%),1fr))]`.
   Test with 2 and 9 items; then switch to `auto-fit` and compare.
2. Build the page shell. Put a wide table in `<main>`, then remove `min-w-0` and see what
   happens at 800px wide.
3. Make a 4-column grid where the first card spans two columns and two rows.
4. Build three cards with subgrid and confirm the links align when titles differ in length.
